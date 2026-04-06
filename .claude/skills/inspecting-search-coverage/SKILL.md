---
name: inspecting-search-coverage
description: Audits technical and on-page search coverage for the OpenClaw Railway Template, checking route discoverability, endpoint documentation completeness, and static asset indexability.
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Inspecting-search-coverage Skill

Audits search coverage across the OpenClaw Railway Template by examining Express route definitions, public-facing endpoints, static HTML metadata, and setup wizard discoverability. Surfaces gaps between what the server exposes and what is documented or indexable.

## Quick Start

1. Grep `src/server.js` for all registered routes (`app.get`, `app.post`, `app.use`) to build a route inventory.
2. Cross-reference routes against `CLAUDE.md` architecture docs to find undocumented endpoints.
3. Check `src/public/*.html` files for `<title>`, `<meta name="description">`, and canonical tags.
4. Verify `/healthz` and `/setup/api/debug` are reachable and return structured responses.
5. Confirm auth-gated routes (`/logs`, `/tui`, `/setup/*`) are not inadvertently crawlable.

## Key Concepts

**Route inventory** — All Express route handlers in `src/server.js` represent the full surface area. Routes split into: public (`/healthz`), auth-gated (`/setup/*`, `/logs`, `/tui`), and proxied (`/*` → internal gateway on port 18789).

**On-page metadata** — Pages in `src/public/` (`setup.html`, `tui.html`, `logs.html`, `loading.html`) are served statically. Each should carry accurate `<title>` and `<meta>` tags so browser history and bookmarks are meaningful, even though Railway deployments are not publicly indexed.

**Auth boundary** — `/setup/*` requires HTTP Basic auth (`SETUP_PASSWORD`). Coverage audits must confirm that no setup API routes (e.g., `/setup/api/run`, `/setup/api/console`) leak outside the auth middleware.

**Proxy passthrough** — All unmatched routes proxy to `localhost:18789`. Coverage gap: if the gateway exposes routes the wrapper doesn't explicitly document, those are invisible to operators.

## Common Patterns

**Find all registered routes:**
```bash
grep -n "app\.\(get\|post\|put\|delete\|use\|all\)" src/server.js
```

**Check static HTML for missing metadata:**
```bash
grep -L "<meta name=\"description\"" src/public/*.html
```

**Audit auth middleware coverage:**
```bash
grep -n "requireSetupAuth\|basicAuth\|401\|403" src/server.js
```

**List all public asset files:**
```bash
find src/public -type f | sort
```

**Verify health endpoint response shape:**
```bash
curl -s http://localhost:8080/healthz | jq .
```

**Detect routes missing from CLAUDE.md:**
Compare `grep "app\.\(get\|post\|use\)" src/server.js` output against the Request Flow diagram in `CLAUDE.md` — any route not listed there is a coverage gap.