---
name: crafting-empty-states
description: Creates empty states and onboarding affordances for the OpenClaw Railway Template UI pages
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Crafting-empty-states Skill

This skill guides the creation of empty states and onboarding affordances across the OpenClaw Railway Template's browser UI — covering the `/setup` wizard, `/logs` viewer, `/tui` terminal, and `/loading` indicator. It produces contextual placeholder UI that orients first-time users and surfaces actionable next steps when content or gateway state is absent.

## Quick Start

1. Identify which page needs an empty state (`src/public/setup.html`, `logs.html`, `tui.html`, or `loading.html`)
2. Read the existing HTML structure and `styles.css` to match visual conventions
3. Add an empty-state container that is shown/hidden based on JS state in `setup-app.js` or inline scripts
4. Ensure the affordance links to the relevant action (e.g., redirect to `/setup`, show a "Start Setup" button, or display a spinner with status text)

## Key Concepts

- **Lifecycle awareness**: Empty states must reflect the current lifecycle phase — unconfigured (no `openclaw.json`) redirects all non-`/setup` routes to `/setup`; configured but gateway not ready shows a loading/polling state
- **Vanilla JS only**: No build step — client logic lives in `setup-app.js` or inline `<script>` tags; use `document.querySelector` and `classList` toggling to show/hide empty-state elements
- **Shared styles**: All pages import `styles.css`; add empty-state utility classes there rather than inline styles on individual pages
- **Auth context**: `/logs` and `/tui` require setup auth; empty states on those pages should not leak configuration details to unauthenticated users
- **Gateway readiness**: The gateway polls multiple endpoints before traffic is proxied; a loading empty state should communicate this wait rather than showing an error

## Common Patterns

**Conditional visibility toggle (JS)**
```js
const isEmpty = !items || items.length === 0;
document.getElementById('empty-state').classList.toggle('hidden', !isEmpty);
document.getElementById('content').classList.toggle('hidden', isEmpty);
```

**Empty state HTML block**
```html
<div id="empty-state" class="empty-state" hidden>
  <p class="empty-state__message">No logs yet. Start the gateway to see output here.</p>
  <a href="/setup" class="btn btn-primary">Go to Setup</a>
</div>
```

**CSS utility class (in `styles.css`)**
```css
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 2rem;
  color: var(--color-muted, #888);
  text-align: center;
}
.hidden { display: none !important; }
```

**Loading/polling affordance (for gateway startup)**
```html
<div id="loading-state" class="empty-state">
  <div class="spinner"></div>
  <p>Waiting for gateway to become ready…</p>
</div>
```

**Onboarding CTA (unconfigured state)**
```html
<div id="unconfigured-state" class="empty-state">
  <h2>Welcome to OpenClaw</h2>
  <p>Complete the setup wizard to get started.</p>
  <a href="/setup" class="btn btn-primary">Start Setup</a>
</div>
```