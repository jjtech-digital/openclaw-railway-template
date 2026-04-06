---
name: streamlining-signup-steps
description: Reduces friction in the /setup wizard signup and trial activation flow for the OpenClaw Railway Template
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Streamlining-signup-steps Skill

This skill audits and improves the `/setup` wizard onboarding flow in the OpenClaw Railway Template, targeting friction points between first visit and a fully running gateway — covering form UX, validation feedback, step sequencing, error recovery, and time-to-value for new users.

## Quick Start

1. Read `src/public/setup.html` and `src/public/setup-app.js` to understand the current wizard steps and form fields.
2. Read `src/server.js` around the `/setup/api/run` handler (lines 522–693) to understand the server-side onboarding pipeline.
3. Identify friction: required fields with no inline validation, steps that block without clear feedback, errors that don't surface actionable messages.
4. Apply targeted edits — reduce required inputs, improve inline feedback, shorten the path from form submit to gateway ready.

## Key Concepts

- **Onboarding pipeline**: `/setup` wizard collects auth provider + optional channel configs → POSTs to `/setup/api/run` → server runs `openclaw onboard --non-interactive` → writes channel config via `openclaw config set --json` → spawns gateway → polls readiness.
- **Two-layer auth**: Setup wizard uses HTTP Basic auth (`SETUP_PASSWORD`); gateway uses a bearer token injected transparently. Users never handle the gateway token directly.
- **Channel config is optional**: Telegram, Discord, and Slack channels are not required to complete onboarding. The gateway starts without them — surface this clearly in the UI so users aren't blocked.
- **Gateway readiness polling**: The server polls multiple endpoints after spawn. The UI should reflect this async wait honestly with progress feedback, not a spinner with no timeout message.
- **`allowInsecureAuth` is always set**: Device pairing is bypassed by design — do not add pairing steps to the signup flow.

## Common Patterns

**Remove unnecessary required fields**
If a channel section (Telegram/Discord/Slack) is optional, ensure its fields are not `required` in HTML and that `setup-app.js` skips validation for unchecked sections.

```javascript
// setup-app.js: guard channel field collection
if (document.getElementById('enable-telegram')?.checked) {
  payload.telegram = { token: document.getElementById('telegram-token').value.trim() };
}
```

**Inline validation before submit**
Validate `SETUP_PASSWORD` presence and auth provider selection client-side before the POST hits the server, so users get instant feedback.

```javascript
if (!authProvider) {
  showError('auth-error', 'Select an auth provider to continue.');
  return;
}
```

**Progress messaging during gateway startup**
The `/setup/api/run` call is slow (onboarding + gateway spawn + readiness poll). Show a step-by-step status rather than a generic loading state.

```javascript
setStatus('Running onboard…');
const res = await fetch('/setup/api/run', { method: 'POST', body: … });
setStatus('Waiting for gateway…');
```

**Surface actionable errors**
When `/setup/api/run` returns a non-OK response, extract and display `error` from the JSON body rather than a generic failure message so users know what to fix.

```javascript
const data = await res.json();
if (!data.ok) showError('setup-error', data.error ?? 'Setup failed. Check /logs for details.');
```

**Skip redundant confirmation steps**
If a field has a sensible default (e.g., `OPENCLAW_STATE_DIR=/data/.openclaw`), pre-fill it and hide the field unless the user expands an "Advanced" section — don't make every user interact with infrastructure details.