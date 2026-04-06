# Growth Engineering

## When to use
When release assets need to drive adoption of a new capability (not just announce a fix). Apply when a release introduces a feature that changes the operator's workflow or unlocks a previously unavailable use case.

## Patterns

**Activation milestone framing**
Map each new feature to its first activation milestone — the moment an operator knows the feature is working:
- Pairing support → first successful paired device shown in Control UI
- TUI endpoint → first terminal session opened at `/tui`
- Channel config → first message received via Telegram/Discord/Slack

Use this milestone as the final line of the feature highlight: "You'll know it's working when…"

**Reduce time-to-first-value in rollout checklist**
Order checklist steps so the operator reaches a working state as fast as possible. Railway variable checks before Docker build; health endpoint before wizard walkthrough. Operators who reach a working state faster are more likely to configure optional features.

**Surface optional capabilities explicitly**
Features behind env flags (`ENABLE_WEB_TUI`, `OPENCLAW_GATEWAY_TOKEN`) are invisible unless called out. Add a "What you can enable next" section to release notes for capabilities that are off by default.

## Pitfalls
Do not frame every release as a growth moment. Bug fixes and stability improvements build trust differently than features. Overclaiming growth impact on maintenance releases erodes operator trust in release copy over time.