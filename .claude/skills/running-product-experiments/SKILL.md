---
name: running-product-experiments
description: Sets up product experiments and rollout checks in the OpenClaw Railway Template
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Running-product-experiments Skill

This skill guides incremental feature rollouts and product experiments in the OpenClaw Railway Template. It covers gating new behavior behind environment variables, validating rollout state via health and debug endpoints, and safely toggling features without redeploying the full stack.

## Quick Start

1. Define a new experiment flag as an environment variable (e.g., `ENABLE_MY_FEATURE=true`)
2. Read the flag in `src/server.js` alongside existing optional vars (e.g., `ENABLE_WEB_TUI`)
3. Gate the feature behind the flag using a conditional block
4. Expose rollout state in `/setup/api/debug` so operators can verify activation
5. Test via `/healthz` and `/setup/api/debug` before enabling in production Railway Variables

## Key Concepts

- **Feature flags via env vars** — All optional behavior is controlled through environment variables. No config files, no feature-flag services. Set variables in Railway → Variables panel or `.env` locally.
- **Debug endpoint as rollout check** — `/setup/api/debug` (`src/server.js`) returns current runtime state. Add new experiment flags to this response to make rollout status observable without SSH access.
- **Lifecycle gate** — Experiments that affect gateway startup must be gated before `startGateway()` is called. Experiments that affect proxying must be gated in `proxyReq`/`proxyReqWs` handlers.
- **Logging rollout state** — Use `log.info("experiment", "feature X enabled")` on startup so the `/logs` viewer confirms activation.
- **Redact secrets** — If an experiment touches auth tokens or passwords, always pass output through `redactSecrets()` before logging or returning to the client.

## Common Patterns

**Gating a new route experiment:**
```js
const ENABLE_MY_FEATURE = process.env.ENABLE_MY_FEATURE === "true";

if (ENABLE_MY_FEATURE) {
  app.get("/my-feature", requireSetupAuth, (req, res) => { ... });
}
```

**Exposing rollout state in the debug endpoint:**
```js
// Inside the /setup/api/debug handler in src/server.js
experimentMyFeature: ENABLE_MY_FEATURE,
```

**Logging activation on startup:**
```js
if (ENABLE_MY_FEATURE) {
  log.info("experiment", "my-feature enabled via ENABLE_MY_FEATURE");
}
```

**Verifying rollout without redeploying:**
```bash
curl -u admin:$SETUP_PASSWORD https://<railway-domain>/setup/api/debug | jq .experimentMyFeature
```

**Rolling back** — Unset the Railway Variable and redeploy. No code change required if the flag defaults to `false`/`undefined`.