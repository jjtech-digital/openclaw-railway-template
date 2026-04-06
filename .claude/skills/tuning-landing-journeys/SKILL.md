---
name: tuning-landing-journeys
description: Improves landing page flow, hierarchy, and conversion paths for the OpenClaw Railway Template setup wizard and browser UI
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Tuning-landing-journeys Skill

This skill audits and improves the landing and onboarding flow of the OpenClaw Railway Template — covering the `/setup` wizard, loading states, and the path from first visit to a running gateway. It focuses on visual hierarchy, copy clarity, step sequencing, and reducing drop-off between unconfigured and configured states.

## Quick Start

1. Read `src/public/setup.html` and `src/public/setup-app.js` to understand current wizard steps and form structure.
2. Read `src/public/styles.css` to understand visual hierarchy and component styling.
3. Read `src/public/loading.html` to review transition states between setup completion and gateway readiness.
4. Identify friction points: unclear CTAs, missing progress indicators, ambiguous field labels, or dead-end error states.
5. Edit HTML, CSS, and client JS in `src/public/` to improve flow — no build step required.

## Key Concepts

- **Lifecycle gates**: The template has two hard states — unconfigured (redirects all traffic to `/setup`) and configured (proxies to gateway). Landing improvements must respect these gates; users cannot reach `/openclaw` until setup completes.
- **Setup wizard is the landing page**: For new deployments, `/setup` is the first and only page users see. Hierarchy, copy, and step order here directly determine activation rate.
- **No framework, no build**: All client-side code is vanilla JS in `src/public/setup-app.js`. Improvements ship as plain HTML/CSS/JS edits.
- **Auth layer is always present**: The setup wizard is behind HTTP Basic auth (`SETUP_PASSWORD`). Landing improvements apply to the post-auth experience, not the browser credential prompt.
- **Loading state is a conversion moment**: `loading.html` is shown while the gateway starts. Copy and progress feedback here reduces abandonment during the 10–60 second wait.

## Common Patterns

**Clarify step sequencing**
Add a step indicator (e.g., "Step 2 of 4") to wizard sections in `setup.html` so users know how far they are from completion.

**Improve CTA copy**
Replace generic button labels ("Submit", "Next") with action-oriented labels ("Connect Telegram", "Start Gateway") that reflect what happens on click.

**Surface inline validation early**
In `setup-app.js`, validate required fields on blur rather than only on submit — reduces frustration at the final step.

**Add context to loading states**
In `loading.html`, show what the system is doing ("Starting gateway…", "Waiting for OpenClaw to respond…") rather than a generic spinner. Gateway readiness polling is in `src/server.js`; the `/setup/api/status` endpoint can be polled from the client for real-time feedback.

**Reduce field cognitive load**
Group optional channel fields (Telegram/Discord/Slack) behind a disclosure or conditional section — show only what the user opted into during auth provider selection.

**Error recovery paths**
Ensure every error state in `setup-app.js` includes a clear next action (retry button, link to logs at `/logs`, or a reset option) rather than leaving users at a dead end.