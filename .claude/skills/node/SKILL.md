---
name: node
description: Provides Node.js runtime environment and built-in modules guidance for the OpenClaw Railway Template project
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Node Skill

This skill assists with Node.js runtime usage in the OpenClaw Railway Template, a Node.js 24.x Express-based wrapper that proxies traffic to an internal OpenClaw gateway. It covers built-in module patterns, ESM syntax, child process management, and runtime conventions used throughout this codebase.

## Quick Start

```bash
# Verify Node.js version (requires >=24)
node --version

# Syntax check the main server file
node -c src/server.js

# Run the development server
node src/server.js
```

## Key Concepts

- **ESM modules**: The project uses `"type": "module"` — always use `import`/`export`, never `require()`
- **Built-in imports**: Use the `node:` prefix (e.g., `import fs from "node:fs"`, `import crypto from "node:crypto"`)
- **Import order**: Built-ins first (alphabetical), then external packages (alphabetical)
- **Child processes**: Gateway is spawned via `child_process` — use `spawn()` with inherited stdio
- **Crypto**: Token generation uses `crypto.randomBytes(32).toString("hex")`
- **File permissions**: Sensitive files (e.g., `gateway.token`) created with mode `0o600`

## Common Patterns

**Spawning a child process (gateway lifecycle):**
```javascript
import childProcess from "node:child_process";

const proc = childProcess.spawn(OPENCLAW_NODE, args, {
  stdio: "inherit",
  env: { ...process.env }
});
```

**Generating a secure token:**
```javascript
import crypto from "node:crypto";

const token = crypto.randomBytes(32).toString("hex");
```

**Reading/writing files with error handling:**
```javascript
import fs from "node:fs";

try {
  const data = fs.readFileSync(filePath, "utf8");
} catch (err) {
  log.error("config", err.message);
}

fs.writeFileSync(filePath, content, { mode: 0o600 });
```

**Path construction:**
```javascript
import path from "node:path";

const configPath = path.join(STATE_DIR, "openclaw.json");
```