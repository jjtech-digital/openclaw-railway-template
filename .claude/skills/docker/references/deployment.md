# Deployment Reference

## When to use
Reference this when deploying or redeploying to Railway, configuring environment variables, or troubleshooting post-deploy issues.

## Patterns

### Required Railway Variables
| Variable | Notes |
|---|---|
| `SETUP_PASSWORD` | Protects `/setup` — set before first deploy |
| `OPENCLAW_GATEWAY_TOKEN` | Use `${{ secret() }}` in Railway to generate securely |
| `OPENCLAW_STATE_DIR` | Set to `/data/.openclaw` |
| `OPENCLAW_WORKSPACE_DIR` | Set to `/data/workspace` |

### Confirming persistence across redeploys
After redeploying, visit `/setup/api/debug` and verify:
- `openclaw.json` exists at `OPENCLAW_STATE_DIR`
- `gateway.token` file is present and unchanged
- Gateway token matches the previously paired device

### Gateway token stability
The wrapper resolves the token in this order:
1. `OPENCLAW_GATEWAY_TOKEN` env var (Railway secret)
2. `${STATE_DIR}/gateway.token` on disk
3. Auto-generates and persists a new one

Set `OPENCLAW_GATEWAY_TOKEN` as a Railway secret to ensure token stability independent of volume state.

## Pitfalls
- **Volume must be mounted at `/data`** — Railway template requires this; without it, every redeploy resets all configuration.
- **Changing `OPENCLAW_GATEWAY_TOKEN` after pairing breaks existing devices** — treat it as immutable once set.
- **Public networking must be enabled** in Railway to receive a `*.up.railway.app` domain for CORS and browser access.