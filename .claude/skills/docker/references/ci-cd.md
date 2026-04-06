# CI/CD Reference

## When to use
Reference this when modifying the build pipeline, automating image builds, or validating the template before Railway deployment.

## Patterns

### Pre-push validation checklist
```bash
# Syntax check
npm run lint

# Docker build smoke test
docker build -t openclaw-railway-template .

# Health check after container start
curl http://localhost:8080/healthz
```

### Pinning the OpenClaw version for reproducible builds
Edit the install line in `Dockerfile`:
```dockerfile
RUN npm install -g openclaw@2026.3.13 clawhub@latest
```
See `docs/OPENCLAW-VERSION-CONTROL.md` for version management strategy.

### Verifying setup flow after a build
```bash
# Wipe state, then re-run setup wizard end-to-end
rm -rf .tmpdata
docker run --rm -p 8080:8080 \
  -e SETUP_PASSWORD=test \
  -e OPENCLAW_STATE_DIR=/data/.openclaw \
  -e OPENCLAW_WORKSPACE_DIR=/data/workspace \
  -v $(pwd)/.tmpdata:/data \
  openclaw-railway-template
# Visit http://localhost:8080/setup
```

## Pitfalls
- **No automated test suite** — validation is manual. Always complete the full setup wizard flow before marking a build ready.
- **`npm run lint` only checks `server.js` syntax** — it does not catch runtime errors or broken proxy logic.