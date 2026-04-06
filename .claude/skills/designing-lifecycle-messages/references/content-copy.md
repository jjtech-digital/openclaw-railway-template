# Content & Copy

## When to use
When drafting subject lines, body copy, or in-app text for any lifecycle message tied to OpenClaw deployment stages.

## Patterns

**Match copy to lifecycle state, not elapsed time**
Copy should reflect what the system actually knows: unconfigured, wizard complete, gateway ready, or channel active. "Your gateway is warming up" is accurate during startup; "Your AI assistant is live" is accurate after the health check passes. Time-based copy ("Day 3 email") risks mismatching the user's real state.

**Never surface authentication details**
Users never handle bearer tokens — the wrapper injects them transparently. Copy should not mention tokens, `Authorization` headers, or pairing flows. If auth is relevant (e.g., a pairing error), redirect to `/setup` → Tools → Debug Console rather than explaining the mechanism.

**Channel-specific voice**
- Telegram: casual, command-driven ("Send `/start` to your bot")
- Discord: permission-forward ("Check MESSAGE CONTENT INTENT is enabled in the Developer Portal")
- Slack: workspace-context ("Your bot is now in your workspace — try mentioning it in any channel")

## Pitfalls
Avoid generic SaaS copy ("Unlock the power of AI"). Users chose a self-hosted Railway deployment for control and privacy — copy that emphasizes ownership and persistence ("your config survives every redeploy") resonates more than cloud-platform language.