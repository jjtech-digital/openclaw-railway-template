---
name: embedding-decision-cues
description: Applies behavioral cues to UI and copy in the OpenClaw Railway Template that improve user conversion decisions during setup and onboarding.
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Embedding-decision-cues Skill

This skill embeds behavioral conversion cues — progress indicators, social proof, friction reducers, and urgency signals — into the OpenClaw Railway Template's setup wizard and onboarding UI (`src/public/setup.html`, `src/public/setup-app.js`, `src/public/styles.css`) to increase the likelihood that users complete setup and activate the gateway.

## Quick Start

1. Read the current setup wizard: `src/public/setup.html` and `src/public/setup-app.js`
2. Identify the target conversion moment (e.g., auth provider selection, channel config, gateway launch)
3. Apply the appropriate cue pattern from the list below
4. Validate no inline styles or scripts are added to `server.js` — keep all UI changes in `src/public/`
5. Run `npm run lint` to confirm server syntax is unchanged

## Key Concepts

**Conversion moment**: A step in the `/setup` wizard where a user must make a decision — choosing an auth provider, entering a channel token, or clicking "Launch Gateway."

**Friction reducer**: Any copy, UI element, or default value that lowers the perceived cost of completing a step. Example: pre-filling recommended values, adding helper text explaining why a field is required.

**Progress signal**: Visual or textual feedback that tells the user how far they are and what comes next. The wizard's step flow (`step-1` → `step-2` → ...) is the natural anchor for these.

**Commitment cue**: An element that reinforces a decision already made, reducing abandonment after the user starts. Example: a summary of selected options shown before the final "Run Setup" action.

**Success signal**: Immediate positive feedback after gateway launch (e.g., the loading state in `loading.html` and the redirect to `/openclaw`).

## Common Patterns

### Add a step progress indicator
Inject a `<div class="progress-steps">` block at the top of each wizard step in `setup.html`, reflecting current position. Update active state via `setup-app.js` when the step changes.

### Reduce field anxiety with helper text
Add `<small class="field-hint">` beneath sensitive inputs (e.g., bot tokens, API keys) explaining what the value is used for and that it is stored only on the user's Railway volume.

### Show a pre-launch commitment summary
Before the final setup submission, render a read-only summary card listing the auth provider and enabled channels. This uses data already collected in `setup-app.js` state — no new API calls needed.

### Reinforce gateway launch with outcome copy
Replace generic "Starting…" text in `loading.html` with specific outcome language: "Connecting your AI assistant — this takes about 30 seconds on first run."

### Default to the lowest-friction auth option
In `setup-app.js`, pre-select the auth provider with the fewest required fields. Ensure the selection is visible and labeled as "Recommended" in `setup.html`.

### Surface the "what happens next" expectation
At the end of each step, add a one-line preview of the next step so users are not surprised. Add this as a `<p class="next-step-hint">` element and style it via `styles.css`.