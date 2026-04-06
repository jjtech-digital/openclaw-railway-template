---
name: strengthening-upgrade-moments
description: Improves upgrade prompts and paywall messaging in the OpenClaw Railway Template UI to increase conversion at friction points.
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Strengthening-upgrade-moments Skill

This skill audits and rewrites upgrade prompts, gate messages, and paywall copy across the OpenClaw Railway Template UI. It identifies where users hit limits or locked features and sharpens the messaging to communicate value, reduce friction, and motivate action.

## Quick Start

1. Grep for existing upgrade/limit copy in `src/public/`:
   ```bash
   grep -ri "upgrade\|limit\|pro\|plan\|unlock\|paywall" src/public/
   ```
2. Read relevant HTML files (`setup.html`, `tui.html`, `logs.html`) to understand context.
3. Identify the user's state at the moment the message appears (onboarding, post-setup, feature gate).
4. Rewrite copy using the patterns below.
5. Verify no inline HTML was introduced into `src/server.js` — all UI copy belongs in `src/public/`.

## Key Concepts

- **Moment of friction**: The point where a user is blocked or nudged — setup wizard limits, TUI session expiry, feature flags like `ENABLE_WEB_TUI=false`, or missing channel configs.
- **Value-first framing**: Lead with what the user gains, not what they lack. "Unlock real-time terminal access" beats "TUI requires upgrade."
- **Context awareness**: Messaging must match lifecycle state — unconfigured users need different copy than users hitting post-setup limits.
- **No secrets in copy**: Never reference token values, passwords, or internal env vars in user-facing strings.

## Common Patterns

**Feature gate (disabled endpoint):**
> Before: "TUI is disabled. Set ENABLE_WEB_TUI=true."
> After: "Live terminal access is available on supported plans. Enable it in your Railway Variables to get started."

**Session expiry:**
> Before: "Session expired."
> After: "Your terminal session ended after 30 minutes. Start a new session to continue."

**Channel not configured:**
> Before: "No channels configured."
> After: "Connect a channel — Telegram, Discord, or Slack — to start receiving agent updates."

**Upgrade CTA placement:** Insert upgrade prompts immediately after the user encounters a limit, not before. Gate messages in `setup.html` should appear inline in the relevant wizard step, not as a separate screen.

**Tone:** Direct, developer-friendly, no marketing fluff. One sentence of context, one sentence of action.