---
name: node-pty
description: Implements terminal emulation for interactive TUI features using node-pty in the OpenClaw Railway Template
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Node-pty Skill

This skill covers working with `node-pty` (v1.0.x) to implement pseudoterminal (PTY) sessions that back the `/tui` endpoint in `src/server.js`. The TUI bridges a browser WebSocket connection to a real PTY running an interactive CLI process, with session lifecycle management (idle timeout, max duration) enforced server-side.

## Quick Start

Enable and access the TUI:
```bash
# Set env vars
export ENABLE_WEB_TUI=true
export TUI_IDLE_TIMEOUT_MS=300000   # 5 min idle (default)
export TUI_MAX_SESSION_MS=1800000   # 30 min max (default)

# Start server
npm run dev
# Visit http://localhost:8080/tui (requires SETUP_PASSWORD auth)
```

## Key Concepts

- **PTY spawn**: `pty.spawn(cmd, args, { cols, rows, env })` creates a pseudoterminal — output is raw ANSI/VT100 bytes streamed to the browser via WebSocket.
- **Resize**: PTY must be resized when the browser terminal resizes; call `ptyProcess.resize(cols, rows)` on resize messages from the client.
- **Session management**: Each `/tui` WebSocket connection gets one PTY. Idle and max-duration timers are tracked per session and cleared on disconnect.
- **Auth gate**: `/tui` requires the same Basic auth as `/setup` (`requireSetupAuth` middleware); it is never exposed without credentials.
- **`onlyBuiltDependencies`**: `node-pty` requires a native build step. The `pnpm.onlyBuiltDependencies` field in `package.json` ensures only `node-pty` runs its build script — this matters in Docker where build tooling must be present.

## Common Patterns

**Spawn a PTY session:**
```javascript
import pty from "node-pty";

const ptyProcess = pty.spawn("bash", [], {
  name: "xterm-color",
  cols: 80,
  rows: 24,
  cwd: WORKSPACE_DIR,
  env: { ...process.env },
});

ptyProcess.onData((data) => ws.send(data));
ws.on("message", (msg) => ptyProcess.write(msg));
```

**Handle terminal resize from browser:**
```javascript
ws.on("message", (msg) => {
  const parsed = tryParseJson(msg);
  if (parsed?.type === "resize") {
    ptyProcess.resize(parsed.cols, parsed.rows);
    return;
  }
  ptyProcess.write(msg);
});
```

**Enforce idle + max session timeouts:**
```javascript
let idleTimer = setTimeout(cleanup, TUI_IDLE_TIMEOUT_MS);
const maxTimer = setTimeout(cleanup, TUI_MAX_SESSION_MS);

ptyProcess.onData((data) => {
  clearTimeout(idleTimer);
  idleTimer = setTimeout(cleanup, TUI_IDLE_TIMEOUT_MS);
  ws.send(data);
});

function cleanup() {
  clearTimeout(idleTimer);
  clearTimeout(maxTimer);
  ptyProcess.kill();
  ws.close();
}

ws.on("close", cleanup);
```

**Log PTY lifecycle (follow project conventions):**
```javascript
log.info("tui", `PTY session started (pid=${ptyProcess.pid})`);
ptyProcess.onExit(({ exitCode }) =>
  log.info("tui", `PTY exited with code ${exitCode}`)
);
```