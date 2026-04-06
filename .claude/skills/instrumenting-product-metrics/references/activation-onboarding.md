# Activation & Onboarding Metrics

## When to use
Instrument these events when measuring how many users complete the setup wizard and reach a working gateway — the primary activation funnel for the OpenClaw Railway Template.

## Patterns

**Funnel step emission (server-side)**
```js
// src/server.js — after onboard exits 0
log.info("metrics", JSON.stringify({
  event: "onboard.completed",
  authProvider,
  durationMs: Date.now() - onboardStartTime,
}));
```

**Funnel step emission (client → server)**
```js
// src/public/setup-app.js — when user selects auth provider
await fetch("/setup/api/metrics", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ event: "setup.auth_provider_selected", provider }),
});
```

**Activation rate query**
```bash
# Compare completed onboards to setup visits
curl -u admin:$SETUP_PASSWORD http://localhost:8080/logs \
  | grep '"event"' \
  | jq -r '.event' \
  | sort | uniq -c
```

## Pitfalls
- `onboard.completed` only fires when `openclaw onboard --non-interactive` exits 0. A non-zero exit with partial config written can mislead funnel counts — always pair it with an `onboard.failed` event carrying the exit code.
- Do not emit funnel events from the client without gating them behind `requireSetupAuth`; unauthenticated POSTs to `/setup/api/metrics` would pollute counts.