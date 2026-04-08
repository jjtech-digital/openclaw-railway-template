# OpenClaw Railway Template

Deploy OpenClaw on Railway with a browser-first setup flow. No SSH required for onboarding.

**Upgrading?** Remove the `OPENCLAW_ENTRY` env var -- OpenClaw is now installed via npm.

## Read This First

This template exposes your OpenClaw gateway to the public internet.

- Review OpenClaw security guidance: <https://docs.openclaw.ai/gateway/security>
- Use a strong `SETUP_PASSWORD`
- If you only use chat channels, consider disabling public networking after setup

## What You Get

- OpenClaw Gateway + Control UI at `/` and `/openclaw`
- Setup Wizard at `/setup` (Basic auth protected)
- Optional browser TUI at `/tui`
- Persistent state on Railway volume (`/data`)
- Health endpoint at `/healthz`
- Diagnostics and logs via setup tools + `/logs`
- Config backup, restore, export, and import
- Device pairing management from the setup wizard
- Debug console for running diagnostic commands

## Quick Start (Railway)

1. Deploy this template to Railway.
2. Ensure a volume is mounted at `/data`.
3. Set variables:
   - `SETUP_PASSWORD` (required)
   - `OPENCLAW_STATE_DIR=/data/.openclaw`
   - `OPENCLAW_WORKSPACE_DIR=/data/workspace`
   - Optional: `ENABLE_WEB_TUI=true`
4. Open `https://<your-domain>/setup` and complete onboarding.
5. Open `https://<your-domain>/openclaw` from the setup page.

## Environment Variables

### Required

- `SETUP_PASSWORD`: password for `/setup`

### Recommended

- `OPENCLAW_STATE_DIR=/data/.openclaw`
- `OPENCLAW_WORKSPACE_DIR=/data/workspace`
- `OPENCLAW_GATEWAY_TOKEN` (stable token across redeploys)

### Optional

- `PORT=8080`
- `INTERNAL_GATEWAY_PORT=18789`
- `INTERNAL_GATEWAY_HOST=127.0.0.1`
- `ENABLE_WEB_TUI=false`
- `TUI_IDLE_TIMEOUT_MS=300000`
- `TUI_MAX_SESSION_MS=1800000`

## Day-1 Setup Checklist

- Confirm `/setup` loads and accepts password
- Run onboarding once
- Verify `/healthz` returns `{ "ok": true, ... }`
- Open `/openclaw` via setup link
- If using Telegram/Discord, approve pending devices from the Devices panel in `/setup`
- Create a manual backup from the Backups panel (good practice before going live)

## Chat Token Prep

### Telegram

1. Message `@BotFather`
2. Run `/newbot`
3. Copy bot token (looks like `123456789:AA...`)
4. Paste into setup wizard

### Discord

1. Create app in Discord Developer Portal
2. Add bot + copy bot token
3. Invite bot to server (`bot`, `applications.commands` scopes)
4. Enable required intents for your use case

## Backup & Restore

The setup wizard includes a full backup and restore system for your OpenClaw configuration.

**Automatic backups** are created before destructive actions (reset, restore, import). On startup, if the config file is corrupted, the wrapper automatically restores from the latest backup.

**Manual backups** can be created from the Backups panel in `/setup`. Up to 10 backups are retained; older ones are pruned automatically.

### Export / Import

- **Export**: Download a password-protected ZIP of your config and gateway token from `/setup` (Tools > Export Data).
- **Import**: Upload a previously exported ZIP via `/setup` (Tools > Import Data). A backup is created before overwriting.

This is useful for migrating between Railway instances or keeping an off-site copy of your configuration.

## Device Management

Manage paired devices directly from `/setup` (Tools > Manage Devices). You can view pending and approved devices, and approve or reject pairing requests without needing SSH access.

## Debug Console

The setup wizard includes a debug console (Tools > Debug Console) for running pre-approved diagnostic commands against the OpenClaw CLI. Commands include checking config values, listing devices, running doctor, and more.

## Web TUI (`/tui`)

Disabled by default. Set `ENABLE_WEB_TUI=true` to enable.

Built-in safeguards:

- Protected by `SETUP_PASSWORD`
- Single active session
- Idle timeout
- Max session duration

## Local Smoke Test

```bash
docker build -t openclaw-railway-template .

docker run --rm -p 8080:8080 \
  -e PORT=8080 \
  -e SETUP_PASSWORD=test \
  -e OPENCLAW_STATE_DIR=/data/.openclaw \
  -e OPENCLAW_WORKSPACE_DIR=/data/workspace \
  -e ENABLE_WEB_TUI=true \
  -v $(pwd)/.tmpdata:/data \
  openclaw-railway-template
```

- Setup: `http://localhost:8080/setup` (password: `test`)
- UI: `http://localhost:8080/openclaw`
- TUI: `http://localhost:8080/tui`

## Troubleshooting

### Control UI says disconnected / auth error

- Open `/setup` first, then click the OpenClaw UI link from there.
- Approve pending devices from the Devices panel in `/setup`.

### 502 / gateway unavailable

- Check `/healthz`
- Run `openclaw doctor --repair` from the debug console in `/setup`
- Verify `/data` volume is mounted and writable

### Setup keeps resetting after redeploy

- `OPENCLAW_STATE_DIR` or `OPENCLAW_WORKSPACE_DIR` is not on `/data`
- Fix both vars and redeploy

### Config corrupted after crash

- The wrapper auto-restores from the latest backup on startup
- If auto-restore fails, use the Backups panel in `/setup` to manually restore
- If no backups exist, use Reset Setup to start fresh (or import a previously exported ZIP)

### TUI not visible

- Set `ENABLE_WEB_TUI=true`
- Redeploy and reload `/setup`

## Useful Endpoints

| Endpoint | Auth | Purpose |
|----------|------|---------|
| `/setup` | Basic auth | Onboarding wizard, config tools, backups, devices |
| `/openclaw` | Auto-injected | OpenClaw Control UI |
| `/healthz` | None | Public health check |
| `/setup/healthz` | None | Setup-specific health check |
| `/logs` | Basic auth | Live server logs viewer |
| `/tui` | Basic auth | Browser terminal (if enabled) |

## Support

Need help? Open an issue or use Railway Station support for this template.
