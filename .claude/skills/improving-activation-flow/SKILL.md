---
name: improving-activation-flow
description: Optimizes activation steps and time-to-value milestones for the OpenClaw Railway Template onboarding experience
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Improving-activation-flow Skill

This skill audits and improves the user activation flow in the OpenClaw Railway Template — from first visit at `/setup` through gateway readiness — reducing friction and accelerating time-to-value for new deployments.

## Quick Start

1. Read `src/server.js` to understand the current onboarding handler (`buildOnboardArgs`, `/setup/api/run`)
2. Read `src/public/setup.html` and `src/public/setup-app.js` for the wizard UI and client logic
3. Identify drop-off points: steps with unclear errors, missing feedback, or long waits
4. Apply targeted improvements to reduce steps, clarify state, or surface errors earlier

## Key Concepts

- **Lifecycle states**: The template has two states — *Unconfigured* (no `openclaw.json`) and *Configured* (gateway running). Activation means moving from the first to the second successfully.
- **Onboarding handler** (`/setup/api/run` in `src/server.js`): Runs `openclaw onboard --non-interactive`, writes channel configs via `openclaw config set --json`, force-sets gateway token auth, then spawns the gateway.
- **Gateway readiness** (`waitForGatewayReady`): Polls `/openclaw`, `/`, and `/health` endpoints — a key latency milestone. Timeout is 60 seconds.
- **Token stability**: `OPENCLAW_GATEWAY_TOKEN` must be stable across redeploys to avoid breaking device pairings. Check `${STATE_DIR}/gateway.token` persistence logic.
- **Channel config**: Written directly via `config set --json` (not `channels add`) to avoid CLI version incompatibilities.

## Common Patterns

### Reduce gateway wait time
Grep for `waitForGatewayReady` in `src/server.js` and review poll interval and endpoint order. Prioritize the endpoint most likely to respond first for the installed OpenClaw version.

```bash
grep -n "waitForGatewayReady\|poll\|health" src/server.js
```

### Surface setup errors earlier
Check the `/setup/api/run` handler for error responses. Ensure client-side `setup-app.js` renders error detail (not just a generic failure) so users can self-diagnose auth or config issues.

### Shorten the wizard step count
Review `setup.html` for optional fields that can be collapsed or deferred. Fields not required for initial activation (e.g., optional channels) should default to skipped.

### Validate environment before onboarding
Add a pre-flight check in the `/setup/api/run` handler — verify `STATE_DIR` is writable and `OPENCLAW_ENTRY` resolves before spawning the onboard process, so failures are caught with actionable messages.

```javascript
// Example pre-flight pattern in src/server.js
await fs.promises.access(STATE_DIR, fs.constants.W_OK);
await fs.promises.access(OPENCLAW_ENTRY, fs.constants.R_OK);
```

### Track time-to-value milestones
Add `log.info` timestamps at key activation milestones:
- Setup form submitted
- `openclaw onboard` completed
- Gateway spawned
- Gateway ready (first successful health poll)

This makes it easy to identify which step dominates activation time in production logs at `/logs`.