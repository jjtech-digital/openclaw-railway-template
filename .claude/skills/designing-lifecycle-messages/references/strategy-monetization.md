# Strategy & Monetization

## When to use
When framing the value proposition in lifecycle messages, or when designing upgrade prompts tied to Railway plan limits or OpenClaw feature tiers.

## Patterns

**Lead with operational control, not AI features**
The Railway template's core value is self-hosted, persistent, browser-accessible deployment — not the AI model itself. Lifecycle messaging should emphasize: "You own the deployment, the config, and the data." This differentiates from managed SaaS alternatives and justifies Railway infrastructure cost.

**Tie upgrade prompts to observable limits**
If a user hits Railway's free-tier volume or compute limits, the signal will appear in `/logs` as gateway restarts or timeout errors. An in-app banner ("Seeing restarts? Your Railway plan may be hitting compute limits — consider upgrading your Railway service") is more credible than a generic upsell because it references a concrete, visible symptom.

**OpenClaw version as a retention hook**
The wrapper pins an OpenClaw version in the Dockerfile (`openclaw@2026.x.x`). Version upgrade announcements (tied to `OPENCLAW-VERSION-CONTROL.md` guidance) are a recurring engagement touchpoint. Frame each upgrade as new capabilities available in the same self-hosted setup the user already trusts.

## Pitfalls
Do not conflate Railway cost with OpenClaw value. Users pay Railway for infrastructure; OpenClaw is the application layer. Lifecycle messages that blur this distinction ("upgrade to get more AI") will confuse users about what they are actually paying for. Keep the two value propositions distinct.