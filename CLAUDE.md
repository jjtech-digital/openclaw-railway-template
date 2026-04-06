# CLAUDE.md – OpenClaw Railway Template

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **Railway deployment wrapper** for **OpenClaw** (an AI coding assistant platform). It provides a browser-first onboarding experience without requiring SSH access.

**What it does:**
- Exposes OpenClaw gateway to the public internet via Express + HTTP reverse proxy
- Provides a `/setup` wizard (password-protected) for initial configuration
- Manages OpenClaw lifecycle: onboarding → gateway startup → traffic proxying
- Injects authentication tokens transparently so browser clients work without token knowledge
- Persists configuration and workspace to Railway volumes
- Provides logging, diagnostics, and device pairing tools

## Tech Stack

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| Runtime | Node.js | 24.x | Lightweight, proven stability |
| Framework | Express | 5.1.x | Minimal HTTP wrapper |
| Proxy | http-proxy | 1.18.x | Transparent request forwarding |
| WebSocket | ws | 8.18.x | Real-time communication channels |
| CLI Tools | node-pty | 1.0.x | Terminal emulation for TUI |
| Package Manager | pnpm | Latest | Fast, lock-based dependency management |

## Project Structure

```
openclaw-railway-template/
├── src/
│   ├── server.js              # Main Express app, proxy, gateway lifecycle
│   └── public/                # Static assets for /setup wizard and tools
│       ├── setup.html         # Setup wizard UI
│       ├── styles.css         # Shared styling
│       ├── setup-app.js       # Client-side JS for setup (vanilla, no build)
│       ├── tui.html           # Terminal UI (/tui)
│       ├── logs.html          # Live logs viewer (/logs)
│       └── loading.html       # Loading indicator
├── entrypoint.sh              # Docker entry point (Homebrew + user setup)
├── Dockerfile                 # Single-stage build (Node 22 base)
├── package.json               # Express + proxy dependencies
├── pnpm-lock.yaml             # Locked dependency versions
├── railway.toml               # Railway deployment config
├── CLAUDE.md                  # This file
├── CONTRIBUTING.md            # Dev guidelines
└── docs/
    ├── MIGRATION_FROM_MOLTBOT.md
    ├── OPENCLAW-VERSION-CONTROL.md
    └── STARTUP-IMPROVEMENTS.md
```

## Quick Start

### Prerequisites
- **Node.js 24+** (for local development)
- **OpenClaw installed globally** or `OPENCLAW_ENTRY` env var set
  ```bash
  # Install OpenClaw globally (required for local dev)
  npm install -g openclaw@latest
  ```
- **Docker** (for testing the full Docker build)

### Local Development

```bash
# Clone and install
git clone https://github.com/codetitlan/openclaw-railway-template.git
cd openclaw-railway-template
pnpm install  # or: npm install

# Set environment variables
export SETUP_PASSWORD=test
export OPENCLAW_STATE_DIR=/tmp/openclaw-test/.openclaw
export OPENCLAW_WORKSPACE_DIR=/tmp/openclaw-test/workspace
export OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)  # or any 64-char hex string

# Run the wrapper
npm run dev
# Visit http://localhost:8080/setup (password: test)
# Then visit http://localhost:8080/openclaw from the setup page
```

### Docker Testing

```bash
# Build the image
docker build -t openclaw-railway-template .

# Run locally with persistent volume
docker run --rm -p 8080:8080 \
  -e SETUP_PASSWORD=test \
  -e OPENCLAW_STATE_DIR=/data/.openclaw \
  -e OPENCLAW_WORKSPACE_DIR=/data/workspace \
  -v $(pwd)/.tmpdata:/data \
  openclaw-railway-template

# Access
# Setup: http://localhost:8080/setup (password: test)
# UI: http://localhost:8080/openclaw
# TUI: http://localhost:8080/tui (if ENABLE_WEB_TUI=true)
# Logs: http://localhost:8080/logs
```

## Available Commands

| Command | Purpose |
|---------|---------|
| `npm run dev` | Start development server (requires OpenClaw installed) |
| `npm start` | Start production server |
| `npm run lint` | Syntax check via `node -c src/server.js` |
| `docker build -t openclaw-railway-template .` | Build Docker image |

## Architecture

### Request Flow

```
User Browser
    ↓
Public URL (Railway: *.up.railway.app)
    ↓
Express Wrapper (PORT=8080)
    ├─→ /setup/*          → Setup wizard (Basic auth: SETUP_PASSWORD)
    ├─→ /logs             → Live logs viewer (Basic auth required)
    ├─→ /tui              → Terminal UI (Basic auth + session management)
    ├─→ /healthz          → Public health check
    └─→ /* (all others)   → Reverse proxy to internal gateway
         ↓
Internal Gateway (localhost:18789)
    └─→ [Wrapper injects Authorization: Bearer <token>]
```

### Lifecycle States

1. **Unconfigured**: No `openclaw.json` exists
   - All non-`/setup` routes redirect to `/setup`
   - User completes setup wizard → runs `openclaw onboard --non-interactive`

2. **Configured**: `openclaw.json` exists
   - Wrapper spawns `openclaw gateway run` as child process
   - Waits for gateway to respond on multiple health endpoints
   - Proxies all traffic with injected bearer token

### Key Files

- **src/server.js** (main entry): Express wrapper, proxy setup, gateway lifecycle management, configuration persistence (server logic only - no inline HTML/CSS)
- **src/public/** (static assets for setup wizard):
  - **setup.html**: Setup wizard HTML structure
  - **styles.css**: Setup wizard styling (extracted from inline styles)
  - **setup-app.js**: Client-side JS for `/setup` wizard (vanilla JS, no build step)
  - **tui.html, logs.html, loading.html**: Additional UI pages
- **entrypoint.sh**: Docker entry point (Homebrew persistence, user switching)
- **Dockerfile**: Single-stage build (installs OpenClaw via npm, installs wrapper deps)

### Environment Variables

**Required:**
- `SETUP_PASSWORD` — protects `/setup` wizard

**Recommended (Railway template defaults):**
- `OPENCLAW_STATE_DIR=/data/.openclaw` — config + credentials
- `OPENCLAW_WORKSPACE_DIR=/data/workspace` — agent workspace

**Optional:**
- `OPENCLAW_GATEWAY_TOKEN` — auth token for gateway (auto-generated if unset)
- `PORT` — wrapper HTTP port (default: 8080)
- `INTERNAL_GATEWAY_PORT` — gateway internal port (default: 18789)
- `INTERNAL_GATEWAY_HOST` — gateway bind address (default: 127.0.0.1)
- `OPENCLAW_ENTRY` — path to OpenClaw entry.js (default: `/usr/local/lib/node_modules/openclaw/dist/entry.js`)
- `OPENCLAW_NODE` — Node.js binary for running OpenClaw (default: `node`)
- `OPENCLAW_CONFIG_PATH` — custom config file location (default: `${STATE_DIR}/openclaw.json`)
- `ENABLE_WEB_TUI` — enable `/tui` endpoint (default: false)
- `TUI_IDLE_TIMEOUT_MS` — TUI session idle timeout (default: 300000 = 5 min)
- `TUI_MAX_SESSION_MS` — TUI max session duration (default: 1800000 = 30 min)
- `RAILWAY_PUBLIC_DOMAIN` — set automatically by Railway for CORS headers

### Authentication Flow

The wrapper manages a **two-layer auth scheme**:

1. **Setup wizard auth**: Basic auth with `SETUP_PASSWORD` (src/server.js:190)
2. **Gateway auth**: Bearer token (auto-generated or from `OPENCLAW_GATEWAY_TOKEN` env)
   - Token is auto-injected into proxied requests (src/server.js:736, src/server.js:741)
   - Persisted to `${STATE_DIR}/gateway.token` if not provided via env

**Gateway Token Persistence:**
The wrapper manages a stable gateway token across redeploys:
1. Check `OPENCLAW_GATEWAY_TOKEN` env var (for Railway secrets)
2. If not set: check `${STATE_DIR}/gateway.token` on disk
3. If not found: generate new one with `crypto.randomBytes(32).toString("hex")`
4. Persist to disk with restrictive permissions (mode 0o600)

**Why stable tokens matter:** Without them, every redeploy breaks existing device pairings. On Railway, set `OPENCLAW_GATEWAY_TOKEN` via Variables (use `${{ secret() }}` to generate securely).

### Onboarding Process

When the user runs setup (src/server.js:522-693):

1. Calls `openclaw onboard --non-interactive` with user-selected auth provider
2. Writes channel configs (Telegram/Discord/Slack) directly to `openclaw.json` via `openclaw config set --json`
3. Force-sets gateway config to use token auth + loopback bind + allowInsecureAuth
4. Spawns gateway process
5. Waits for gateway readiness (polls multiple endpoints)

**Important**: Channel setup bypasses `openclaw channels add` and writes config directly because `channels add` is flaky across different OpenClaw builds.

### Gateway Token Injection

The wrapper **always** injects the bearer token into proxied requests so browser clients don't need to know it:

- HTTP requests: via `proxy.on("proxyReq")` event handler (src/server.js:736)
- WebSocket upgrades: via `proxy.on("proxyReqWs")` event handler (src/server.js:741)

**Important**: Token injection uses `http-proxy` event handlers (`proxyReq` and `proxyReqWs`) rather than direct `req.headers` modification. Direct header modification does not reliably work with WebSocket upgrades, causing intermittent `token_missing` or `token_mismatch` errors.

This allows the Control UI at `/openclaw` to work without user authentication.

## Code Style & Conventions

### File Naming
- Main server: `server.js` (camelCase for JS files)
- Public assets: `setup.html`, `setup-app.js` (kebab-case + extension)
- Style: `styles.css`
- Scripts: `entrypoint.sh`

### Code Naming
- **Functions:** camelCase (`startGateway`, `waitForGatewayReady`)
- **Variables:** camelCase (`gatewayProc`, `stateDir`)
- **Constants:** SCREAMING_SNAKE_CASE (`STATE_DIR`, `GATEWAY_TARGET`, `MAX_LOG_FILE_SIZE`)
- **Middleware:** camelCase, descriptive (`requireSetupAuth`)

### Import Order
```javascript
// 1. Built-in Node.js modules (alphabetical)
import childProcess from "node:child_process";
import crypto from "node:crypto";
import fs from "node:fs";
import path from "node:path";

// 2. External packages (alphabetical)
import express from "express";
import httpProxy from "http-proxy";
import pty from "node-pty";
import { WebSocketServer } from "ws";
```

### Logging
- Use `log.info()`, `log.warn()`, `log.error()` (not `console.log`)
- Always include a category: `log.info("gateway", "message")`
- **Never log secrets** (tokens, passwords) unless behind `OPENCLAW_TEMPLATE_DEBUG=true`
- Format: `[timestamp] [level] [category] message`
- Ring buffer keeps last 1000 lines in memory for `/logs` endpoint

### Error Handling
- Use `try-catch` for async operations
- Log errors with context: `catch(err) { log.error("context", err.message) }`
- Graceful degradation: non-critical errors log but don't crash
- Proxy errors: caught and logged, client sees 502/503

### Configuration
- All config via environment variables (no config files)
- Defaults provided for optional variables
- No secrets in code (use Railway Variables / .env for secrets)

## Common Development Tasks

### Testing the setup wizard

1. Delete `${STATE_DIR}/openclaw.json` (or run Reset in the UI)
2. Visit `/setup` and complete onboarding
3. Check logs for gateway startup and channel config writes

### Testing authentication

- Setup wizard: Clear browser auth, verify Basic auth challenge
- Gateway: Remove `Authorization` header injection (src/server.js:736) and verify requests fail

### Debugging gateway startup

Check logs for:
- `[gateway] starting with command: ...` (src/server.js:247)
- `[gateway] ready at <endpoint>` (src/server.js:193)
- `[gateway] failed to become ready after 60000ms` (src/server.js:207)

If gateway doesn't start:
- Verify `openclaw.json` exists and is valid JSON
- Check `STATE_DIR` and `WORKSPACE_DIR` are writable
- Ensure bearer token is set in config

### Modifying onboarding args

Edit `buildOnboardArgs()` (src/server.js) to add new CLI flags or auth providers.

### Adding new channel types

1. Add channel-specific fields to `/setup` HTML (src/public/setup.html)
2. Add config-writing logic in `/setup/api/run` handler (src/server.js)
3. Update client JS to collect the fields (src/public/setup-app.js)

### Adding new debug console commands

Debug console is exposed at `/setup` → "Tools" → "Debug Console" with pre-approved commands.

**To add a new command:**

1. **Add to allowedCommands** in `src/server.js`:
   ```javascript
   const allowedCommands = new Set([
     "gateway.restart",
     "openclaw.your.command",  // ← Add this
   ]);
   ```

2. **Add handler** in the POST `/setup/api/console` switch statement:
   ```javascript
   case "openclaw.your.command": {
     if (!args?.trim()) {
       return res.status(400).json({ ok: false, error: "Argument required" });
     }
     const result = await runCmd(
       OPENCLAW_NODE,
       clawArgs(["your", "command", args.trim()])
     );
     return res.json({
       ok: result.code === 0,
       output: redactSecrets(result.output),
       exitCode: result.code,
     });
   }
   ```

3. **Add UI option** in `src/public/setup.html`:
   ```html
   <optgroup label="Your Category">
     <option value="openclaw.your.command">openclaw your command &lt;arg&gt;</option>
   </optgroup>
   ```

## Testing Requirements

Before submitting a PR:

1. **Syntax validation:**
   ```bash
   npm run lint
   ```

2. **Fresh setup flow:**
   - Delete config/workspace: `rm -rf ${OPENCLAW_STATE_DIR} ${OPENCLAW_WORKSPACE_DIR}`
   - Run through `/setup` wizard completely
   - Verify all channels work (if enabled)

3. **Debug console:**
   - Test each debug command with valid/invalid inputs
   - Verify error handling

4. **Config editor:**
   - Load config, modify, save
   - Verify backup creation
   - Test with invalid JSON (error handling)

5. **Docker build:**
   ```bash
   docker build -t openclaw-railway-template .
   ```
   Should complete without errors.

6. **Health endpoints:**
   - `/healthz` returns status
   - `/setup/api/debug` shows diagnostic info

7. **Railway deployment (if modifying deployment logic):**
   - Test on Railway staging environment
   - Verify persistence: redeploy and check if config persists
   - Check Docker build time (Homebrew installation is slow on first build)

## Railway Deployment Notes

- Template must mount a volume at `/data`
- Must set `SETUP_PASSWORD` in Railway Variables
- Public networking must be enabled (assigns `*.up.railway.app` domain)
- OpenClaw is installed via `npm install -g openclaw@2026.3.13 clawhub@latest` during Docker build
- Homebrew is installed for additional CLI tools (git, curl, etc.) and persisted to volume
- Health check configured to `/setup/healthz` endpoint

## Security Considerations

**Authentication:**
- Setup wizard: HTTP Basic auth (not secrets-safe, okay for internal use)
- Gateway: Bearer token in `Authorization` header
- Token never logged (redacted in logs)

**Authorization:**
- `/setup/*` — only accessible with correct `SETUP_PASSWORD`
- `/openclaw/*` — proxied with automatic token injection
- `/logs` — requires setup auth
- `/tui` — requires setup auth + session management

**Secrets Management:**
- `SETUP_PASSWORD` — set via Railway Variables or `.env`
- `OPENCLAW_GATEWAY_TOKEN` — set via Railway Variables
- Never log tokens/passwords (use `redactSecrets()` if needed)
- File permissions: `gateway.token` created with mode 0o600

**Deployment Security:**
- No hardcoded credentials in code
- HTTPS enforced in browser (Railway handles TLS)
- CORS headers set via `RAILWAY_PUBLIC_DOMAIN`
- CSP headers recommended (implement in proxy if needed)

## Quirks & Gotchas

1. **Gateway token must be stable across redeploys** → persisted to volume if not in env
2. **Channels are written via `config set --json`, not `channels add`** → avoids CLI version incompatibilities
3. **Gateway readiness check polls multiple endpoints** (`/openclaw`, `/`, `/health`) → some builds only expose certain routes
4. **Discord bots require MESSAGE CONTENT INTENT** → must be enabled in Discord Developer Portal
5. **Gateway spawn inherits stdio** → logs appear in wrapper output
6. **WebSocket auth requires proxy event handlers** → Direct `req.headers` modification doesn't work for WebSocket upgrades with http-proxy; must use `proxyReqWs` event to reliably inject Authorization header
7. **Control UI requires allowInsecureAuth to bypass pairing** → Set `gateway.controlUi.allowInsecureAuth=true` during onboarding to prevent "disconnected (1008): pairing required" errors (GitHub issue #2284). Wrapper already handles bearer token auth, so device pairing is unnecessary.
8. **Homebrew installation in Docker is slow** → Full install takes 2-3 minutes, cached in Docker layer, volume persistence handles cross-redeploy state
9. **`OPENCLAW_ENTRY` path changed across versions** → Backward-compat symlink `/openclaw/dist` created in Dockerfile for pre-npm versions

## Serena Semantic Coding

This project has been onboarded with **Serena** (semantic coding assistant via MCP). Comprehensive memory files are available covering:

- Project overview and architecture
- Tech stack and codebase structure
- Code style and conventions
- Development commands and task completion checklist
- Quirks and gotchas

**When working on tasks:**
1. Check `mcp__serena__check_onboarding_performed` first to see available memories
2. Read relevant memory files before diving into code (e.g., `mcp__serena__read_memory`)
3. Use Serena's semantic tools for efficient code exploration:
   - `get_symbols_overview` - Get high-level file structure without reading entire file
   - `find_symbol` - Find classes, functions, methods by name path
   - `find_referencing_symbols` - Understand dependencies and usage
4. Prefer symbolic editing (`replace_symbol_body`, `insert_after_symbol`) for precise modifications

This avoids repeatedly reading large files and provides instant context about the project.

## Additional Resources

- **[README.md](./README.md)** — User guide and features overview
- **[CONTRIBUTING.md](./CONTRIBUTING.md)** — Developer contribution guidelines
- **[docs/MIGRATION_FROM_MOLTBOT.md](./docs/MIGRATION_FROM_MOLTBOT.md)** — Upgrade guide from old version
- **[docs/OPENCLAW-VERSION-CONTROL.md](./docs/OPENCLAW-VERSION-CONTROL.md)** — OpenClaw version management
- **[docs/STARTUP-IMPROVEMENTS.md](./docs/STARTUP-IMPROVEMENTS.md)** — Startup optimization notes
- **OpenClaw Docs:** https://docs.openclaw.ai
- **Railway Docs:** https://docs.railway.app


## Skill Usage Guide

When working on tasks involving these technologies, invoke the corresponding skill:

| Skill | Invoke When |
|-------|-------------|
| node | Provides Node.js runtime environment and built-in modules |
| ws | Implements WebSocket server for real-time communication |
| express | Enables HTTP server routing, middleware, and request handling |
| http-proxy | Provides HTTP reverse proxy and request forwarding |
| node-pty | Implements terminal emulation for interactive TUI features |
| bash | Provides shell scripts for Docker entrypoint and setup |
| pnpm | Manages package dependencies and lock file versioning |
| mapping-user-journeys | Maps in-app journeys and identifies friction points in code |
| improving-activation-flow | Optimizes activation steps and time-to-value milestones |
| crafting-empty-states | Creates empty states and onboarding affordances |
| docker | Defines containerized deployment and multi-stage builds |
| orchestrating-feature-adoption | Plans feature discovery, nudges, and adoption flows |
| running-product-experiments | Sets up product experiments and rollout checks |
| designing-inapp-guidance | Builds tooltips, tours, and contextual guidance |
| triaging-user-feedback | Routes feedback into backlog and quick wins |
| structuring-offer-ladders | Frames plan tiers, value ladders, and upgrade logic |
| crafting-page-messaging | Writes conversion-focused messaging for pages and key CTAs |
| generating-growth-hypotheses | Generates channel experiments and growth loops |
| embedding-decision-cues | Applies behavioral cues that improve conversion decisions |
| writing-release-notes | Drafts release notes tied to shipped features |
| instrumenting-product-metrics | Defines product events, funnels, and activation metrics |
| tightening-brand-voice | Refines copy for clarity, tone, and consistency |
| tuning-landing-journeys | Improves landing page flow, hierarchy, and conversion paths |
| planning-editorial-arcs | Defines content themes, briefs, and editorial cadence |
| framing-release-stories | Builds launch narratives, assets, and rollout checklists |
| streamlining-signup-steps | Reduces friction in signup and trial activation |
| accelerating-first-run | Improves onboarding sequence and time-to-value |
| reducing-form-falloff | Improves lead capture forms to reduce drop-off |
| strengthening-upgrade-moments | Improves upgrade prompts and paywall messaging |
| refining-prompt-surfaces | Optimizes banners, modals, and in-app prompts |
| calibrating-paid-campaigns | Aligns paid acquisition with landing pages and pixels |
| building-acquisition-tools | Designs lead magnets or free tools for acquisition |
| engineering-referral-loops | Designs referral or partner loop mechanics |
| designing-lifecycle-messages | Designs onboarding and lifecycle email sequences |
| inspecting-search-coverage | Audits technical and on-page search coverage |
| designing-variation-tests | Plans A/B experiments and measurement plans |
| scaling-template-pages | Builds scalable, template-driven search pages |
| adding-structured-signals | Adds structured data for rich results |
| building-compare-hubs | Creates comparison and alternative pages for discovery |
