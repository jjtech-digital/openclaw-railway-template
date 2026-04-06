# Distribution

## When to use
Apply when deciding how to split traffic between variants, assign cohorts without server-side sessions, or ensure consistent variant exposure across page reloads.

## Patterns

### Cookie-Based Cohort Assignment
Assign variant on first `/setup` GET request; set a short-lived cookie (`openclaw-variant=A|B`). Read cookie in subsequent requests to maintain cohort consistency. Log assignment with `log.info("setup", "variant assigned", { variant })`.

### Hash-Based Deterministic Split
Derive variant from a hash of a stable identifier (e.g., hashed IP + user-agent) for stateless assignment. Useful when cookies are cleared between sessions. No server state required.

### Percentage Rollout via Env Var
Use `VARIANT_ROLLOUT_PCT` environment variable (e.g., `10` for 10% in variant). Check at request time: `Math.random() < pct / 100`. Allows gradual ramp without redeployment — change the Railway Variable value.

## Pitfalls
- Railway redeploys reset in-memory state. Never store cohort assignments in a module-level variable; use cookies or deterministic hashing.
- Unequal traffic splits (e.g., 90/10) require proportionally larger sample sizes for the minority group — recalculate power before launching.