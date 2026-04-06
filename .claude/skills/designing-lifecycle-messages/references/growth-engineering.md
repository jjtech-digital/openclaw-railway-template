# Growth Engineering

## When to use
When building or modifying in-product mechanisms that drive organic growth, referrals, or expansion from an existing deployment (e.g., adding team members, connecting additional channels).

## Patterns

**Volume persistence as a trust signal**
Stable gateway tokens and volume-persisted config are a structural growth enabler: users who trust their setup survives redeploys are more likely to invite teammates and expand channel usage. Lifecycle messages should reinforce this: "Your setup is locked to this Railway volume — redeploys won't reset your channels or pairings."

**Multi-channel expansion nudge**
After a user's first channel is active, a 48h follow-up nudge for a second channel (e.g., "You're live on Telegram — want to add Slack for your team?") is a natural expansion vector. The `/setup` wizard supports all three channels; the nudge can link directly to the wizard's channel configuration step.

**Debug console as a power-user retention hook**
The debug console (`/setup` → Tools) is a differentiator for technical users. Highlighting its existence in a "did you know" message 3–7 days post-activation retains users who would otherwise self-host a raw OpenClaw instance. Frame it as control, not complexity.

## Pitfalls
Do not push multi-channel expansion before the first channel is confirmed working. A user who hasn't received their first bot message yet is in a fragile state — growth nudges at this stage read as spam and increase churn risk.