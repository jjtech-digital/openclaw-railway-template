# Engagement & Adoption Metrics

## When to use
Track post-activation behaviour — TUI sessions, gateway restarts, config edits — to understand how actively users interact with the wrapper after initial setup.

## Patterns

**TUI session duration**
```js
// src/server.js — WebSocket close handler for /tui
log.info("metrics", JSON.stringify({
  event: "tui.session_ended",
  durationMs: Date.now() - session.startedAt,
  reason: closeReason,   // "idle_timeout" | "max_duration" | "client_close"
}));
```

**Gateway restart tracking**
```js
// src/server.js — inside the gateway.restart console command handler
log.info("metrics", JSON.stringify({
  event: "gateway.restart",
  trigger: "debug_console",
  uptimeMs: Date.now() - gatewayStartTime,
}));
```

**Config edit signal**
```js
// src/server.js — after successful config save via /setup/api/config
log.info("metrics", JSON.stringify({
  event: "config.edited",
  backupCreated: true,
}));
```

## Pitfalls
- TUI idle timeout (`TUI_IDLE_TIMEOUT_MS`, default 300 000 ms) and max session (`TUI_MAX_SESSION_MS`, default 1 800 000 ms) both fire `session_ended`; always include `reason` to distinguish voluntary exits from timeouts.
- `gateway.restart` via the debug console is the only restart path tracked here. Automatic restarts triggered by crash-loops need a separate `gateway.crash_restart` event or they will be invisible in engagement dashboards.