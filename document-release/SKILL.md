---
name: junshi-document-release
description: "文书 — 发布后文档更新：基于 diff 交叉检查所有文档"
---

# Document Release: Post-Ship Documentation Update

You are running the `/document-release` workflow. This runs **after `/ship`** (code committed, PR exists or about to exist) but **before the PR merges**. Your job: ensure every documentation file in the project is accurate, up to date, and written in a friendly, user-forward voice.

You are mostly automated. Make obvious factual updates directly. Stop and ask only for risky or subjective decisions.

**Only stop for:**
- Risky/questionable doc changes (narrative, philosophy, security, removals, large rewrites)
- VERSION bump decision (if not already bumped)
- New TODOS items to add
- Cross-doc contradictions that are narrative (not factual)

**Never stop for:**
- Factual corrections clearly from the diff
- Adding items to tables/lists
- Updating paths, counts, version numbers
- Fixing stale cross-references
- CHANGELOG voice polish (minor wording adjustments)
- Marking TODOS complete
- Cross-doc factual inconsistencies (e.g., version number mismatch)

**NEVER do:**
- Overwrite, replace, or regenerate CHANGELOG entries — polish wording only, preserve all content
- Bump VERSION without asking — always use AskUserQuestion for version changes
- Use write_to_file on CHANGELOG.md — always use replace_in_file with exact old_string matches

## Step 0: Detect base branch

Determine which branch this PR targets.

1. `gh pr view --json baseRefName -q .baseRefName` — if succeeds, use it.
2. `gh repo view --json defaultBranchRef -q .defaultBranchRef.name` — fallback.
3. If both fail, fall back to `main`.

## Step 1: Pre-flight & Diff Analysis

1. Check current branch. If on base branch, **abort**.
2. Gather context:

```bash
git diff <base>...HEAD --stat
git log <base>..HEAD --oneline
git diff <base>...HEAD --name-only
```

3. Discover all documentation files:

```bash
find . -maxdepth 2 -name "*.md" -not -path "./.git/*" -not -path "./node_modules/*" -not -path "./.gstack/*" -not -path "./.context/*" | sort
```

4. Classify changes: New features, Changed behavior, Removed functionality, Infrastructure.
5. Output: "Analyzing N files changed across M commits. Found K documentation files to review."

## Step 2: Per-File Documentation Audit

Read each doc file and cross-reference against the diff:

**README.md:** Features match diff? Install/setup instructions current? Examples valid? Troubleshooting accurate?

**ARCHITECTURE.md:** ASCII diagrams match code? Design decisions accurate? Be conservative — only update things clearly contradicted by the diff.

**CONTRIBUTING.md — New contributor smoke test:** Walk through setup as a new contributor. Commands accurate? Test tiers match? Workflows current? Flag anything that would fail or confuse.

**CLAUDE.md / project instructions:** Project structure match file tree? Commands accurate? Build/test instructions match package.json?

**Any other .md files:** Read, determine purpose and audience. Cross-reference against diff.

For each file, classify updates as:
- **Auto-update** — Factual corrections clearly warranted by the diff.
- **Ask user** — Narrative changes, section removal, security changes, large rewrites (>10 lines), ambiguous relevance, new sections.

## Step 3: Apply Auto-Updates

Make all clear, factual updates directly using replace_in_file. For each file, output a one-line summary: "README.md: added /new-skill to skills table, updated skill count from 9 to 10."

**Never auto-update:** README intro/positioning, ARCHITECTURE philosophy, Security model descriptions. Never remove entire sections.

## Step 4: Ask About Risky/Questionable Changes

For each risky update, AskUserQuestion with context, the decision, RECOMMENDATION, and options including C) Skip.

## Step 5: CHANGELOG Voice Polish

**CRITICAL — NEVER CLOBBER CHANGELOG ENTRIES.**

**Rules:**
1. Read entire CHANGELOG.md first.
2. Only modify wording within existing entries. Never delete, reorder, or replace.
3. Never regenerate from scratch — the entry is the source of truth.
4. If entry looks wrong, AskUserQuestion — do NOT silently fix.
5. Use replace_in_file with exact old_string — never write_to_file on CHANGELOG.

**If CHANGELOG not modified in branch:** skip.

**If modified**, review for voice:
- **Sell test:** Would a user think "oh nice, I want to try that"?
- Lead with what user can now **do** — not implementation details.
- "You can now..." not "Refactored the..."
- Flag entries that read like commit messages.
- Internal changes → separate "### For contributors" subsection.
- Auto-fix minor voice. AskUserQuestion if rewrite alters meaning.

## Step 6: Cross-Doc Consistency & Discoverability Check

1. README features match CLAUDE.md descriptions?
2. ARCHITECTURE components match CONTRIBUTING structure?
3. CHANGELOG latest version match VERSION file?
4. **Discoverability:** Every doc reachable from README or CLAUDE.md? Flag orphans.
5. Flag contradictions. Auto-fix factual; AskUserQuestion for narrative.

## Step 7: TODOS.md Cleanup

If TODOS.md doesn't exist, skip.

1. **Completed items:** Cross-reference diff against open TODOs. Mark completed with `**Completed:** vX.Y.Z.W (YYYY-MM-DD)`. Be conservative.
2. **Stale descriptions:** If TODO references changed files/components, AskUserQuestion to confirm update/complete/leave.
3. **New deferred work:** Check diff for `TODO`, `FIXME`, `HACK`, `XXX` comments. AskUserQuestion for each meaningful one.

## Step 8: VERSION Bump Question

**NEVER BUMP WITHOUT ASKING.**

1. If VERSION doesn't exist: skip.
2. Check if already modified: `git diff <base>...HEAD -- VERSION`
3. **If NOT bumped:** AskUserQuestion — A) PATCH, B) MINOR, C) Skip (recommended for docs-only).
4. **If already bumped:** Check if bump covers full scope. If CHANGELOG covers everything, skip. If significant uncovered changes exist, AskUserQuestion — A) Bump next patch, B) Keep current, C) Skip.

## Step 9: Commit & Output

**Empty check:** `git status`. If no docs modified, output "All documentation is up to date." and exit.

**Commit:**
1. Stage by name (never `git add -A`).
2. Commit:

```bash
git commit -m "$(cat <<'EOF'
docs: update project documentation for vX.Y.Z.W

Co-Authored-By: CodeBuddy <noreply@codebuddy.ai>
EOF
)"
```

3. Push: `git push`

**PR body update:**
1. `gh pr view --json body -q .body > /tmp/gstack-pr-body-$$.md`
2. If `## Documentation` section exists, replace it; otherwise append.
3. Include doc diff preview — what specifically changed per file.
4. `gh pr edit --body-file /tmp/gstack-pr-body-$$.md`
5. `rm -f /tmp/gstack-pr-body-$$.md`
6. If no PR: skip with message.
7. If gh pr edit fails: warn and continue.

**Doc health summary (final output):**

```
Documentation health:
  README.md       [status] ([details])
  ARCHITECTURE.md [status] ([details])
  CONTRIBUTING.md [status] ([details])
  CHANGELOG.md    [status] ([details])
  TODOS.md        [status] ([details])
  VERSION         [status] ([details])
```

Status: Updated, Current, Voice polished, Not bumped, Already bumped, Skipped.

## Important Rules

- **Read before editing.** Always read full content before modifying.
- **Never clobber CHANGELOG.** Polish wording only.
- **Never bump VERSION silently.** Always ask.
- **Be explicit about what changed.** Every edit gets a one-line summary.
- **Generic heuristics, not project-specific.** Works on any repo.
- **Discoverability matters.** Every doc reachable from README or CLAUDE.md.
- **Voice: friendly, user-forward, not obscure.**
