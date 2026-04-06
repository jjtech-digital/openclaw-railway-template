---
name: ws
description: Implements WebSocket server for real-time communication using the ws 8.18.x library in the OpenClaw Railway Template
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Ws Skill

Adds or modifies WebSocket server functionality in the OpenClaw Railway Template. This project uses `ws` 8.18.x alongside Express 5.x and `http-proxy` to support real-time channels (TUI terminal streaming, logs, gateway proxying). WebSocket upgrades are handled separately from HTTP requests and require special care when injecting authorization headers.

## Quick Start

```javascript
import { WebSocketServer } from "ws";

// Attach to existing HTTP server (do not create a separate server)
const wss = new WebSocketServer({ noServer: true });

server.on("upgrade", (req, socket, head) => {
  wss.handleUpgrade(req, socket, head, (ws) => {
    wss.emit("connection", ws, req);
  });
});

wss.on("connection", (ws, req) => {
  ws.on("message", (data) => { /* handle */ });
  ws.on("close", () => { /* cleanup */ });
  ws.send(JSON.stringify({ type: "ready" }));
});
```

## Key Concepts

- **`noServer: true`** — Required when sharing a port with Express; upgrades are routed manually via `server.on("upgrade")`.
- **Auth injection for proxied WebSockets** — Use `proxy.on("proxyReqWs")` to inject `Authorization: Bearer <token>`. Direct `req.headers` mutation does not reliably propagate through `http-proxy` WebSocket upgrades.
- **Session lifecycle** — TUI sessions enforce `TUI_IDLE_TIMEOUT_MS` (default 5 min) and `TUI_MAX_SESSION_MS` (default 30 min) via `setTimeout`; always clear timers on `close`.
- **Auth guard** — WebSocket upgrade handlers must validate the session/cookie before calling `handleUpgrade`; reject unauthorized upgrades by destroying the socket.

## Common Patterns

**Broadcast to all connected clients:**
```javascript
for (const client of wss.clients) {
  if (client.readyState === WebSocket.OPEN) {
    client.send(data);
  }
}
```

**Idle timeout:**
```javascript
let idleTimer = setTimeout(() => ws.close(1001, "idle"), TUI_IDLE_TIMEOUT_MS);
ws.on("message", () => {
  clearTimeout(idleTimer);
  idleTimer = setTimeout(() => ws.close(1001, "idle"), TUI_IDLE_TIMEOUT_MS);
});
ws.on("close", () => clearTimeout(idleTimer));
```

**Proxy WebSocket with token injection (project pattern):**
```javascript
proxy.on("proxyReqWs", (proxyReq) => {
  proxyReq.setHeader("Authorization", `Bearer ${gatewayToken}`);
});
```

**Graceful shutdown:**
```javascript
process.on("SIGTERM", () => {
  wss.clients.forEach((ws) => ws.close(1001, "server shutting down"));
  wss.close();
});
```