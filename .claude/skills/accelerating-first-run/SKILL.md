---
name: accelerating-first-run
description: Improves onboarding sequence and time-to-value for the OpenClaw Railway Template by reducing friction in the /setup wizard, gateway startup, and first-run configuration flow.
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Accelerating-first-run Skill

This skill audits and improves the time from first deployment to a working OpenClaw gateway — covering the `/setup` wizard UX, onboarding process in `src/server.js`, gateway readiness polling, and channel configuration writes.

## Quick Start

1. Read `src/server.js` around the onboarding handler (`buildOnboardArgs`, `/setup/api/run`, `waitForGatewayReady`) to understand the current flow.
2. Read `src/public/setup.html` and `src/public/setup-app.js` to audit wizard UX and form field order.
3. Identify the longest blocking steps: `openclaw onboard --non-interactive`, gateway readiness polling, and channel config writes.
4. Apply improvements to reduce wall-clock time and perceived latency.

## Key Concepts

**Onboarding sequence** (`src/server.js:522–693`): Calls `openclaw onboard --non-interactive`, writes channel configs via `openclaw config set --json`, force-sets gateway token/bind config, then spawns the gateway and polls for readiness.

**Gateway readiness polling** (`waitForGatewayReady`): Polls `/openclaw`, `/`, and `/health` endpoints until one responds — timeout is 60 seconds. Slow starts are common; polling interval and parallelism matter here.

**Channel config writes**: Done via `openclaw config set --json` directly into `openclaw.json`, bypassing `channels add` (which is flaky). This is the correct pattern — don't revert it.

**Token stability**: Gateway token is loaded from env → disk → generated. Stable tokens prevent broken device pairings across redeploys. Don't alter this resolution order.

**Setup wizard** (`src/public/setup.html` + `setup-app.js`): Vanilla JS, no build step. Progress indicators, field validation, and early error feedback directly reduce perceived setup time.

## Common Patterns

**Parallelize config writes**: If multiple channels are configured, write their configs concurrently rather than sequentially before spawning the gateway.

**Fail fast on bad credentials**: Validate auth provider tokens before running the full `openclaw onboard` command. Surface errors in the wizard immediately instead of after a long CLI timeout.

**Reduce polling interval for fast environments**: `waitForGatewayReady` can use an adaptive backoff — start at 200ms, increase to 2s — rather than a fixed interval, to catch fast starts sooner.

**Show incremental progress in wizard**: Break the single "Setting up..." state into labeled steps (Authenticating → Configuring channels → Starting gateway → Ready) so users see forward motion and don't abandon the flow.

**Pre-validate required env vars**: At wizard load time, call `/setup/api/debug` to check that `STATE_DIR` and `WORKSPACE_DIR` are writable before the user submits the form. Surface missing-volume errors before the long onboard command runs.

**Skip redundant health checks**: If `openclaw.json` already exists and the gateway is running, skip the onboard step entirely and redirect to `/openclaw` — avoid re-running setup on page refresh.