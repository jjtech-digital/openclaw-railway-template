---
name: instrumenting-product-metrics
description: Defines product events, funnels, and activation metrics for the OpenClaw Railway Template
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Instrumenting-product-metrics Skill

Guides instrumentation of product analytics events, conversion funnels, and activation metrics within the OpenClaw Railway Template. Covers the key lifecycle moments — setup wizard completion, gateway startup, channel activation, and ongoing engagement — so product health can be measured without introducing external SDK dependencies into the lightweight Express wrapper.

## Quick Start

1. Identify the lifecycle stage to instrument (setup, onboarding, gateway, proxy traffic).
2. Locate the relevant handler in `src/server.js` or client-side in `src/public/setup-app.js`.
3. Emit a structured log event via `log.info("metrics", JSON.stringify({ event, ...props }))` — the ring buffer and `/logs` endpoint make these queryable.
4. For client-side funnel steps, dispatch a `fetch` to a `/setup/api/metrics` endpoint (auth-gated by `requireSetupAuth`) and record server-side.

## Key Concepts

**Activation funnel** — the critical path from first visit to a working AI assistant:
1. `setup.visited` — user lands on `/setup`
2. `setup.auth_provider_selected` — user picks GitHub/Google/etc.
3. `onboard.started` — `openclaw onboard --non-interactive` invoked
4. `onboard.completed` — onboard exits 0
5. `channel.configured` — at least one channel written via `config set --json`
6. `gateway.started` — child process spawned
7. `gateway.ready` — health poll succeeds
8. `proxy.first_request` — first proxied request reaches the gateway

**Engagement events** — post-activation signals:
- `tui.session_started` / `tui.session_ended` (with duration)
- `gateway.restart` (from debug console)
- `config.edited` / `config.reset`

**Token-stable redeploys** — instrument `gateway_token.source` (`env` | `disk` | `generated`) to track pairing stability across redeploys.

## Common Patterns

**Server-side structured event log:**
```js
// In src/server.js — emit after key lifecycle transitions
log.info("metrics", JSON.stringify({
  event: "gateway.ready",
  ms: Date.now() - gatewayStartTime,
  endpoint: readyEndpoint,
}));
```

**Funnel step from setup wizard (client → server):**
```js
// src/public/setup-app.js
await fetch("/setup/api/metrics", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ event: "setup.auth_provider_selected", provider }),
});
```

**Querying events from the ring buffer:**
```bash
curl -u admin:$SETUP_PASSWORD http://localhost:8080/logs \
  | grep '"event"' | jq -r '.event' | sort | uniq -c
```

**Activation rate calculation** — compare `onboard.completed` count to `setup.visited` count in logs over a time window to derive setup-to-activation conversion.

**TUI session duration metric:**
```js
// On session end in the ws handler (src/server.js)
log.info("metrics", JSON.stringify({
  event: "tui.session_ended",
  durationMs: Date.now() - session.startedAt,
  reason: closeReason,
}));
```