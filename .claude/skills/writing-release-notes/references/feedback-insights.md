# Feedback & Insights Release Notes

## When to use
Reference this when drafting notes that close known issues, address user-reported bugs, or reflect changes driven by deployment feedback (e.g., token instability, WebSocket drops, Discord intent errors).

## Patterns

**Known issue resolved:**
```
### Fixed: WebSocket connections drop after redeploy
Token injection now uses `proxyReqWs` event handlers. Resolves `token_missing` errors reported after Railway redeploys without `OPENCLAW_GATEWAY_TOKEN` set.
```

**Workaround formalized:**
```
### `allowInsecureAuth` now set automatically during onboarding
Previously required manual config edit to prevent "pairing required" disconnects in the Control UI. The wrapper sets this flag as part of the onboard sequence.
```

**Discord-specific fix:**
```
### Discord setup now warns if MESSAGE CONTENT INTENT is missing
The setup wizard detects missing intent and surfaces a direct link to the Discord Developer Portal. Previously the bot would silently fail to receive messages.
```

## Pitfalls
- When closing a long-standing issue, name the symptom users experienced (e.g., "token_missing error"), not the internal cause — users search their logs, not the source code.
- If a fix requires user action on redeploy (clearing a volume, resetting config), mark it with a callout block, not just inline text.