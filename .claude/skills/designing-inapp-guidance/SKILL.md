---
name: designing-inapp-guidance
description: Builds tooltips, tours, and contextual guidance for the OpenClaw Railway Template setup wizard and browser UI
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Designing-inapp-guidance Skill

This skill guides implementation of tooltips, step-by-step tours, and contextual help within the OpenClaw Railway Template's browser-first UI — specifically the `/setup` wizard (`src/public/setup.html`, `src/public/setup-app.js`) and supporting pages (`tui.html`, `logs.html`). All guidance is implemented in vanilla JS with no build step, styled via `src/public/styles.css`.

## Quick Start

1. Read the target page to understand existing structure:
   - `src/public/setup.html` — wizard steps and form fields
   - `src/public/setup-app.js` — client-side state and step transitions
   - `src/public/styles.css` — shared CSS classes and variables
2. Identify where guidance is needed: form fields, auth provider selection, channel config, or gateway status indicators.
3. Add tooltip markup to `setup.html`, CSS to `styles.css`, and interaction logic to `setup-app.js`.
4. Validate syntax: `npm run lint` (checks `server.js`); manually open pages in browser for UI review.

## Key Concepts

- **No build step**: All JS is vanilla ES modules loaded directly by the browser. No bundler, no TypeScript, no React.
- **Wizard steps**: The setup wizard progresses through discrete steps managed in `setup-app.js`. Contextual guidance should be scoped to the active step.
- **Auth layers**: Users encounter two auth concepts — `SETUP_PASSWORD` (Basic auth for `/setup`) and the gateway bearer token. Guidance must clearly distinguish these.
- **Channel config complexity**: Telegram/Discord/Slack each have unique fields and external prerequisites (bot tokens, intents). Inline help here reduces setup failures.
- **CSS conventions**: Use existing CSS custom properties and class patterns from `styles.css`; avoid inline styles.

## Common Patterns

### Tooltip on a form field
```html
<!-- In setup.html -->
<label for="gatewayToken">
  Gateway Token
  <span class="tooltip" aria-label="Auto-generated if left blank. Set via Railway Variables for stable cross-redeploy pairing.">?</span>
</label>
<input id="gatewayToken" type="text" placeholder="Leave blank to auto-generate" />
```

```css
/* In styles.css */
.tooltip {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 1rem;
  height: 1rem;
  border-radius: 50%;
  background: var(--color-muted);
  cursor: default;
  font-size: 0.75rem;
  position: relative;
}
.tooltip::after {
  content: attr(aria-label);
  position: absolute;
  bottom: 125%;
  left: 50%;
  transform: translateX(-50%);
  background: var(--color-surface);
  border: 1px solid var(--color-border);
  padding: 0.4rem 0.6rem;
  border-radius: 4px;
  white-space: nowrap;
  font-size: 0.8rem;
  display: none;
  z-index: 10;
}
.tooltip:hover::after { display: block; }
```

### Step-scoped contextual banner
```javascript
// In setup-app.js — show help text when entering a step
function onStepEnter(stepId) {
  const hints = {
    'step-channels': 'Select the messaging platform your team uses. Each channel requires a bot token from the provider.',
    'step-gateway': 'The gateway token secures communication between the browser and OpenClaw. Stable tokens prevent pairing loss on redeploy.',
  };
  const banner = document.getElementById('step-hint');
  if (banner) banner.textContent = hints[stepId] ?? '';
}
```

### Discord intent warning
```javascript
// Inline warning when Discord is selected — MESSAGE CONTENT INTENT must be enabled
authSelect.addEventListener('change', () => {
  const discordWarning = document.getElementById('discord-intent-warning');
  if (discordWarning) {
    discordWarning.hidden = authSelect.value !== 'discord';
  }
});
```

```html
<p id="discord-intent-warning" hidden class="warning">
  Discord bots require <strong>Message Content Intent</strong> enabled in the Discord Developer Portal under your bot's settings.
</p>
```