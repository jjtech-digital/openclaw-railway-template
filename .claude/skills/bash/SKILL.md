---
name: bash
description: Provides shell scripts for Docker entrypoint and setup in the OpenClaw Railway Template
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Bash Skill

Assists with shell scripting for the OpenClaw Railway Template, covering `entrypoint.sh` (Docker entry point handling Homebrew persistence and user switching) and related setup scripts that bootstrap the Express wrapper and gateway lifecycle.

## Quick Start

```bash
# Validate server.js syntax
npm run lint

# Build Docker image
docker build -t openclaw-railway-template .

# Run with local volume
docker run --rm -p 8080:8080 \
  -e SETUP_PASSWORD=test \
  -e OPENCLAW_STATE_DIR=/data/.openclaw \
  -e OPENCLAW_WORKSPACE_DIR=/data/workspace \
  -v $(pwd)/.tmpdata:/data \
  openclaw-railway-template
```

## Key Concepts

- **entrypoint.sh**: Docker entry point that installs/restores Homebrew from the `/data` volume and switches to the appropriate user before starting the Node wrapper.
- **Volume persistence**: `/data` is the Railway-mounted volume; state dir (`/data/.openclaw`) and workspace (`/data/workspace`) must be writable.
- **Gateway token file**: Written to `${STATE_DIR}/gateway.token` with mode `0o600`; never logged or echoed.
- **Homebrew layer caching**: Full install takes 2–3 minutes; cached in Docker layer, volume-persisted across redeploys.

## Common Patterns

### Generate a gateway token
```bash
openssl rand -hex 32
```

### Reset state for a fresh setup flow
```bash
rm -rf ${OPENCLAW_STATE_DIR} ${OPENCLAW_WORKSPACE_DIR}
```

### Check gateway token file permissions
```bash
ls -la ${OPENCLAW_STATE_DIR}/gateway.token
# Should show: -rw------- (0600)
```

### Tail wrapper logs during local dev
```bash
npm run dev 2>&1 | tee /tmp/openclaw-dev.log
```

### Verify entrypoint script syntax
```bash
bash -n entrypoint.sh
```