# Error: Control UI requires gateway.controlUi.allowedOrigins

Soon before 🦞 OpenClaw 2026.2.25 (d942e59), a breaking change was made to the config file:

`/Users/user/.openclaw/openclaw.json`

It caused this error:

```
11:01:33 Gateway failed to start: Error: non-loopback Control UI requires gateway.controlUi.allowedOrigins (set explicit
origins), or set gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true to use Host-header origin fallback mode
```

The fix is to explicitly set `gateway.controlUi.allowedOrigins` for your actual Control UI
origin(s) (instead of relying on Host-header fallback).

Use this exact config block:

```json
"gateway": {
"bind": "lan",
"controlUi": {
"allowedOrigins": [
"http://192.168.1.41:18789",
"http://localhost:18789",
"http://127.0.0.1:18789"
]
}
}
```

If you only want one origin, keep just that one (must be full origin: scheme + host + port).

Alternative (less safe) if you can’t pin origins yet:

```json
"gateway": {
"controlUi": {
"dangerouslyAllowHostHeaderOriginFallback": true
}
}
```

But preferred/hardened fix is allowedOrigins.
