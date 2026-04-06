---
name: building-acquisition-tools
description: Designs lead magnets or free tools for acquisition in the OpenClaw Railway Template
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Building-acquisition-tools Skill

This skill guides the design and implementation of lead magnets and free acquisition tools within the OpenClaw Railway Template. It covers adding new public-facing endpoints, lightweight interactive tools served via Express, and self-contained UI pages that lower the barrier to entry for new users discovering the platform.

## Quick Start

1. Identify the acquisition goal (email capture, trial activation, demo access, etc.)
2. Choose a delivery surface: a new static page in `src/public/`, a new Express route in `src/server.js`, or a combination
3. Add the route before the catch-all proxy handler so it is served without authentication
4. Wire any server-side logic (form handling, token generation, redirects) as a POST handler under a clearly namespaced path (e.g., `/acquire/*`)
5. Validate with `npm run lint` and smoke-test locally via `npm run dev`

## Key Concepts

- **Public vs. protected routes** — acquisition tools must be reachable without `SETUP_PASSWORD`. Place them before the `requireSetupAuth` middleware in `src/server.js`.
- **Static assets** — drop HTML, CSS, and vanilla JS into `src/public/`. Express serves this directory automatically; no build step is needed.
- **Minimal auth surface** — lead magnets should not expose gateway internals. Do not inject the bearer token or proxy requests for unauthenticated acquisition pages.
- **Redirection to value** — after a user completes a lead-magnet action, redirect to `/setup` or `/openclaw` to move them into the configured onboarding funnel.
- **Secrets hygiene** — never embed `SETUP_PASSWORD` or `OPENCLAW_GATEWAY_TOKEN` in acquisition page responses. Use `redactSecrets()` if any server output is reflected.

## Common Patterns

**New public landing page**
Add `src/public/acquire.html` with a self-contained form, then register a GET route in `src/server.js`:
```js
app.get("/acquire", (req, res) => {
  res.sendFile(path.join(PUBLIC_DIR, "acquire.html"));
});
```

**Form submission handler**
Accept POST data, perform lightweight server logic (e.g., write a trial token, send a webhook), then redirect:
```js
app.post("/acquire/submit", express.urlencoded({ extended: false }), async (req, res) => {
  const { email } = req.body;
  // ... process lead ...
  res.redirect("/setup");
});
```

**Gated free-tool endpoint**
Expose a diagnostic or demo tool that requires no password but rate-limits by IP to prevent abuse:
```js
const trialHits = new Map();
app.get("/free-tool", (req, res) => {
  const ip = req.ip;
  if ((trialHits.get(ip) ?? 0) >= 5) return res.status(429).send("Limit reached");
  trialHits.set(ip, (trialHits.get(ip) ?? 0) + 1);
  res.json({ hint: "Deploy OpenClaw on Railway in 60 seconds." });
});
```

**Linking into setup funnel**
End every acquisition flow with a clear CTA that points to `/setup` so users transition from free tool to configured deployment without friction.