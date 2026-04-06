---
name: crafting-page-messaging
description: Writes conversion-focused messaging for pages and key CTAs in the OpenClaw Railway Template UI
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Crafting-page-messaging Skill

Writes and refines conversion-focused copy for the OpenClaw Railway Template's browser UI — including the `/setup` wizard, loading states, error messages, and CTA buttons — so that developer-operators move quickly from deployment to a working OpenClaw instance.

## Quick Start

1. Read the target page (`src/public/setup.html`, `loading.html`, `logs.html`, etc.)
2. Identify the user's current lifecycle state: unconfigured → onboarding → gateway starting → proxying
3. Rewrite headlines, body copy, and CTA labels to match the state and reduce friction
4. Edit the file in place; no build step required

## Key Concepts

**Lifecycle-aware copy** — every screen belongs to one of four states. Copy must match the state: don't show "Almost there!" while the user is still on step 1.

| State | Screen | Primary job |
|---|---|---|
| Unconfigured | `/setup` wizard | Reduce setup anxiety; make the path obvious |
| Onboarding running | `loading.html` | Set time expectations; surface progress |
| Gateway starting | loading/status UI | Reassure; show what's happening |
| Ready | Redirect / success | Confirm success; tell them what to do next |

**Audience** — developers deploying on Railway. They understand Docker and env vars but may not know OpenClaw internals. Skip jargon; explain consequences, not mechanisms.

**CTA hierarchy** — each screen has one primary CTA and at most one secondary escape hatch (e.g., "View logs" or "Reset"). Never compete with two equal-weight buttons.

**Error copy** — always answer three questions: what failed, why (if known), what to do next. Avoid generic "Something went wrong."

## Common Patterns

**Wizard step headline** — action verb + outcome, not feature name:
- Instead of: *Gateway Configuration*
- Use: *Connect your AI assistant to the internet*

**CTA label** — verb + object, present tense:
- Instead of: *Submit* / *Next*
- Use: *Start onboarding* / *Save and launch gateway*

**Loading message** — progress verb + estimated scope:
- Instead of: *Loading…*
- Use: *Starting gateway — usually takes 15–30 seconds*

**Inline error** — past tense cause + imperative fix:
- Instead of: *Error: token_missing*
- Use: *Gateway rejected the connection — check that OPENCLAW_GATEWAY_TOKEN matches the value in Railway Variables, then restart.*

**Empty / success state** — confirm the win, then point forward:
- *Your OpenClaw gateway is live. Open the Control UI to start your first session.*