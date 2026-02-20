## Telegram Setup

### Summary
1. `openclaw pairing approve telegram` requires a code: `openclaw pairing approve telegram <CODE>`.
2. `openclaw pairing list telegram` shows pending pairing requests, not the final allowlist.
3. `openclaw onboard --skip-install --skip-model` are not valid flags.
4. Config file is `~/.openclaw/openclaw.json` (or `OPENCLAW_CONFIG_PATH` override).
5. `openclaw plugins enable telegram` is usually optional for normal setup.
6. `channels add` does not prompt polling vs webhook. Polling is default unless webhook keys are set.
7. `channels.telegram.requireMention` is invalid. Use `channels.telegram.groups."*".requireMention`.

### Command Prefix
Use one command prefix consistently:

- Global install: `openclaw ...`
- Repo checkout (if `openclaw: command not found`): `pnpm openclaw ...`

In examples below, replace `<cmd>` with either `openclaw` or `pnpm openclaw`.

### Clean Up Invalid Config (if needed)
If you see `Unrecognized key: "requireMention"` under `channels.telegram`:

```bash
<cmd> doctor --fix
```

This removes unknown keys and repairs legacy config shapes.

### Setup Flow (No Manual JSON Editing)
1. Add Telegram with the interactive wizard:
```bash
<cmd> channels add
```

2. In the wizard:
- Select `telegram`.
- Enter bot token (or keep `TELEGRAM_BOT_TOKEN` if detected).
- Optional account/display name.
- Optional DM policy:
  - `pairing` (default/recommended)
  - `allowlist` (wizard can resolve `@username` to numeric ID when token is available)
  - `open`
  - `disabled`

3. Restart/apply runtime:
```bash
<cmd> gateway restart
```
If you run foreground instead of service:
```bash
<cmd> gateway
```

4. If DM policy is `pairing`, approve first DM:
```bash
<cmd> pairing list telegram
<cmd> pairing approve telegram <CODE>
```

5. Verify:
```bash
<cmd> channels status --probe
```

### Make Bot Reply Without `@mention`
Session-only override in Telegram:
- Send `/activation always` in that chat.

Persistent config:
```bash
<cmd> config set channels.telegram.groups.*.requireMention false
```

If group replies are still blocked by sender policy, open group sender policy:
```bash
<cmd> config set channels.telegram.groupPolicy open
```

Then restart:
```bash
<cmd> gateway restart
```

### Telegram Side Requirement (Important)
If bot says it cannot see normal group messages:

1. In `@BotFather`, run `/setprivacy` and disable privacy mode for this bot.
2. Remove and re-add the bot to the group.

Without this, non-mention messages may never reach the bot.

### Common Pitfalls
1. `channels.telegram.requireMention` is invalid (top-level key).
2. If `channels.telegram.groups` exists, it becomes a group allowlist. Include `"*"` or specific chat IDs you want allowed.
3. `groupPolicy: allowlist` requires sender IDs in `groupAllowFrom` (or fallback `allowFrom`).
4. Telegram allowlists should be numeric IDs. Legacy `@username` entries should be migrated with `doctor --fix`.

### Plugin Edge Case
Only needed if Telegram plugin was explicitly disabled/denied:

```bash
<cmd> plugins enable telegram
<cmd> gateway restart
```

### Telegram Bot API Limitation: Group Members
Telegram Bot API cannot return a full list of all group members in one call.

What it can do:
1. `getChatAdministrators` (admins only)
2. `getChatMemberCount` (count only)
3. `getChatMember(chat_id, user_id)` (single user lookup)
4. Membership updates via `chat_member` events (typically requires admin privileges)

### Canonical Docs
- https://docs.openclaw.ai/channels/telegram
- https://docs.openclaw.ai/cli/channels
- https://docs.openclaw.ai/channels/pairing
- https://docs.openclaw.ai/cli/gateway
- https://docs.openclaw.ai/cli/doctor
- https://docs.openclaw.ai/cli/config
- https://core.telegram.org/bots/api
- https://core.telegram.org/bots/faq
