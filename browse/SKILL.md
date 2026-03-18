---
name: junshi-browse
description: "斥候 — 无头浏览器 QA 测试与 Dogfooding（agent-browser CLI）"
---

# gstack browse: QA Testing & Dogfooding

Persistent headless Chromium via `agent-browser` CLI. First call auto-starts (~3s), then ~100-200ms per command.
Auto-shuts down after 30 min idle. State persists between calls (cookies, tabs, sessions).

## SETUP

Verify agent-browser is available:

```bash
which agent-browser >/dev/null 2>&1 && echo "READY" || echo "NEEDS_SETUP"
```

If `NEEDS_SETUP`:
1. Tell the user: "agent-browser CLI is required but not installed. Install it first."
2. Check if npm/npx can install it: `npm install -g agent-browser` or equivalent.

## IMPORTANT

- Use the `agent-browser` CLI for all browser operations.
- Browser persists between calls — cookies, login sessions, and tabs carry over.
- Dialogs (alert/confirm/prompt) are auto-accepted by default — no browser lockup.
- **Show screenshots:** After taking screenshots, always use the Read tool on the output PNG(s) so the user can see them. Without this, screenshots are invisible.

## QA Workflows

### Test a user flow (login, signup, checkout, etc.)

```bash
# 1. Go to the page
agent-browser navigate https://app.example.com/login

# 2. See what's interactive
agent-browser snapshot

# 3. Fill the form using refs
agent-browser type @e3 "test@example.com"
agent-browser type @e4 "password123"
agent-browser click @e5

# 4. Verify it worked
agent-browser snapshot --diff              # diff shows what changed after clicking
agent-browser execute "document.querySelector('.dashboard') !== null"  # assert dashboard appeared
agent-browser screenshot --output /tmp/after-login.png
```

### Verify a deployment / check prod

```bash
agent-browser navigate https://yourapp.com
agent-browser execute "document.body.innerText"       # read the page — does it load?
agent-browser execute "window.__console_errors || []"  # any JS errors?
agent-browser execute "document.title"                 # correct title?
agent-browser execute "document.querySelector('.hero-section') !== null"  # key elements present?
agent-browser screenshot --output /tmp/prod-check.png
```

### Dogfood a feature end-to-end

```bash
# Navigate to the feature
agent-browser navigate https://app.example.com/new-feature

# Take annotated screenshot — shows every interactive element with labels
agent-browser snapshot --annotate --output /tmp/feature-annotated.png

# Walk through the flow
agent-browser snapshot                  # baseline
agent-browser click @e3                 # interact
agent-browser snapshot --diff           # what changed? (unified diff)

# Check element states
agent-browser execute "document.querySelector('.success-toast') !== null"
agent-browser execute "!document.querySelector('#next-step-btn').disabled"
agent-browser execute "document.querySelector('#agree-checkbox').checked"
```

### Test responsive layouts

```bash
# Manual: specific viewport
agent-browser execute "window.resizeTo(375, 812)"   # iPhone
agent-browser screenshot --output /tmp/mobile.png
agent-browser execute "window.resizeTo(1440, 900)"   # Desktop
agent-browser screenshot --output /tmp/desktop.png

# Element screenshot (crop to specific element)
agent-browser screenshot --output /tmp/hero.png
```

### Test file upload

```bash
agent-browser navigate https://app.example.com/upload
agent-browser snapshot
# Use type to set file input
agent-browser execute "document.querySelector('input[type=file]').setAttribute('data-test-file', '/path/to/test-file.pdf')"
agent-browser execute "document.querySelector('.upload-success') !== null"
agent-browser screenshot --output /tmp/upload-result.png
```

### Test forms with validation

```bash
agent-browser navigate https://app.example.com/form
agent-browser snapshot

# Submit empty — check validation errors appear
agent-browser click @e10                                # submit button
agent-browser snapshot --diff                           # diff shows error messages appeared
agent-browser execute "document.querySelector('.error-message') !== null"

# Fill and resubmit
agent-browser type @e3 "valid input"
agent-browser click @e10
agent-browser snapshot --diff                           # diff shows errors gone, success state
```

### Test dialogs (delete confirmations, prompts)

```bash
# Dialogs are auto-accepted by default
agent-browser click "#delete-button"     # triggers confirmation dialog
agent-browser snapshot --diff            # verify the item was deleted
```

### Test authenticated pages (import real browser cookies)

```bash
# If you have cookies exported as JSON:
# agent-browser execute "document.cookie = 'session=YOUR_TOKEN; domain=.github.com; path=/'"

# Now test authenticated pages
agent-browser navigate https://github.com/settings/profile
agent-browser snapshot
agent-browser screenshot --output /tmp/github-profile.png
```

### Compare two pages / environments

```bash
# Take screenshots of both environments and compare visually
agent-browser navigate https://staging.app.com
agent-browser screenshot --output /tmp/staging.png
agent-browser navigate https://prod.app.com
agent-browser screenshot --output /tmp/prod.png
```

### Multi-step chain (efficient for long flows)

```bash
# Execute multiple steps sequentially
agent-browser navigate https://app.example.com
agent-browser snapshot
agent-browser type @e3 "test@test.com"
agent-browser type @e4 "password"
agent-browser click @e5
agent-browser snapshot --diff
agent-browser screenshot --output /tmp/result.png
```

## Quick Assertion Patterns

```bash
# Element exists and is visible
agent-browser execute "document.querySelector('.modal') !== null"

# Button is enabled/disabled
agent-browser execute "!document.querySelector('#submit-btn').disabled"
agent-browser execute "document.querySelector('#submit-btn').disabled"

# Checkbox state
agent-browser execute "document.querySelector('#agree').checked"

# Page contains text
agent-browser execute "document.body.textContent.includes('Success')"

# Element count
agent-browser execute "document.querySelectorAll('.list-item').length"

# Specific attribute value
agent-browser execute "JSON.stringify(Array.from(document.querySelector('#logo').attributes).map(a => ({[a.name]: a.value})))"

# CSS property
agent-browser execute "getComputedStyle(document.querySelector('.button')).backgroundColor"
```

## Snapshot System

The snapshot is your primary tool for understanding and interacting with pages.

```
agent-browser snapshot              # Full accessibility tree with @e refs
agent-browser snapshot --diff       # Unified diff against previous snapshot
agent-browser snapshot --annotate   # Annotated screenshot with red overlay boxes and ref labels
agent-browser snapshot --annotate --output /tmp/annotated.png  # Save annotated screenshot
```

**Ref numbering:** @e refs are assigned sequentially (@e1, @e2, ...) in tree order.

After snapshot, use @refs as selectors in any command:
```bash
agent-browser click @e3       agent-browser type @e4 "value"     agent-browser execute "document.querySelector('[data-ref=e1]').style.color"
```

**Output format:** indented accessibility tree with @ref IDs, one element per line.
```
  @e1 [heading] "Welcome" [level=1]
  @e2 [textbox] "Email"
  @e3 [button] "Submit"
```

Refs are invalidated on navigation — run `snapshot` again after `navigate`.

## Command Reference

### Navigation
| Command | Description |
|---------|-------------|
| `navigate <url>` | Navigate to URL |
| (use browser back/forward via execute) | History navigation |

### Reading
| Command | Description |
|---------|-------------|
| `snapshot` | Accessibility tree with @e refs for element selection |
| `execute "document.body.innerText"` | Cleaned page text |
| `execute "document.title"` | Page title |
| `execute "JSON.stringify(Array.from(document.querySelectorAll('a')).map(a => a.href))"` | All links |

### Interaction
| Command | Description |
|---------|-------------|
| `click <sel>` | Click element |
| `type <sel> <val>` | Fill input / type text |
| `execute "<expr>"` | Run JavaScript expression and return result |

### Visual
| Command | Description |
|---------|-------------|
| `screenshot --output <path>` | Save screenshot |
| `snapshot --annotate --output <path>` | Annotated screenshot with ref labels |
| `snapshot --diff` | Diff against previous snapshot |

### Snapshot Flags
| Flag | Description |
|------|-------------|
| `--diff` / `-D` | Unified diff against previous snapshot (first call stores baseline) |
| `--annotate` / `-a` | Annotated screenshot with red overlay boxes and ref labels |
| `--output <path>` / `-o` | Output path for annotated screenshot |

## Tips

1. **Navigate once, query many times.** `navigate` loads the page; then `execute`, `screenshot` all hit the loaded page instantly.
2. **Use `snapshot` first.** See all interactive elements, then click/type by ref. No CSS selector guessing.
3. **Use `snapshot --diff` to verify.** Baseline → action → diff. See exactly what changed.
4. **Use assertions via `execute`.** `execute "document.querySelector('.modal') !== null"` is faster and more reliable than parsing page text.
5. **Use `snapshot --annotate` for evidence.** Annotated screenshots are great for bug reports.
6. **Check console after actions.** Catch JS errors that don't surface visually via `execute "window.__console_errors"`.
