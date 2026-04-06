# Docker Reference

## When to use
Use these patterns when building, testing, or debugging the OpenClaw Railway Template container locally before pushing to Railway.

## Patterns

### Standard build and run
```bash
docker build -t openclaw-railway-template .

docker run --rm -p 8080:8080 \
  -e SETUP_PASSWORD=test \
  -e OPENCLAW_STATE_DIR=/data/.openclaw \
  -e OPENCLAW_WORKSPACE_DIR=/data/workspace \
  -v $(pwd)/.tmpdata:/data \
  openclaw-railway-template
```

### Force clean rebuild (after dependency or Dockerfile changes)
```bash
docker build --no-cache -t openclaw-railway-template .
```

### Drop into a shell for inspection
```bash
docker run --rm -it \
  -v $(pwd)/.tmpdata:/data \
  openclaw-railway-template bash
```

## Pitfalls
- **Homebrew install takes 2–3 minutes** on first build — this is expected and cached in the Docker layer. Do not interrupt.
- **Volume at `/data` is required** — without it, config and tokens are lost on container restart, breaking device pairings.
- **`/data` must be writable** — Railway mounts this automatically; for local runs, ensure the bind-mount path exists and is writable.