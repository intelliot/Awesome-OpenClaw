## Mac-First Headed Browser Relay + Linux Gateway Browser

### Summary
`Mac Relay First` - for true headed usage.
Plan is to:
1. Keep the Gateway on your Debian/Ubuntu Proxmox LXC.
2. Use Tailscale even on same LAN (recommended and safe).
3. Pair your Mac as a node host, run Chrome extension relay there, and drive it from the Gateway.
4. Also install a supported browser on Linux for fallback/automation.
5. Add multi-browser support in a controlled way after first success.

### Public Interfaces / Types Used (No API Changes)
No product API changes are needed. This tutorial uses existing config + CLI/tool interfaces:

- Config keys:
`browser.enabled`, `browser.defaultProfile`, `browser.executablePath`, `browser.headless`, `browser.noSandbox`, `gateway.nodes.browser.mode`, `gateway.nodes.browser.node`, `nodeHost.browserProxy.enabled`, `agents.defaults.sandbox.browser.allowHostControl`
- CLI surfaces:
`openclaw browser ...`, `openclaw browser extension ...`, `openclaw node ...`, `openclaw nodes pending|approve|list`
- Agent tool params:
`browser` tool with `profile`, `target`, `node`, `action`
- Important behavior lock-in:
When using agent tool with `profile="chrome"`, explicitly pass `target="node"` for remote Mac relay control.

### Step-by-Step Implementation

### 1. Linux Gateway preflight (LXC)
Run on the Linux Gateway host:

```bash
source /etc/os-release && echo "$ID $VERSION_ID"
dpkg --print-architecture
openclaw --version
openclaw gateway status
openclaw config get browser
openclaw config get gateway.nodes.browser
```

Expected:
- Debian/Ubuntu
- Architecture known (`amd64` vs `arm64`)
- Gateway running

### 2. Secure connectivity baseline (Tailscale on same LAN is OK and recommended)
On Linux + Mac:
1. Bring both hosts onto the same tailnet.
2. Keep Gateway private; do not expose browser relay ports publicly.

On Linux Gateway host:

```bash
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
openclaw config set gateway.bind tailnet
openclaw gateway restart
openclaw gateway status
```

Record:
- Tailnet IP/hostname of Linux Gateway
- `gateway.auth.token` (needed by Mac node host)

### 3. Install supported browser on Linux LXC (first local baseline)
Recommended first local browser on Ubuntu/Debian LXC: `Google Chrome stable` (amd64), because Chromium on Ubuntu commonly routes via snap transitional packaging.

If `amd64`:

```bash
wget -q https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install -y ./google-chrome-stable_current_amd64.deb
```

Configure OpenClaw browser on Linux:

```bash
openclaw config set browser.enabled true --json
openclaw config set browser.defaultProfile openclaw
openclaw config set browser.executablePath /usr/bin/google-chrome-stable
openclaw config set browser.headless false --json
openclaw config set browser.noSandbox false --json
openclaw gateway restart
```

Smoke test:

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw tabs
```

If launch fails in container with sandbox error:

```bash
openclaw config set browser.noSandbox true --json
openclaw gateway restart
openclaw browser --browser-profile openclaw start
```

### 4. Mac node host setup (for headed relay control)
On Mac:

```bash
openclaw --version
export OPENCLAW_GATEWAY_TOKEN="<token-from-linux-gateway>"
openclaw node install --host <linux-gateway-tailnet-ip-or-name> --port 18789 --display-name "mac-browser-node"
openclaw node restart
openclaw node status
```

On Linux Gateway host, approve pairing:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes list
```

Enable browser routing policy on Gateway:

```bash
openclaw config set gateway.nodes.browser.mode auto
# If more than one browser-capable node exists:
# openclaw config set gateway.nodes.browser.node "<node-id-or-name>"
openclaw gateway restart
```

### 5. Install and attach OpenClaw Browser Relay extension on Mac
On Mac:

```bash
openclaw browser extension install
openclaw browser extension path
```

In Chrome:
1. Open `chrome://extensions`
2. Enable `Developer mode`
3. `Load unpacked` and choose the path from `openclaw browser extension path`
4. Pin `OpenClaw Browser Relay` icon
5. Open target tab, click extension icon once to attach

Badge states:
- `ON` = attached (good)
- `…` = connecting
- `!` = relay unreachable

### 6. Verify relay path from Linux Gateway
From Linux Gateway host:

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile chrome open https://example.com
openclaw browser --browser-profile chrome snapshot --interactive --compact
openclaw browser --browser-profile chrome screenshot --full-page
```

For agent-driven calls (important for remote Mac relay):
- Use `profile="chrome"` plus `target="node"` (and optionally `node="<mac-node-id>"`).

### 7. Browser capture workflow for web-dev progress
Operational loop:
1. `openclaw browser --browser-profile chrome open <dev-url>`
2. `openclaw browser --browser-profile chrome snapshot --interactive --compact`
3. Interact (`click`, `type`, `navigate`) as needed
4. Capture:
   `openclaw browser --browser-profile chrome screenshot --full-page`
5. Use returned `MEDIA:<path>` artifacts for progress checkpoints

### 8. Multi-browser expansion (after first success)
Order to add:
1. Chrome (done)
2. Brave
3. Edge
4. Chromium (only where packaging is reliable for your distro)

Rules:
- Local OpenClaw-managed profiles share global `browser.executablePath`, so switch executable per run when needed.
- For stable concurrent multi-browser setups, prefer separate remote CDP endpoints or separate nodes and pin with `gateway.nodes.browser.node`.
- You can create additional routing profiles:
```bash
openclaw browser create-profile --name brave-remote --cdp-url http://127.0.0.1:9223 --color "#00AA00"
```

## Test Cases and Scenarios

1. Linux local profile starts:
`openclaw browser --browser-profile openclaw start` shows running state.
2. Mac relay attach:
Extension badge becomes `ON` on selected tab.
3. Gateway-to-node routing works:
`openclaw browser --browser-profile chrome tabs` on Linux returns Mac tab list.
4. Agent node targeting works:
Agent browser calls succeed only when `target="node"` is specified for remote relay path.
5. Screenshot capture path works:
`openclaw browser --browser-profile chrome screenshot --full-page` returns `MEDIA:<path>`.
6. Failure signatures handled:
- `Chrome extension relay ... no tab connected` → click extension icon on tab.
- `Failed to start Chrome CDP` → check executable path; in LXC consider `browser.noSandbox=true`.

## Assumptions and Defaults Chosen
- Distro is Debian/Ubuntu (your selection).
- Primary milestone is Mac headed relay first.
- Tailscale is used even on same LAN (recommended default).
- Linux browser is installed as a fallback and for non-relay scenarios.
- Gateway and node remain private (tailnet/loopback), not public-exposed.

## References
- [https://docs.openclaw.ai/tools/browser](https://docs.openclaw.ai/tools/browser)
- [https://docs.openclaw.ai/tools/chrome-extension](https://docs.openclaw.ai/tools/chrome-extension)
- [https://docs.openclaw.ai/tools/browser-linux-troubleshooting](https://docs.openclaw.ai/tools/browser-linux-troubleshooting)
- [https://docs.openclaw.ai/cli/browser](https://docs.openclaw.ai/cli/browser)
- [https://docs.openclaw.ai/cli/node](https://docs.openclaw.ai/cli/node)
- [https://docs.openclaw.ai/nodes](https://docs.openclaw.ai/nodes)
- [https://docs.openclaw.ai/gateway/security](https://docs.openclaw.ai/gateway/security)
- [https://docs.openclaw.ai/gateway/tailscale](https://docs.openclaw.ai/gateway/tailscale)
- [https://support.google.com/chrome/a/answer/9025903?hl=en](https://support.google.com/chrome/a/answer/9025903?hl=en)
- [https://support.google.com/chrome/a/answer/9025926?hl=en](https://support.google.com/chrome/a/answer/9025926?hl=en)
- [https://brave.com/linux/](https://brave.com/linux/)
- [https://www.microsoft.com/edge/business/download](https://www.microsoft.com/edge/business/download)
- [https://ubuntu.com/blog/chromium-in-ubuntu-deb-to-snap-transition](https://ubuntu.com/blog/chromium-in-ubuntu-deb-to-snap-transition)

















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
