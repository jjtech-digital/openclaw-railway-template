---
name: pnpm
description: Manages package dependencies and lock file versioning for the OpenClaw Railway Template using pnpm
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Pnpm Skill

Handles dependency installation, updates, and `pnpm-lock.yaml` integrity for this project. The project uses pnpm as its primary package manager with `node-pty` as the only package requiring a native build step.

## Quick Start

```bash
# Install all dependencies
pnpm install

# Add a new dependency
pnpm add <package>

# Add a dev dependency
pnpm add -D <package>

# Remove a dependency
pnpm remove <package>

# Update a specific package
pnpm update <package>
```

## Key Concepts

- **Lock file**: `pnpm-lock.yaml` at the project root pins exact versions for reproducible Railway builds — always commit it alongside `package.json` changes.
- **Only built dependencies**: `package.json` restricts native builds to `node-pty` only via `"pnpm": { "onlyBuiltDependencies": ["node-pty"] }` — do not add other packages to this list without good reason.
- **ESM project**: `"type": "module"` in `package.json` means all `.js` files use ES module syntax (`import`/`export`), not CommonJS.
- **Node engine**: Requires Node.js `>=24`; ensure any added packages are compatible.

## Common Patterns

**After pulling changes that touch `package.json`:**
```bash
pnpm install
```

**Verify no unintended lock file drift:**
```bash
pnpm install --frozen-lockfile
```

**Check for outdated packages:**
```bash
pnpm outdated
```

**Audit for vulnerabilities:**
```bash
pnpm audit
```

**Syntax-check the server after dependency changes:**
```bash
npm run lint
```

When adding dependencies, confirm they are ESM-compatible or provide named exports usable with `import`. Avoid CJS-only packages that require `createRequire` workarounds.