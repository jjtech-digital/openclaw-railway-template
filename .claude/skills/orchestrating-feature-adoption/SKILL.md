---
name: orchestrating-feature-adoption
description: Plans feature discovery, nudges, and adoption flows for the OpenClaw Railway Template — maps where users enter new features, identifies friction, and proposes in-UI nudges or wizard steps.
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Orchestrating-feature-adoption Skill

This skill guides planning and implementing feature discovery flows in the OpenClaw Railway Template. It analyzes the existing setup wizard, `/setup` UI pages, and server-side lifecycle states to identify where users can be nudged toward underused features (TUI, logs viewer, device pairing, debug console) and proposes concrete adoption patterns grounded in the codebase.

## Quick Start

1. Identify the target feature and its entry point (route, UI element, or config flag).
2. Locate the relevant files: `src/public/setup.html`, `src/public/setup-app.js`, `src/server.js`.
3. Determine the lifecycle state where the nudge fits: unconfigured (`/setup`) or configured (post-gateway).
4. Add discovery UI (banner, tooltip, callout) or wizard step without breaking existing flows.
5. Validate with `npm run lint` and a fresh setup run.

## Key Concepts

- **Lifecycle gates**: Features are only reachable after specific states. `/tui` and `/logs` require `ENABLE_WEB_TUI=true` and setup auth; nudges must respect these gates.
- **Setup wizard as adoption surface**: `setup.html` and `setup-app.js` are the primary surfaces for first-run nudges. New steps or callouts go here.
- **Server-side feature flags**: Optional features are toggled via env vars (`ENABLE_WEB_TUI`, `RAILWAY_PUBLIC_DOMAIN`). Adoption flows should check these before surfacing nudges.
- **Debug console discoverability**: The Tools → Debug Console in `/setup` is the least-discovered surface. Adding `<option>` entries in `setup.html` and handlers in `server.js` is the adoption path.
- **Config-driven defaults**: Features like pairing and TUI can be pre-enabled during onboarding by writing to `openclaw.json` via `openclaw config set --json`.

## Common Patterns

### Surface a nudge after setup completes
Add a post-completion callout in `setup-app.js` triggered when the setup API returns `ok: true`, pointing users to `/tui` or `/logs`.

### Add a wizard step for an optional feature
Insert a new `<section>` step in `setup.html` with a skip option. Wire its data in `setup-app.js` and handle the resulting config write in the `/setup/api/run` handler in `server.js`.

### Enable a feature by default during onboarding
In `buildOnboardArgs()` (or the config-write block in `/setup/api/run`), add a `openclaw config set --json` call that sets the relevant config key so the feature is active on first gateway start.

### Add a debug console command for a new feature
1. Add the command string to `allowedCommands` in `server.js`.
2. Add a `case` handler in the `/setup/api/console` switch.
3. Add an `<option>` under the appropriate `<optgroup>` in `setup.html`.

### Detect first-visit vs. returning user
Check for the existence of `openclaw.json` at `OPENCLAW_CONFIG_PATH` (readable from `src/server.js` state) to branch nudge logic — show discovery prompts only to first-time visitors.