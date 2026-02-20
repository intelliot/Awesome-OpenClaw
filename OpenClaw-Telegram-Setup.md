## Telegram Setup

### Summary

1. `openclaw pairing approve telegram` **requires a code** (`<CODE>`).  
2. `openclaw pairing list telegram` shows **pending requests**, not approved allowlist users.  
3. `openclaw onboard --skip-install --skip-model` flags are **not valid**.  
4. Config path is `~/.openclaw/openclaw.json` (or env override), not `~/.openclaw/config.json`.  
5. `openclaw plugins enable telegram` is usually **optional** for this flow.  
6. `channels add` wizard does **not** currently prompt webhook vs polling (polling is default unless webhook config is set).

### Setup Flow
1. Use the installed CLI directly (recommended if already globally installed):
```bash
openclaw channels add
```

2. In the wizard:
- Select `telegram`.
- Enter bot token (or keep `TELEGRAM_BOT_TOKEN` if detected).
- Optional: set display/account name.
- Optional: configure DM policy.
  - `pairing` (default/recommended): first DM gives a pairing code.
  - `allowlist`: add your numeric Telegram user ID (wizard can resolve `@username` to ID if token is available).

3. Apply changes to runtime:
- If running as a service:
```bash
openclaw gateway restart
```
- If running foreground:
```bash
openclaw gateway
```
(`openclaw daemon restart` also exists as a service alias.)

4. If DM policy is `pairing`, pair your user:
- DM bot with `/start` or any message.
- Then:
```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

5. Verify:
```bash
openclaw channels status --probe
```

### Test Cases / Scenarios
1. **Pairing mode path**: DM bot -> code appears -> approve with `<CODE>` -> replies work.
2. **Allowlist mode path**: your ID pre-added -> DM should work without pairing code.
3. **Service path**: `openclaw gateway restart` applies config.
4. **Plugin edge case**: if Telegram plugin was manually disabled/denied, enable/fix plugin policy first, then rerun `channels add`.

### Assumptions / Defaults
1. You already have a valid OpenClaw config (true if Slack is already configured).
2. You want no manual JSON editing.
3. Default Telegram DM policy is `pairing`.
4. Long polling is default unless webhook fields are explicitly configured.

### Canonical docs
- [https://docs.openclaw.ai/channels/telegram](https://docs.openclaw.ai/channels/telegram)
- [https://docs.openclaw.ai/cli/channels](https://docs.openclaw.ai/cli/channels)
- [https://docs.openclaw.ai/channels/pairing](https://docs.openclaw.ai/channels/pairing)
- [https://docs.openclaw.ai/cli/gateway](https://docs.openclaw.ai/cli/gateway)
- [https://docs.openclaw.ai/cli/onboard](https://docs.openclaw.ai/cli/onboard)

### Verified in code
- `/openclaw/src/commands/channels/add.ts`
- `/openclaw/src/commands/onboard-channels.ts`
- `/openclaw/src/channels/plugins/onboarding/telegram.ts`
- `/openclaw/src/cli/pairing-cli.ts`
- `/openclaw/src/cli/program/register.onboard.ts`
- `/openclaw/src/config/paths.ts`
- `/openclaw/src/cli/daemon-cli/register.ts`
