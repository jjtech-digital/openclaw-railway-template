# Product Analytics Release Notes

## When to use
Reference this when drafting notes for changes to health endpoints, diagnostic output, logging format, or anything that affects observability of a running deployment.

## Patterns

**New health endpoint field:**
```
### `/healthz` now reports gateway state
Response includes `gatewayReady: true/false` and uptime. Useful for Railway health check configuration and external monitoring.
```

**Log format changed:**
```
### Log entries now include category prefix
Format: `[timestamp] [level] [category] message`. Affects log parsing if you pipe `/logs` output to external tools.
```

**Debug API expanded:**
```
### `/setup/api/debug` returns expanded diagnostics
Now includes env var presence (not values), gateway PID, and config file path. Useful for diagnosing misconfigured Railway deployments.
```

## Pitfalls
- Never include token or password values in example log output in release notes — use `[REDACTED]` placeholders.
- Log format changes are breaking for users who parse `/logs` programmatically. Mark these clearly even if the change seems minor.