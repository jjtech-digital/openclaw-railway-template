---
name: writing-release-notes
description: Drafts release notes tied to shipped features for the OpenClaw Railway Template project
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Writing-release-notes Skill

Generates structured release notes by inspecting git history, changed files, and commit messages to produce user-facing summaries of shipped features, fixes, and breaking changes for the OpenClaw Railway Template.

## Quick Start

1. Run `git log --oneline <prev-tag>..HEAD` to collect commits since the last release.
2. For each commit, identify the affected layer: gateway lifecycle, setup wizard, proxy/auth, Docker/entrypoint, or environment config.
3. Group changes by category (Features, Fixes, Breaking Changes, Internal).
4. Draft notes in plain language aimed at Railway deployers, not Node.js internals.

## Key Concepts

- **Layers to watch:** `src/server.js` (gateway lifecycle, proxy, auth), `src/public/` (setup wizard UI), `entrypoint.sh` / `Dockerfile` (container behavior), `railway.toml` (deployment config).
- **Audience:** Self-hosted Railway users who interact via `/setup`, `/logs`, `/tui`, and `/openclaw` — not contributors reading source.
- **Breaking changes:** Any change to required env vars (`SETUP_PASSWORD`, `OPENCLAW_GATEWAY_TOKEN`), volume mount paths (`/data`), or gateway token persistence behavior warrants a prominent callout.
- **Stable token note:** Changes affecting `gateway.token` persistence or `OPENCLAW_GATEWAY_TOKEN` handling must include a migration note — broken tokens break all existing device pairings.

## Common Patterns

**Feature entry:**
```
### Browser-based setup wizard
Users can now complete full OpenClaw onboarding at `/setup` without SSH access. Channels (Telegram, Discord, Slack) are configured directly through the UI.
```

**Fix entry:**
```
### WebSocket auth no longer intermittently fails
Token injection now uses `proxyReqWs` event handlers instead of direct header modification, resolving `token_missing` errors on WebSocket upgrades.
```

**Breaking change entry:**
```
### ⚠ Volume mount path changed to `/data`
Update your Railway volume attachment from the previous path. Existing `openclaw.json` and `gateway.token` must be migrated manually.
```

**Scanning changed files for scope:**
```bash
git diff --name-only <prev-tag>..HEAD
```

**Pulling commit messages for a range:**
```bash
git log --pretty=format:"%h %s" <prev-tag>..HEAD
```