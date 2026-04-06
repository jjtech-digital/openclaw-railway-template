---
name: express
description: Enables HTTP server routing, middleware, and request handling for the OpenClaw Railway wrapper using Express 5.1.x
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Express Skill

Handles Express 5.1.x routing, middleware configuration, and request/response logic within `src/server.js` — the main wrapper that proxies traffic to the internal OpenClaw gateway, serves the `/setup` wizard, and manages authentication layers.

## Quick Start

```bash
# Validate server syntax after edits
npm run lint

# Run locally
npm run dev
# Server listens on PORT (default 8080)
```

## Key Concepts

- **Single entry point**: All Express logic lives in `src/server.js`. No inline HTML/CSS — static assets are in `src/public/`.
- **Two auth layers**: Basic auth via `requireSetupAuth` middleware protects `/setup/*` and `/logs`; bearer token injection protects the proxied gateway routes.
- **Route hierarchy**:
  - `/setup/*` — setup wizard (Basic auth: `SETUP_PASSWORD`)
  - `/logs` — live log viewer (Basic auth)
  - `/tui` — terminal UI (Basic auth + session management, requires `ENABLE_WEB_TUI=true`)
  - `/healthz` — public health check
  - `/*` — reverse proxy to internal gateway (`localhost:18789`)
- **Static assets**: Served via `express.static` from `src/public/` under the `/setup` prefix.
- **Logging**: Use `log.info(category, message)` — never `console.log`. Tokens must be redacted via `redactSecrets()`.

## Common Patterns

**Adding a new protected route:**
```javascript
app.get("/my-route", requireSetupAuth, (req, res) => {
  res.sendFile(path.join(PUBLIC_DIR, "my-page.html"));
});
```

**Adding a new API endpoint under `/setup/api/`:**
```javascript
app.post("/setup/api/my-action", requireSetupAuth, async (req, res) => {
  try {
    const result = await someAsyncOperation(req.body);
    res.json({ ok: true, result });
  } catch (err) {
    log.error("my-action", err.message);
    res.status(500).json({ ok: false, error: err.message });
  }
});
```

**Modifying proxy behavior (HTTP):**
Edit the `proxy.on("proxyReq")` handler in `src/server.js` around line 736. Always use proxy event handlers — do not modify `req.headers` directly, as it does not work reliably for WebSocket upgrades.

**Adding a debug console command:**
1. Add the command string to `allowedCommands` (a `Set`).
2. Add a `case` in the `/setup/api/console` POST handler.
3. Add an `<option>` in `src/public/setup.html` under the relevant `<optgroup>`.