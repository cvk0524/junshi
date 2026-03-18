---
name: junshi-setup-browser-cookies
description: "密使 — 为需要认证的 QA 测试导入浏览器 Cookie"
---

# Setup Browser Cookies

Import logged-in sessions from your real Chromium browser into the headless agent-browser session.

## How it works

1. Verify agent-browser is available
2. Use browser cookie import capabilities to load cookies
3. Cookies are loaded into the Playwright session
4. Verify cookies are active

## Steps

### 1. Verify agent-browser is available

```bash
which agent-browser >/dev/null 2>&1 && echo "READY" || echo "NEEDS_SETUP"
```

If `NEEDS_SETUP`:
1. Tell the user: "agent-browser CLI is required. Install it first."
2. Provide installation guidance.

### 2. Import cookies

If the user specifies a domain directly (e.g., `/setup-browser-cookies github.com`), use the agent-browser to set cookies:

```bash
# Option A: If you have cookies as JSON, import them via execute
agent-browser execute "document.cookie = 'session=TOKEN; domain=.github.com; path=/'"

# Option B: Navigate to the site and manually set cookies
agent-browser navigate https://github.com
agent-browser execute "document.cookie = 'YOUR_COOKIE_STRING'"
```

For more complex cookie import scenarios, the user may need to:
1. Export cookies from their browser (via a browser extension or developer tools)
2. Save them as a JSON file
3. Load them via agent-browser's execute command

### 3. Alternative: Manual Login

If cookie import is too complex, the user can log in directly through agent-browser:

```bash
agent-browser navigate https://github.com/login
agent-browser snapshot
# Fill login form
agent-browser type @e3 "username"
agent-browser type @e4 "password"
agent-browser click @e5
# Verify login succeeded
agent-browser snapshot --diff
```

### 4. Verify

After importing cookies or logging in:

```bash
agent-browser execute "document.cookie"
```

Show the user a summary of active cookies (domain counts).

Test that the session works:

```bash
agent-browser navigate https://github.com/settings/profile
agent-browser snapshot
agent-browser screenshot --output /tmp/auth-check.png
```

## Notes

- The agent-browser session persists cookies between commands, so imported cookies work immediately
- If a site requires 2FA, the user will need to complete that step manually
- Cookie-based auth may expire — re-import if sessions become stale
- For sites with short-lived sessions, prefer the manual login approach
