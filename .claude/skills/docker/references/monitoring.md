# Monitoring Reference

## When to use
Reference this when diagnosing gateway startup failures, inspecting live logs, or verifying system health in a running container.

## Patterns

### Health check
```bash
curl http://localhost:8080/healthz
# Returns JSON with wrapper and gateway status
```

### Live log viewer
Visit `/logs` in the browser (requires `SETUP_PASSWORD` auth). The ring buffer holds the last 1000 log lines in memory.

### Key log markers to watch
| Log fragment | Meaning |
|---|---|
| `[gateway] starting with command: ...` | Gateway process spawned |
| `[gateway] ready at <endpoint>` | Gateway is healthy and proxying |
| `[gateway] failed to become ready after 60000ms` | Gateway did not start — check `openclaw.json` and token config |
| `token_missing` / `token_mismatch` | Bearer token not injected — check proxy event handlers in `server.js` |

### Debug console
Available at `/setup` → Tools → Debug Console. Runs pre-approved commands (e.g., `gateway.restart`) without shell access.

## Pitfalls
- **Secrets are redacted in logs** — if you need to verify token values, check `${STATE_DIR}/gateway.token` directly inside the container.
- **`/logs` requires setup auth** — unauthenticated requests are rejected; use the browser with `SETUP_PASSWORD` credentials.
- **Gateway readiness polls multiple endpoints** (`/openclaw`, `/`, `/health`) — a single failing endpoint does not indicate full gateway failure.