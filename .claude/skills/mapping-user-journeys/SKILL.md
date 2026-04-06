---
name: mapping-user-journeys
description: Maps in-app user journeys and identifies friction points in code for the OpenClaw Railway Template
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Mapping-user-journeys Skill

Maps the end-to-end user journeys through the OpenClaw Railway Template — from initial `/setup` wizard through gateway startup and proxied traffic — and surfaces friction points in route handlers, lifecycle transitions, auth flows, and client-side JS.

## Quick Start

1. Identify the journey to map (e.g., first-time onboarding, gateway restart, device pairing)
2. Trace the request flow through `src/server.js` route handlers and middleware
3. Cross-reference client-side behavior in `src/public/setup-app.js` and HTML files
4. Flag redirects, auth gates, error states, and polling loops as friction candidates
5. Report findings as an ordered journey with friction points annotated inline

## Key Concepts

**Lifecycle states** — The wrapper has two top-level states: `Unconfigured` (no `openclaw.json`, all routes redirect to `/setup`) and `Configured` (gateway running, traffic proxied). Transitions between states are the most common friction points.

**Auth layers** — Two independent auth schemes: Basic auth on `/setup/*` and `/logs`/`/tui`, and Bearer token injection on all proxied requests. Friction arises when users hit auth gates without context.

**Gateway readiness polling** — After onboarding, the wrapper polls multiple endpoints (`/openclaw`, `/`, `/health`) before declaring the gateway ready. Long poll cycles or silent failures are friction.

**Route ownership** — `/setup/*` routes are handled entirely by `server.js`; everything else is reverse-proxied to the internal gateway on port 18789. Misrouted requests are a common confusion point.

**Client–server handoff** — `setup-app.js` drives the wizard UI; `server.js` exposes `/setup/api/*` endpoints. Mismatches between what the UI expects and what the API returns create silent failures.

## Common Patterns

**Tracing a journey end-to-end:**
```
Grep for route definitions → Read handler logic → follow redirects and middleware chain → check client JS for corresponding fetch calls
```

**Finding redirect friction:**
```
Grep pattern: `res\.redirect` in src/server.js — note conditions that trigger each redirect and whether the user receives context before being redirected
```

**Identifying silent failures:**
```
Grep pattern: `res\.json\(\{ ok: false` — these are API error responses; check whether setup-app.js surfaces them to the user or swallows them
```

**Mapping the onboarding journey:**
```
Entry: GET /setup → Basic auth → setup.html
Wizard: POST /setup/api/run → buildOnboardArgs() → openclaw onboard → config writes → gateway spawn → readiness poll
Exit: redirect to /openclaw
Friction zones: auth challenge with no guidance, poll timeout with no user feedback, channel config write failures logged but not surfaced
```

**Mapping the returning-user journey:**
```
Entry: any route → isConfigured check → gateway ready check → proxy
Friction zones: gateway not yet ready (loading.html shown), token mismatch after redeploy, WebSocket upgrade failures
```

**Identifying auth friction:**
```
Grep for `requireSetupAuth` middleware — every route it wraps will challenge unauthenticated users; verify each challenge includes a clear prompt and that the password hint is accessible
```