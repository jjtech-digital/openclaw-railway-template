---
name: reducing-form-falloff
description: Improves lead capture forms to reduce drop-off by auditing field count, error messaging, inline validation, and progressive disclosure in the OpenClaw Railway Template setup wizard.
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Reducing-form-falloff Skill

Audits and improves the `/setup` wizard forms in `src/public/setup.html` and `src/public/setup-app.js` to reduce user drop-off. Focuses on minimizing required fields, surfacing clear inline validation errors, improving placeholder copy, and applying progressive disclosure so users only see fields relevant to their chosen auth provider or channel type.

## Quick Start

1. Read `src/public/setup.html` to inventory all form fields and steps.
2. Read `src/public/setup-app.js` to understand current validation logic and step transitions.
3. Read `src/public/styles.css` to understand available error/success state classes.
4. Identify fields that can be deferred, removed, or collapsed behind a toggle.
5. Apply changes to HTML structure, client-side validation, and CSS as needed.

## Key Concepts

- **Field count**: Every extra field increases drop-off. Remove or make optional any field not strictly required for onboarding (e.g., channel configs can be deferred post-setup).
- **Progressive disclosure**: Show channel-specific fields (Telegram token, Discord bot token, Slack webhook) only when the user selects that channel type — already partially handled via JS show/hide; ensure all edge cases are covered.
- **Inline validation**: Validate on `blur`, not just on submit. Show errors adjacent to the field with a clear, actionable message (e.g., "Token must be 64 hex characters" not "Invalid input").
- **Error recovery**: Pre-fill fields from `localStorage` on page reload so users don't lose progress after a failed submit.
- **CTA clarity**: The primary action button label should reflect the current step outcome (e.g., "Connect Telegram" not "Next").

## Common Patterns

**Add blur validation to an input:**
```javascript
document.getElementById('telegramToken').addEventListener('blur', (e) => {
  const val = e.target.value.trim();
  const err = document.getElementById('telegramToken-error');
  if (val && !/^\d+:[A-Za-z0-9_-]{35,}$/.test(val)) {
    err.textContent = 'Enter a valid Telegram bot token (e.g. 123456:ABC-DEF...)';
    err.hidden = false;
  } else {
    err.hidden = true;
  }
});
```

**Progressive disclosure for channel fields:**
```javascript
document.getElementById('channelSelect').addEventListener('change', (e) => {
  document.querySelectorAll('.channel-fields').forEach(el => el.hidden = true);
  const target = document.getElementById(`${e.target.value}-fields`);
  if (target) target.hidden = false;
});
```

**Persist partial form state:**
```javascript
function saveProgress() {
  const fields = ['authProvider', 'telegramToken', 'discordToken'];
  const data = Object.fromEntries(fields.map(id => {
    const el = document.getElementById(id);
    return [id, el ? el.value : ''];
  }));
  localStorage.setItem('setup-progress', JSON.stringify(data));
}
```

**Error element convention** (matches existing `styles.css` patterns):
```html
<input id="telegramToken" type="password" autocomplete="off" placeholder="123456:ABC-DEF1234...">
<span id="telegramToken-error" class="field-error" hidden aria-live="polite"></span>
```