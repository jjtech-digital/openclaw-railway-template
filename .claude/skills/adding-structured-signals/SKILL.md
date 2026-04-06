---
name: adding-structured-signals
description: Adds structured data markup (JSON-LD, Schema.org) to HTML pages for rich search results
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Adding-structured-signals Skill

This skill adds structured data signals (JSON-LD blocks using Schema.org vocabulary) to the OpenClaw Railway Template's static HTML pages (`setup.html`, `tui.html`, `logs.html`, `loading.html`) so search engines and crawlers can parse rich metadata from the UI.

## Quick Start

1. Identify the target HTML file under `src/public/`
2. Read the file to understand existing `<head>` structure
3. Insert a `<script type="application/ld+json">` block inside `<head>` with the appropriate Schema.org type
4. Validate JSON syntax before saving
5. Run `npm run lint` to confirm server.js is unaffected

## Key Concepts

- **JSON-LD**: Preferred format — inlined in `<script type="application/ld+json">`, no attribute pollution
- **Schema.org types**: Use `WebApplication`, `WebPage`, or `SoftwareApplication` for app UI pages
- **Placement**: Always inside `<head>`, after `<meta charset>` but before external stylesheets
- **No build step**: These are plain HTML files served statically — edits take effect immediately on next request
- **Scoping**: Each page gets its own block scoped to that page's function (e.g., `setup.html` → onboarding wizard, `logs.html` → log viewer)

## Common Patterns

### WebApplication block for setup.html
```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "WebApplication",
  "name": "OpenClaw Setup Wizard",
  "applicationCategory": "DeveloperApplication",
  "operatingSystem": "Any",
  "url": "/setup"
}
</script>
```

### WebPage block for logs.html
```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "WebPage",
  "name": "OpenClaw Logs Viewer",
  "description": "Live log stream for the OpenClaw gateway process",
  "url": "/logs"
}
</script>
```

### Locating all HTML pages
```bash
# Find all static HTML files
find src/public -name "*.html"
```

### Verifying no inline scripts were broken
```bash
npm run lint
```