---
name: refining-prompt-surfaces
description: Optimizes banners, modals, and in-app prompts in the OpenClaw Railway Template UI
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Refining-prompt-surfaces Skill

Improves the clarity, timing, and copy of banners, modals, status messages, and inline prompts across the `/setup` wizard and supporting UI pages (`setup.html`, `tui.html`, `logs.html`, `loading.html`). Focuses on reducing friction during onboarding and surfacing actionable guidance at the right moment.

## Quick Start

1. Identify the prompt surface to refine — wizard step, error banner, status indicator, or modal.
2. Locate the relevant file under `src/public/` (HTML structure) or `src/public/setup-app.js` (dynamic copy injected via JS).
3. Read the surrounding context to understand when and how the message is triggered.
4. Apply copy and UX changes directly; no build step is required (vanilla JS, static HTML).
5. Run `npm run lint` to confirm `server.js` is unaffected if any server-side message strings were touched.

## Key Concepts

- **Wizard steps**: Multi-step flow in `setup.html` driven by `setup-app.js`. Step transitions and validation messages are set via DOM manipulation — look for `innerText`, `textContent`, or `innerHTML` assignments.
- **Status banners**: Inline `<div>` elements toggled with CSS classes (e.g., `.error`, `.success`, `.warning`) in `styles.css`. Copy lives in `setup.html` or is injected dynamically in `setup-app.js`.
- **Loading states**: `loading.html` is served during gateway startup. Its message is static HTML — edit directly.
- **Server-side flash messages**: Some error and redirect messages originate in `server.js` as JSON `{ ok, error }` responses consumed by `setup-app.js`. Trace from the fetch call back to the API handler to find the source string.
- **Auth prompts**: The Basic auth challenge copy is browser-native (HTTP 401 + `WWW-Authenticate` header) and cannot be customized without switching to a custom login form.

## Common Patterns

**Update a static banner in the wizard:**
```html
<!-- src/public/setup.html -->
<div class="banner banner--info">
  Your updated message here.
</div>
```

**Change dynamically injected copy in JS:**
```js
// src/public/setup-app.js
statusEl.textContent = "Connecting to gateway — this may take up to 60 seconds.";
```

**Adjust an error message returned from the API:**
```js
// src/server.js — inside the relevant route handler
return res.status(400).json({ ok: false, error: "Descriptive, actionable error copy." });
```

**Refine the loading screen message:**
```html
<!-- src/public/loading.html -->
<p class="loading__message">Starting OpenClaw gateway&hellip;</p>
```

**Locate all user-visible strings across the UI:**
```bash
grep -r "textContent\|innerHTML\|innerText\|\.error\b" src/public/setup-app.js
grep -r 'json.*error' src/server.js
```