---
name: docker
description: Defines containerized deployment and multi-stage builds for the OpenClaw Railway Template
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Docker Skill

Manages containerized builds and local testing for the OpenClaw Railway Template. The project uses a single-stage `Dockerfile` based on Node 22 that installs OpenClaw via npm, sets up Homebrew, and runs the Express wrapper via `entrypoint.sh`.

## Quick Start

```bash
# Build the image
docker build -t openclaw-railway-template .

# Run with persistent volume
docker run --rm -p 8080:8080 \
  -e SETUP_PASSWORD=test \
  -e OPENCLAW_STATE_DIR=/data/.openclaw \
  -e OPENCLAW_WORKSPACE_DIR=/data/workspace \
  -v $(pwd)/.tmpdata:/data \
  openclaw-railway-template
```

Access points after running:
- Setup wizard: `http://localhost:8080/setup` (password: `test`)
- Control UI: `http://localhost:8080/openclaw`
- Logs: `http://localhost:8080/logs`

## Key Concepts

- **Single-stage build** — Node 22 base image; no multi-stage separation needed since there is no compile step
- **Volume at `/data`** — Required for persisting `openclaw.json`, `gateway.token`, and workspace across container restarts and Railway redeploys
- **Homebrew layer** — Installed during build for CLI tooling; slow (~2–3 min) but cached in Docker layer and persisted to volume
- **entrypoint.sh** — Handles Homebrew persistence and user switching before starting the Node server
- **Backward-compat symlink** — `/openclaw/dist` symlinked in Dockerfile for pre-npm OpenClaw versions that expect the old `OPENCLAW_ENTRY` path
- **OpenClaw install** — Installed globally via `npm install -g openclaw@<version> clawhub@latest` during Docker build

## Common Patterns

### Rebuilding after dependency changes
```bash
docker build --no-cache -t openclaw-railway-template .
```

### Inspecting container state
```bash
docker run --rm -it \
  -v $(pwd)/.tmpdata:/data \
  openclaw-railway-template bash
```

### Verifying health endpoint
```bash
curl http://localhost:8080/healthz
```

### Pinning the OpenClaw version
Edit the `npm install -g openclaw@<version>` line in `Dockerfile`. See `docs/OPENCLAW-VERSION-CONTROL.md` for version management strategy.

### Environment variables for local Docker runs
| Variable | Example value | Purpose |
|---|---|---|
| `SETUP_PASSWORD` | `test` | Protects `/setup` wizard |
| `OPENCLAW_STATE_DIR` | `/data/.openclaw` | Config and credentials |
| `OPENCLAW_WORKSPACE_DIR` | `/data/workspace` | Agent workspace |
| `OPENCLAW_GATEWAY_TOKEN` | *(hex string)* | Stable auth token across redeploys |
| `ENABLE_WEB_TUI` | `true` | Enables `/tui` terminal endpoint |