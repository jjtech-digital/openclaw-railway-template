---
name: scaling-template-pages
description: Builds scalable, template-driven search pages for the OpenClaw Railway Template static UI
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Scaling-template-pages Skill

Generates and extends template-driven pages in `src/public/` — search, listing, or filtered views — following the project's vanilla-JS, no-build-step conventions. Pages share `styles.css` and follow the same HTML structure as `setup.html`, `logs.html`, and `tui.html`.

## Quick Start

1. Read an existing page (`logs.html` or `tui.html`) to understand the HTML shell pattern.
2. Create the new `.html` file in `src/public/` with a matching structure.
3. Add a companion `-app.js` file for client-side logic (vanilla JS, ES modules via `<script type="module">`).
4. Register a route in `src/server.js` using `express.static` or an explicit `res.sendFile` handler, protected with `requireSetupAuth` if needed.
5. Run `npm run lint` to validate server-side changes.

## Key Concepts

- **No build step** — all client JS is vanilla ES modules loaded directly by the browser.
- **Shared styles** — link `styles.css` from every page; do not embed inline styles.
- **Auth boundary** — pages under `/setup/*` are protected by `requireSetupAuth` middleware; add new protected pages there.
- **Static serving** — `src/public/` is served via `express.static`; file names become URL paths automatically.
- **Search pattern** — filter arrays client-side with `Array.prototype.filter` + `input` event listeners; avoid server-side search endpoints unless the dataset is large.

## Common Patterns

### Filtering a list client-side
```js
const input = document.getElementById('search');
const items = document.querySelectorAll('.item');
input.addEventListener('input', () => {
  const q = input.value.toLowerCase();
  items.forEach(el => {
    el.hidden = !el.textContent.toLowerCase().includes(q);
  });
});
```

### Fetching data from an existing API endpoint
```js
const res = await fetch('/setup/api/debug', {
  headers: { Authorization: 'Basic ' + btoa(':' + password) }
});
const data = await res.json();
```

### Registering a new protected page in server.js
```js
app.get('/setup/mypage', requireSetupAuth, (req, res) => {
  res.sendFile(new URL('../public/mypage.html', import.meta.url).pathname);
});
```

### Adding a nav link from the setup wizard
Add an anchor inside the existing nav or tools section of `setup.html`; no JS changes needed for static links.