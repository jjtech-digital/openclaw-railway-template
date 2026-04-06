---
name: building-compare-hubs
description: Creates comparison and alternative pages for discovery in the OpenClaw Railway Template
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Building-compare-hubs Skill

This skill creates comparison and "alternatives to" pages that drive organic discovery for the OpenClaw Railway Template. These pages target high-intent search queries (e.g., "OpenClaw vs X", "alternatives to Y") and route visitors into the setup wizard, leveraging the existing `/setup` onboarding flow and `src/public/` static asset pipeline.

## Quick Start

1. Identify the competitor or category to target (e.g., a competing AI coding assistant platform).
2. Create a new HTML page under `src/public/` following the naming pattern `compare-<slug>.html` or `alternatives-<slug>.html`.
3. Reuse `styles.css` for consistent styling — link it as `<link rel="stylesheet" href="/styles.css">`.
4. Add a route in `src/server.js` to serve the page, protected or public depending on intent.
5. Include a CTA that links to `/setup` to drive conversion.

## Key Concepts

- **Consistent shell**: All compare pages inherit `styles.css` and should match the visual language of `setup.html` and `tui.html`.
- **No build step**: Pages use vanilla JS only — no bundler, no framework. Keep client-side logic in an accompanying `-app.js` file if needed.
- **Route registration**: Static files in `src/public/` are not auto-served; explicit `app.get()` routes must be added in `src/server.js` for each new page.
- **Auth boundary**: Compare pages are typically public (no `requireSetupAuth` middleware) since they target cold traffic. Only gate them if the content is sensitive.
- **CTA target**: The primary conversion action is always `/setup` — the existing wizard handles onboarding without duplicating that logic on compare pages.

## Common Patterns

**Serving a new public compare page:**
```javascript
// src/server.js — add alongside other static routes
app.get("/compare/openclaw-vs-competitor", (req, res) => {
  res.sendFile(path.join(PUBLIC_DIR, "compare-competitor.html"));
});
```

**Reusing shared styles in a compare page:**
```html
<link rel="stylesheet" href="/styles.css">
```

**CTA block pointing to setup wizard:**
```html
<a href="/setup" class="btn btn-primary">Get Started with OpenClaw</a>
```

**Feature comparison table structure:**
```html
<table class="compare-table">
  <thead><tr><th>Feature</th><th>OpenClaw</th><th>Competitor</th></tr></thead>
  <tbody>
    <tr><td>Browser-first UI</td><td>Yes</td><td>No</td></tr>
    <tr><td>Railway one-click deploy</td><td>Yes</td><td>No</td></tr>
  </tbody>
</table>
```