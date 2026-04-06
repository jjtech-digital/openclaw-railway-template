---
name: generating-growth-hypotheses
description: Generates channel experiments and growth loops for the OpenClaw Railway Template, mapping acquisition channels to onboarding touchpoints and proposing testable growth experiments.
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Generating-growth-hypotheses Skill

This skill generates structured growth hypotheses for the OpenClaw Railway Template by analyzing existing channel integrations (Telegram, Discord, Slack), setup wizard funnel steps, and gateway onboarding flows to produce prioritized, testable acquisition and retention experiments.

## Quick Start

1. Identify active channels by reading `src/server.js` channel config logic and `src/public/setup.html` channel fields
2. Map the current onboarding funnel: `/setup` wizard → `openclaw onboard` → gateway start → first proxied request
3. Generate hypotheses as structured experiment cards with: channel, hypothesis, metric, test, expected lift
4. Prioritize by reach × impact × confidence

## Key Concepts

- **Channel experiments**: Tests targeting Telegram bot DMs, Discord guild onboarding, Slack workspace installs, or the browser `/setup` wizard directly
- **Growth loops**: Closed feedback cycles where activated users trigger new user acquisition (e.g., shared Railway template links, bot invite flows)
- **Activation metric**: First successful proxied request through the gateway — this is the core activation event
- **Funnel stages**: Awareness → Template deploy → `/setup` completion → Gateway ready → Channel connected → First agent task

## Common Patterns

### Hypothesis card format

```
Channel: <telegram|discord|slack|web>
Hypothesis: If we <change>, then <users> will <action> because <reason>
Metric: <primary KPI> (e.g., setup completion rate, time-to-first-proxy)
Test: <what to build or modify> (e.g., add inline token instructions to setup.html step 3)
Expected lift: <X% improvement in metric>
Baseline: <current state from code/logs>
```

### Sourcing baseline data

- Setup funnel drop-off: audit step transitions in `src/public/setup-app.js`
- Channel config success rate: check error paths in the `/setup/api/run` handler in `src/server.js`
- Gateway readiness latency: review polling logic in `waitForGatewayReady` in `src/server.js`

### Loop mechanics to explore

- **Invite loop**: Bot responds to first user message with a Railway deploy link
- **Workspace loop**: Slack/Discord bot prompts admins to share the template with other workspace members
- **Referral hook**: Embed a referral token in the Railway template URL generated post-setup
- **Reactivation**: Use gateway health webhook to trigger re-onboarding nudge if gateway goes dark