# Strategy & Monetization

## When to use
Apply when an experiment is designed to influence upgrade intent, plan selection framing, or willingness to self-host vs. use a managed tier.

## Patterns

### Value Proposition Framing in Setup
The `/setup` wizard is the first authenticated surface. Test leading with capability framing ("Connect your AI assistant in 3 steps") vs. security framing ("Your data stays on your Railway volume"). Primary metric: setup completion rate; secondary: return visit rate to `/openclaw`.

### Channel Upsell Ordering
Test presenting premium or high-value channels (Slack, Discord) before Telegram in the channel selection UI. Measure which ordering produces higher channel-configuration rates without increasing setup abandonment.

### Post-Setup Upgrade Prompt Timing
After `openclaw.json` is written and the gateway is ready, the redirect lands on `/openclaw`. Test inserting an interstitial with upgrade messaging vs. direct passthrough. Guardrail: gateway error rate and time-to-first-proxy-request must not regress.

## Pitfalls
- Monetization experiments that gate features behind paywalls must not break the core proxy path — all variants must still inject the bearer token correctly via `proxyReq`/`proxyReqWs` handlers.
- Avoid conflating setup completion with monetization intent; a user who completes setup may have no upgrade path available yet. Separate funnel stages in your measurement plan.