# Roadmap & Experiments Reference

## When to use
Use when planning A/B tests, feature flags, or rollout checks within the setup wizard or gateway lifecycle — specifically to validate whether a new onboarding step or UI change improves activation rates.

## Patterns

**Feature flag via environment variable**
```js
// src/server.js — guard experimental behaviour behind an env flag
const ENABLE_CHANNEL_SELECTOR_V2 = process.env.ENABLE_CHANNEL_SELECTOR_V2 === "true";

if (ENABLE_CHANNEL_SELECTOR_V2) {
  log.info("metrics", JSON.stringify({ event: "experiment.variant", name: "channel_selector_v2" }));
}
```

**Experiment exposure event**
```js
// Emit once per session when a user enters the experimental flow
log.info("metrics", JSON.stringify({
  event: "experiment.exposed",
  experiment: "simplified_onboard",
  variant: "control" | "treatment",
}));
```

**Comparing variants in logs**
```bash
curl -u admin:$SETUP_PASSWORD http://localhost:8080/logs \
  | grep '"experiment"' \
  | jq -r '[.experiment, .variant, .event] | @tsv' \
  | sort | uniq -c
```

## Pitfalls
- Railway environment variables are set at deploy time, not per-request — experiment variants cannot be toggled per user without additional state (e.g., a cookie or session flag written during setup). Keep experiments coarse-grained (entire deployment) unless session state is introduced deliberately.
- Remove experiment instrumentation once a variant is fully rolled out. Stale experiment events in the ring buffer mislead future analysis.