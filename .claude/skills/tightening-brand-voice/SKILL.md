---
name: tightening-brand-voice
description: Refines copy in the OpenClaw Railway Template for clarity, tone, and consistency across UI pages, setup wizard text, and user-facing messages.
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Tightening-brand-voice Skill

Audits and rewrites user-facing copy in the OpenClaw Railway Template — setup wizard labels, error messages, loading states, button text, and inline guidance — to be direct, consistent, and technically confident without being terse or jargon-heavy.

## Quick Start

1. Identify copy surfaces: `src/public/setup.html`, `src/public/tui.html`, `src/public/logs.html`, `src/public/loading.html`, and user-visible strings in `src/server.js`.
2. Grep for copy patterns: `Grep pattern="placeholder|innerText|textContent|\.send\(|res\.json" glob="src/**/*"`.
3. Apply edits using the Edit tool; do not rewrite surrounding logic.
4. Verify no secrets or tokens appear in copy changes.

## Key Concepts

- **Voice**: Confident, minimal, developer-friendly. Avoid marketing fluff ("seamlessly", "easily", "simply").
- **Tense**: Present tense for status; imperative for actions ("Connect your bot", not "You should connect your bot").
- **Consistency**: Use "OpenClaw" (not "openclaw" or "Openclaw") in user-visible text. Use "gateway" (lowercase) for the internal process.
- **Error messages**: State what failed and what to do next. Avoid bare codes or stack traces in the UI.
- **Placeholders**: Match the expected format exactly (e.g., `e.g. 8:xxxxxxx` for Telegram tokens).

## Common Patterns

**Button labels** — verb-first, no trailing punctuation:
- Before: `"Submit Configuration"` → After: `"Save and continue"`
- Before: `"Click here to restart"` → After: `"Restart gateway"`

**Status messages** — factual, no ellipsis abuse:
- Before: `"Please wait, loading..."` → After: `"Starting gateway"`
- Before: `"Something went wrong!"` → After: `"Gateway did not start. Check logs for details."`

**Section headings** — title case, no verbs unless a step label:
- Before: `"Configure your channels below"` → After: `"Channel Setup"`

**Inline help text** — one sentence max, no restating the label:
- Before: `"Enter your bot token here. This is the token you got from BotFather."` → After: `"Token from BotFather — format: 123456:ABCdef..."`