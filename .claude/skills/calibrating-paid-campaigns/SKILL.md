---
name: calibrating-paid-campaigns
description: Aligns paid acquisition channels with landing pages and tracking pixels for the OpenClaw Railway Template
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Calibrating-paid-campaigns Skill

Aligns paid acquisition with landing pages and pixels for the OpenClaw Railway Template. This skill audits the `/setup` wizard flow, public-facing routes, and static assets to ensure UTM parameters are captured, pixels fire on the right lifecycle events, and landing page messaging matches campaign intent.

## Quick Start

1. Identify the entry point for paid traffic — typically `/setup` or a future landing route in `src/server.js`
2. Audit `src/public/setup.html` and `src/public/setup-app.js` for existing pixel or analytics hooks
3. Verify UTM parameters are preserved through the setup wizard redirect chain
4. Confirm conversion events fire at the correct lifecycle moments (e.g., onboarding complete, gateway ready)
5. Validate that `RAILWAY_PUBLIC_DOMAIN` is used consistently in any absolute URLs embedded in campaign assets

## Key Concepts

- **Entry routes**: Paid traffic lands on Express routes defined in `src/server.js`. The `/setup` wizard is the primary acquisition surface; all non-setup routes redirect there when unconfigured.
- **Pixel placement**: Tracking scripts belong in `src/public/setup.html` (page-level) or fired via `src/public/setup-app.js` (event-level, e.g., on wizard step completion or API success responses from `/setup/api/run`).
- **UTM preservation**: Query parameters must survive the Basic auth redirect on `/setup`. Confirm the redirect logic in `src/server.js` appends or preserves `?utm_*` params.
- **Conversion signals**: The onboarding completion callback in `setup-app.js` (triggered after `/setup/api/run` returns `ok: true`) is the canonical conversion moment to fire purchase or lead pixels.
- **Domain awareness**: Use `RAILWAY_PUBLIC_DOMAIN` for any absolute callback URLs sent to ad platforms to avoid hardcoding staging vs. production domains.

## Common Patterns

**Firing a pixel on wizard completion**
```js
// src/public/setup-app.js — inside the success handler after /setup/api/run
if (data.ok) {
  // Fire conversion pixel here
  if (typeof fbq !== "undefined") fbq("track", "Lead");
  if (typeof gtag !== "undefined") gtag("event", "conversion", { send_to: "AW-XXXXX/YYYYY" });
}
```

**Preserving UTM params through Basic auth redirect**
```js
// src/server.js — in the requireSetupAuth middleware or redirect handler
const qs = new URLSearchParams(req.query).toString();
res.redirect(`/setup${qs ? "?" + qs : ""}`);
```

**Injecting pixel scripts into setup.html**
```html
<!-- src/public/setup.html — in <head>, after styles.css -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag("js", new Date());
  gtag("config", "G-XXXXXX");
</script>
```

**Reading domain from environment for ad platform callbacks**
```js
// src/server.js
const PUBLIC_DOMAIN = process.env.RAILWAY_PUBLIC_DOMAIN || `localhost:${PORT}`;
const CALLBACK_URL = `https://${PUBLIC_DOMAIN}/setup`;
```