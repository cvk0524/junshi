---
name: junshi-ship
description: "先锋 — 完整发布流程：测试→审查→版本→变更日志→提交→PR"
---

# Ship: Fully Automated Ship Workflow

## Preamble

```bash
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
```

## Completeness Principle — Boil the Lake

AI-assisted coding makes the marginal cost of completeness near-zero. Always prefer the complete implementation.

| Task type | Human team | AI-assisted | Compression |
|-----------|-----------|-------------|-------------|
| Boilerplate / scaffolding | 2 days | 15 min | ~100x |
| Test writing | 1 day | 15 min | ~50x |
| Feature implementation | 1 week | 30 min | ~30x |
| Bug fix + regression test | 4 hours | 15 min | ~20x |
| Architecture / design | 2 days | 4 hours | ~5x |
| Research / exploration | 1 day | 3 hours | ~3x |

## User Question Format

1. **Re-ground:** Project, branch (`_BRANCH`), task.
2. **Simplify:** Plain English.
3. **Recommend:** `Completeness: X/10` per option.
4. **Options:** `A) ... B) ... C) ...`

---

You are running the `/ship` workflow. This is **non-interactive, fully automated**. Do NOT ask for confirmation at any step. Run straight through and output the PR URL at the end.

**Only stop for:**
- On the base branch (abort)
- Merge conflicts that can't be auto-resolved (stop, show conflicts)
- Test failures (stop, show failures)
- Pre-landing review finds ASK items that need user judgment
- MINOR or MAJOR version bump needed (ask — see Step 4)
- Greptile review comments that need user decision
- TODOS.md missing and user wants to create one
- TODOS.md disorganized and user wants to reorganize

**Never stop for:**
- Uncommitted changes (always include them)
- Version bump choice (auto-pick MICRO or PATCH — see Step 4)
- CHANGELOG content (auto-generate from diff)
- Commit message approval (auto-commit)
- Multi-file changesets (auto-split into bisectable commits)
- TODOS.md completed-item detection (auto-mark)
- Auto-fixable review findings (dead code, N+1, stale comments — fixed automatically)
- Test coverage gaps (auto-generate and commit, or flag in PR body)

---

## Step 0: Detect base branch

1. `gh pr view --json baseRefName -q .baseRefName`
2. If fails: `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`
3. If both fail: `main`

---

## Step 1: Pre-flight

1. Check the current branch. If on the base branch → **abort**: "You're on the base branch. Ship from a feature branch."

2. Run `git status` (never use `-uall`). Uncommitted changes are always included.

3. Run `git diff <base>...HEAD --stat` and `git log <base>..HEAD --oneline` to understand what's being shipped.

4. Check review readiness:

### Review Readiness Dashboard

```bash
SLUG=$(basename "$(git remote get-url origin 2>/dev/null || echo 'unknown')" .git)
BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
cat ~/.gstack/projects/$SLUG/$BRANCH-reviews.jsonl 2>/dev/null || echo "NO_REVIEWS"
```

Parse and display:
```
+====================================================================+
|                    REVIEW READINESS DASHBOARD                       |
+====================================================================+
| Review          | Runs | Last Run            | Status    | Required |
|-----------------|------|---------------------|-----------|----------|
| Eng Review      |  ?   | —                   | —         | YES      |
| CEO Review      |  ?   | —                   | —         | no       |
| Design Review   |  ?   | —                   | —         | no       |
+--------------------------------------------------------------------+
| VERDICT: ...                                                        |
+====================================================================+
```

**Review tiers:**
- **Eng Review (required by default):** The only review that gates shipping.
- **CEO Review (optional):** Recommend for big product/business changes.
- **Design Review (optional):** Recommend for UI/UX changes.

**Verdict logic:**
- **CLEARED**: Eng Review has >= 1 entry within 7 days with status "clean"
- **NOT CLEARED**: Eng Review missing, stale (>7 days), or has open issues

**Eng Review gates shipping.** If NOT CLEARED and no prior override, ask user:
- A) Ship anyway  B) Abort — run /plan-eng-review first  C) Change is too small to need eng review

If the user chooses A or C, persist the decision:
```bash
SLUG=$(basename "$(git remote get-url origin 2>/dev/null || echo 'unknown')" .git)
echo '{"skill":"ship-review-override","timestamp":"'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'","decision":"USER_CHOICE"}' >> ~/.gstack/projects/$SLUG/$BRANCH-reviews.jsonl
```

Check for frontend scope:
```bash
FRONTEND_EXTS="tsx|jsx|vue|svelte|css|scss|less|html|erb|haml|slim"
SCOPE_FRONTEND=$(git diff origin/<base> --name-only | grep -qE "\.($FRONTEND_EXTS)$" && echo "true" || echo "false")
```
If `SCOPE_FRONTEND=true` and no design review exists, mention it informally.

---

## Step 2: Merge the base branch (BEFORE tests)

```bash
git fetch origin <base> && git merge origin/<base> --no-edit
```

If merge conflicts: try auto-resolve simple ones (VERSION, CHANGELOG ordering). If complex → **STOP**.
If already up to date: continue silently.

---

## Step 2.5: Test Framework Bootstrap

Detect runtime and existing test framework. If no test framework, offer to bootstrap one. See the shared Test Framework Bootstrap procedure (same as in /qa).

---

## Step 3: Run tests (on merged code)

Run detected test suites. **If any test fails → STOP.** Do not proceed.
**If all pass:** Continue silently — just note the counts briefly.

---

## Step 3.25: Eval Suites (conditional)

Evals are mandatory when prompt-related files change. Skip if no prompt files in the diff.

**1. Check if the diff touches prompt-related files:**
```bash
git diff origin/<base> --name-only
```
Match against patterns like: `*_prompt_builder.*`, `*_generation_service.*`, `system_prompts/*`, `test/evals/**`

**If no matches:** "No prompt-related files changed — skipping evals." Continue.

**2. Identify affected eval suites** from runner files.

**3. Run affected suites** at full tier.

**4. Check results:** If any eval fails → STOP. If all pass → note counts. Continue.

**5. Save eval output** for PR body.

---

## Step 3.4: Test Coverage Audit

100% coverage is the goal — every untested path is a path where bugs hide.

**0. Before/after test count:**
```bash
find . -name '*.test.*' -o -name '*.spec.*' -o -name '*_test.*' -o -name '*_spec.*' | grep -v node_modules | wc -l
```

**1. Trace every codepath changed** using `git diff origin/<base>...HEAD`:

Read every changed file. For each one, trace how data flows through the code:
1. Read the diff. For each changed file, read the full file to understand context.
2. Trace data flow — input source, transforms, output, error points.
3. Diagram the execution — every function, conditional branch, error path, edge.

**2. Map user flows, interactions, and error states:**
- User flows: what sequence of actions touches this code?
- Interaction edge cases: double-click, navigate away, stale data, slow connection, concurrent actions
- Error states: clear message or silent failure? Can user recover?
- Empty/zero/boundary states

**3. Check each branch against existing tests:**
Quality scoring rubric:
- ★★★  Tests behavior with edge cases AND error paths
- ★★   Tests correct behavior, happy path only
- ★    Smoke test / existence check / trivial assertion

**4. Output ASCII coverage diagram:**

```
CODE PATH COVERAGE
===========================
[+] src/services/billing.ts
    │
    ├── processPayment()
    │   ├── [★★★ TESTED] Happy path + card declined + timeout — billing.test.ts:42
    │   ├── [GAP]         Network timeout — NO TEST
    │   └── [GAP]         Invalid currency — NO TEST
    │
    └── refundPayment()
        ├── [★★  TESTED] Full refund — billing.test.ts:89
        └── [★   TESTED] Partial refund (checks non-throw only) — billing.test.ts:101

USER FLOW COVERAGE
===========================
[+] Payment checkout flow
    │
    ├── [★★★ TESTED] Complete purchase — checkout.e2e.ts:15
    ├── [GAP]         Double-click submit — NO TEST
    └── [GAP]         Navigate away during payment — NO TEST

─────────────────────────────────
COVERAGE: 5/12 paths tested (42%)
  Code paths: 3/5 (60%)
  User flows: 2/7 (29%)
QUALITY:  ★★★: 2  ★★: 2  ★: 1
GAPS: 7 paths need tests
─────────────────────────────────
```

**Fast path:** All paths covered → "Step 3.4: All new code paths have test coverage ✓" Continue.

**5. Generate tests for uncovered paths:**
If test framework detected:
- Prioritize error handlers and edge cases first
- Read 2-3 existing test files to match conventions
- Generate unit tests. Mock all external dependencies.
- Run each test. Passes → commit as `test: coverage for {feature}`
- Fails → fix once. Still fails → revert, note gap.

Caps: 30 code paths max, 20 tests generated max, 2-min per-test cap.

**6. After-count and coverage summary:**
For PR body: `Tests: {before} → {after} (+{delta} new)`

---

## Step 3.5: Pre-Landing Review

Run the `/review` workflow inline (two-pass checklist, design review if frontend files changed, Fix-First flow).

Check if diff touches frontend files:
```bash
FRONTEND_EXTS="tsx|jsx|vue|svelte|css|scss|less|html|erb|haml|slim"
SCOPE_FRONTEND=$(git diff origin/<base> --name-only | grep -qE "\.($FRONTEND_EXTS)$" && echo "true" || echo "false")
```

If `SCOPE_FRONTEND=true`: run design review inline.

If fixes applied → commit and tell user to re-run `/ship`. If no fixes → continue.

---

## Step 3.75: Address Greptile review comments (if PR exists)

Process Greptile comments if available. Skip silently if not.

For each classified comment:
- **VALID & ACTIONABLE:** Fix-First flow. If user chooses fix → apply, commit, reply.
- **FALSE POSITIVE:** Ask user: reply/fix/ignore.
- **VALID BUT ALREADY FIXED:** Reply automatically.
- **SUPPRESSED:** Skip silently.

After all comments resolved: if fixes applied, re-run tests before continuing.

---

## Step 4: Version bump (auto-decide)

1. Read `VERSION` file (4-digit: `MAJOR.MINOR.PATCH.MICRO`)

2. Auto-decide:
   - Count lines changed (`git diff origin/<base>...HEAD --stat | tail -1`)
   - **MICRO** (4th digit): < 50 lines changed
   - **PATCH** (3rd digit): 50+ lines changed
   - **MINOR** (2nd digit): **ASK** — major features
   - **MAJOR** (1st digit): **ASK** — milestones or breaking changes

3. Compute: bumping a digit resets all digits to its right to 0.

4. Write new version to `VERSION`.

---

## Step 5: CHANGELOG (auto-generate)

1. Read `CHANGELOG.md` header for format.

2. Auto-generate from **ALL commits on the branch**:
   - `git log <base>..HEAD --oneline`
   - `git diff <base>...HEAD`
   - Categorize: `### Added` / `### Changed` / `### Fixed` / `### Removed`
   - Format: `## [X.Y.Z.W] - YYYY-MM-DD`

**Do NOT ask the user to describe changes.** Infer from diff and commit history.

---

## Step 5.5: TODOS.md (auto-update)

Cross-reference TODOS.md against changes. Mark completed items automatically.

**1. Check if TODOS.md exists.**

**If not:** Ask: A) Create it now, B) Skip.

**2. Check structure and organization.** If disorganized, offer to reorganize.

**3. Detect completed TODOs:** Use diff and commit history. **Be conservative** — only mark with clear evidence.

**4. Move completed items** to `## Completed` section with `**Completed:** vX.Y.Z (YYYY-MM-DD)`.

**5. Output summary:**
- `TODOS.md: N items marked complete. M items remaining.`

**6. Defensive:** Never stop the ship workflow for a TODOS failure.

---

## Step 6: Commit (bisectable chunks)

**Goal:** Create small, logical commits that work well with `git bisect`.

1. Group changes into logical commits. Each = one coherent change.

2. **Commit ordering** (earlier first):
   - **Infrastructure:** migrations, config, routes
   - **Models & services:** with their tests
   - **Controllers & views:** with their tests
   - **VERSION + CHANGELOG + TODOS.md:** always final commit

3. **Rules:**
   - Model + test → same commit
   - Service + test → same commit
   - If total diff < 50 lines across < 4 files → single commit is fine
   - Each commit independently valid — no broken imports

4. Final commit:
```bash
git commit -m "chore: bump version and changelog (vX.Y.Z.W)

Co-Authored-By: CodeBuddy <noreply@codebuddy.ai>"
```

---

## Step 7: Push

```bash
git push -u origin <branch-name>
```

---

## Step 8: Create PR

```bash
gh pr create --base <base> --title "<type>: <summary>" --body "..."
```

Include sections:
- **Summary** — bullet points from CHANGELOG
- **Test Coverage** — coverage diagram from Step 3.4
- **Pre-Landing Review** — findings from Step 3.5
- **Design Review** — if applicable
- **Eval Results** — if applicable
- **Greptile Review** — if applicable
- **TODOS** — completed items
- **Test plan** — pass counts

**Output the PR URL** — this should be the final output the user sees.

---

## Important Rules

- **Never skip tests.** If tests fail, stop.
- **Never skip the pre-landing review.**
- **Never force push.** Use regular `git push` only.
- **Never ask for confirmation** except MINOR/MAJOR bumps and ASK items.
- **Always use 4-digit version format.**
- **Date format in CHANGELOG:** `YYYY-MM-DD`
- **Split commits for bisectability.**
- **TODOS.md completion detection must be conservative.**
- **Step 3.4 generates coverage tests.** They must pass before committing.
- **The goal: user says `/ship`, next thing they see is the PR URL.**
