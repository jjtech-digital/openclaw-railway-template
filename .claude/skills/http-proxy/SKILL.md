---
name: http-proxy
description: Provides HTTP reverse proxy and request forwarding for the OpenClaw Railway template, including WebSocket proxying and bearer token injection.
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Http-proxy Skill

This skill covers working with the `http-proxy` reverse proxy layer in `src/server.js`. The wrapper uses `http-proxy` (v1.18.x) to transparently forward all non-setup traffic from the public Express server (port 8080) to the internal OpenClaw gateway (port 18789), automatically injecting the bearer token so browser clients never handle credentials directly.

## Quick Start

The proxy is initialized and configured in `src/server.js`. Key env vars:

| Variable | Default | Purpose |
|---|---|---|
| `INTERNAL_GATEWAY_PORT` | `18789` | Target gateway port |
| `INTERNAL_GATEWAY_HOST` | `127.0.0.1` | Target gateway host |
| `OPENCLAW_GATEWAY_TOKEN` | auto-generated | Bearer token injected into every proxied request |

Token precedence: `OPENCLAW_GATEWAY_TOKEN` env → `${STATE_DIR}/gateway.token` on disk → newly generated token persisted with mode `0o600`.

## Key Concepts

**Token injection via proxy events, not `req.headers`**
Direct `req.headers` modification is unreliable for WebSocket upgrades. Always use the `http-proxy` event hooks:
- `proxy.on("proxyReq", ...)` — HTTP requests (`src/server.js:736`)
- `proxy.on("proxyReqWs", ...)` — WebSocket upgrades (`src/server.js:741`)

**Stable tokens across redeploys**
Every redeploy must use the same token or existing device pairings break. Persist the token to volume and prefer the `OPENCLAW_GATEWAY_TOKEN` Railway Variable for production.

**Gateway readiness polling**
Before proxying, the wrapper polls multiple endpoints (`/openclaw`, `/`, `/health`) because different OpenClaw builds expose different routes. Traffic is not forwarded until the gateway responds.

## Common Patterns

**Adding a custom request header to all proxied requests**
```javascript
proxy.on("proxyReq", (proxyReq, req, res) => {
  proxyReq.setHeader("X-Custom-Header", value);
});
```

**Adding a header to WebSocket upgrades**
```javascript
proxy.on("proxyReqWs", (proxyReq, req, socket, options, head) => {
  proxyReq.setHeader("X-Custom-Header", value);
});
```

**Handling proxy errors (502/503)**
```javascript
proxy.on("error", (err, req, res) => {
  log.error("proxy", err.message);
  if (!res.headersSent) res.status(502).send("Bad Gateway");
});
```

**Bypassing the proxy for a specific route**
Register the Express route before the catch-all proxy handler. Routes registered first take precedence:
```javascript
app.get("/my-route", handler);       // handled by Express
app.use("/", (req, res) => proxy.web(req, res, { target: GATEWAY_TARGET }));
```