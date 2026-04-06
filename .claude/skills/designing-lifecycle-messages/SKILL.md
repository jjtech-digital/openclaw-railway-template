---
name: designing-lifecycle-messages
description: Designs onboarding and lifecycle email sequences for the OpenClaw Railway Template, mapping user journey stages to targeted message content.
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Designing-lifecycle-messages Skill

Designs onboarding and lifecycle email sequences that guide users through OpenClaw deployment stages — from first deploy through setup wizard completion, gateway startup, channel configuration, and ongoing engagement. Messages are grounded in the actual lifecycle states (`unconfigured → onboarding → configured → proxying`) and activation milestones observable in the wrapper.

## Quick Start

1. Identify the lifecycle stage: unconfigured, setup in progress, gateway starting, gateway ready, or channel connected.
2. Map each stage to a user job-to-be-done (e.g., "get AI assistant answering in Telegram").
3. Draft a sequence: welcome → activation nudge → first-value confirmation → re-engagement.
4. Anchor copy to concrete UI paths (`/setup`, `/logs`, `/tui`) and observable outcomes (gateway health, channel messages arriving).

## Key Concepts

- **Lifecycle states**: Unconfigured (no `openclaw.json`) → Setup wizard → Gateway spawned → Gateway ready → Channels active. Each state is a message trigger.
- **Activation milestone**: First successful proxied request to `/openclaw` after gateway is ready — the moment the user gets real value.
- **Two-layer auth UX**: Users never handle bearer tokens; setup wizard abstracts this. Messaging should not mention tokens.
- **Channel diversity**: Telegram, Discord, and Slack are all supported. Tailor channel-specific sequences to each platform's onboarding friction (e.g., Discord requires MESSAGE CONTENT INTENT).
- **Redeploy continuity**: Stable gateway tokens mean device pairings survive redeploys. Lifecycle messaging should reassure users their setup persists.

## Common Patterns

**Welcome / Deploy Confirmation**
Trigger: Railway deployment succeeds, `/healthz` responds.
Goal: Direct user to `/setup` with password. Set expectation that setup takes ~2 minutes.

**Setup Wizard Completion**
Trigger: `openclaw.json` written, gateway spawning.
Goal: Confirm channels are configured, explain the gateway is warming up, link to `/logs` for transparency.

**First Value — Gateway Ready**
Trigger: Gateway health check passes, proxy traffic begins.
Goal: Celebrate the milestone. Show the user how to send their first message via the connected channel.

**Channel-Specific Nudge**
Trigger: Channel configured but no inbound messages after 24h.
Goal: Remind user of required steps (e.g., Discord MESSAGE CONTENT INTENT toggle, Telegram bot `/start` command).

**Re-engagement / Redeploy**
Trigger: New Railway deployment detected (version bump in logs).
Goal: Confirm config and pairings persisted via volume. Reassure no reconfiguration needed unless SETUP_PASSWORD or token changed.