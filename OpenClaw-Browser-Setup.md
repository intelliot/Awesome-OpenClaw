# OpenClaw Headed Browser Setup

(Verified February 20, 2026)

## Brief Summary
OpenClaw can run visible/headed browser automation.  

1. `curl -fsSL https://openclaw.ai/install.sh | bash` is still the recommended installer.
2. OpenClaw browser supports headful/headless; default is headful (`headless: false`).
3. OpenClaw browser actions use Playwright for advanced operations.
4. Partly outdated: `npx playbooks add skill ...` is not the current official OpenClaw docs path for skills; docs now center on `clawhub`.
5. Outdated: `claw start` style guidance is not the documented command surface; use `openclaw ...`.
6. Unverified in official docs: `PLAYWRIGHT_HEADLESS=false`, `slowMo`, `devtools` as OpenClaw config controls.
7. Outdated naming: browser extension is documented as OpenClaw Browser Relay flow, loaded via `openclaw browser extension install`.

## Step-by-Step Instructions

1. Install OpenClaw and onboard.
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon
```

2. Verify Gateway and open dashboard.
```bash
openclaw gateway status
openclaw dashboard
```
Use `http://127.0.0.1:18789/` if needed.

3. Configure managed visible browser mode (recommended for watching).
Edit `~/.openclaw/openclaw.json` so these values are set:
```json
{
  "browser": {
    "enabled": true,
    "defaultProfile": "openclaw",
    "headless": false
  }
}
```
Then restart the Gateway process/service.

4. Start and verify the managed browser window.
```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open http://127.0.0.1:5173
openclaw browser --browser-profile openclaw snapshot
```
You should see a visible browser window.

5. Run website-building workflow with the agent.
Use a prompt like:
```text
Use the browser tool with profile="openclaw" and target="host".
Open http://127.0.0.1:5173.
After each major UI change: take a full-page screenshot, then summarize what changed.
```

6. Optional: use your existing Chrome window via extension relay (instead of managed browser).
```bash
openclaw browser extension install
openclaw browser extension path
```
Then in Chrome: `chrome://extensions` → enable Developer Mode → Load unpacked → pick printed folder → click extension icon on target tab.

7. Optional: if you intentionally want `agent-browser` (third-party CLI), install directly.
```bash
npm install -g agent-browser
agent-browser install
agent-browser open https://example.com --headed
```
It supports `--headed` and CDP attach (`--cdp` / `connect`), but this is separate from official OpenClaw built-in browser docs.

## Command/Interface Corrections
- Use `openclaw ...` commands, not `claw start`.
- Use `~/.openclaw/openclaw.json` as primary config path.
- Prefer built-in OpenClaw `browser` tool first.
- For skills, official docs now document `clawhub ...` workflows.
- Avoid relying on undocumented OpenClaw knobs like `PLAYWRIGHT_HEADLESS`, `slowMo`, `devtools`.

## Validation Scenarios (Acceptance Checks)
1. `openclaw browser --browser-profile openclaw start` opens a visible browser window.
2. `openclaw browser --browser-profile openclaw snapshot` returns structured page output.
3. Agent can navigate, click, type, and screenshot in that same visible window.
4. If using extension mode, control only works after clicking extension icon on the tab.

## Assumptions and Defaults
- You want the most stable, officially documented path first.
- You are running OpenClaw locally (or with a reachable gateway host).
- You want visual/browser-observable automation for iterative website building.

## Sources
- [Install](https://docs.openclaw.ai/install/index)
- [Getting Started](https://docs.openclaw.ai/start/getting-started)
- [Browser (OpenClaw-managed)](https://docs.openclaw.ai/tools/browser)
- [FAQ (headless/headful default)](https://docs.openclaw.ai/help/faq)
- [CLI browser commands](https://docs.openclaw.ai/cli/browser)
- [Chrome extension relay](https://docs.openclaw.ai/tools/chrome-extension)
- [Browser troubleshooting (Linux)](https://docs.openclaw.ai/tools/browser-linux-troubleshooting)
- [Skills + ClawHub](https://docs.openclaw.ai/skills)
- [ClawHub CLI](https://docs.openclaw.ai/tools/clawhub)
- [agent-browser repo](https://github.com/vercel-labs/agent-browser)
