---
name: triaging-user-feedback
description: Routes user feedback into backlog items and quick wins for the OpenClaw Railway Template project
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Triaging-user-feedback Skill

This skill helps categorize and route incoming user feedback for the OpenClaw Railway Template into two buckets: **quick wins** (low-effort, high-impact fixes addressable in a single PR) and **backlog items** (larger features, architectural changes, or issues requiring deeper investigation). It examines existing issues against the codebase to assess scope and effort.

## Quick Start

1. Collect the raw feedback (bug reports, feature requests, UX complaints).
2. Search the codebase for the affected area using `Grep` and `Glob`.
3. Classify each item using the criteria in Key Concepts below.
4. Output a triage summary grouping items by category with a one-line rationale each.

## Key Concepts

**Quick Win** — meets ALL of these:
- Change is isolated to one or two files (e.g., `src/server.js`, `src/public/setup.html`, `src/public/setup-app.js`, `src/public/styles.css`)
- No new dependencies required
- No environment variable or Railway volume changes needed
- Verifiable with `npm run lint` and a manual setup flow test

**Backlog Item** — meets ANY of these:
- Touches lifecycle state transitions (`Unconfigured → Configured`, gateway spawn, proxy setup)
- Requires a new environment variable, Docker layer change, or `entrypoint.sh` modification
- Involves auth changes (Basic auth, bearer token injection, `proxyReq`/`proxyReqWs` handlers)
- Needs Railway deployment validation (volume persistence, redeploy stability)
- Depends on OpenClaw CLI behavior that varies across builds (e.g., `channels add` flakiness)

**Known Sensitive Areas** (always backlog):
- Gateway token stability across redeploys (`STATE_DIR/gateway.token`)
- WebSocket auth injection (`proxyReqWs` event handler)
- `allowInsecureAuth` / control UI pairing bypass
- Homebrew persistence layer in Docker

## Common Patterns

**Bug in setup wizard UI** → Quick win. Check `src/public/setup.html` and `src/public/setup-app.js`. Verify no server-side handler changes needed.

**Onboarding step fails for a specific auth provider** → Backlog. Involves `buildOnboardArgs()` in `src/server.js` and possible OpenClaw CLI version differences.

**New debug console command requested** → Quick win if it maps to an existing `openclaw` CLI command. Add to `allowedCommands` set, add a `case` in the POST `/setup/api/console` handler, and add an `<option>` in `setup.html`.

**Gateway doesn't start after redeploy** → Backlog. Investigate token persistence flow, `openclaw.json` validity, and `OPENCLAW_ENTRY` path compatibility across OpenClaw versions.

**Styling or copy change in `/setup`** → Quick win. Edit `src/public/styles.css` or `src/public/setup.html` directly.

**New channel type support** → Backlog. Requires coordinated changes across `setup.html`, `setup-app.js`, and the `/setup/api/run` handler in `src/server.js`, plus validation against live OpenClaw config schema.