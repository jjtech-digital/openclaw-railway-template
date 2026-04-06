# Measurement & Testing

## When to use
When defining what "successful release" looks like in verifiable terms, and when building the smoke-test section of the rollout checklist. Also use when deciding which post-deploy signals confirm the release narrative is accurate.

## Patterns

**Health signal hierarchy**
For each release, identify the single health signal that proves the headline outcome:
- Token persistence release → confirm device pairing survives a manual Railway redeploy
- Gateway proxy change → confirm `GET /openclaw` returns 200 with injected auth header
- Setup wizard change → complete `/setup` end-to-end with a fresh `openclaw.json` delete

Document this signal explicitly in the rollout checklist as the final smoke test.

**Regression surface for rollout checklists**
Every checklist should cover the three regression surfaces regardless of what changed:
1. Auth layer: Basic auth challenge on `/setup`, bearer injection on proxy
2. Persistence layer: Config survives redeploy, gateway token stable
3. Proxy layer: WebSocket upgrade works (`/openclaw` Control UI connects)

**Pass/fail criteria over subjective checks**
Write "GET /healthz returns `{"status":"ok"}`" not "verify the app is healthy." Operators running checklists need binary outcomes.

## Pitfalls
Do not add smoke tests for surfaces the release did not touch. Checklist bloat causes operators to skip steps. Keep post-deploy smoke tests to ≤8 items; move comprehensive regression to CI.