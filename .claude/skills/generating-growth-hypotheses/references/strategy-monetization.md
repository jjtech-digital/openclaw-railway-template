# Strategy & Monetization

## When to use
When evaluating which growth hypotheses align with platform constraints (Railway free tier, OpenClaw licensing) or when framing upgrade moments in the setup wizard UI.

## Patterns

### Activation-first sequencing
Growth investment follows activation rate. Prioritize hypotheses that improve time-to-first-proxy before hypotheses that target top-of-funnel acquisition. A leaky activation funnel makes all acquisition spend wasteful.

### Upgrade moment placement
The highest-intent moment for an upgrade prompt is immediately after the first successful proxied request — the user has just seen value. In the proxy handler (`src/server.js`), this is the first non-error response forwarded to the browser. A follow-up message via connected bot channel ("You're live — want to add more channels?") is less intrusive than an in-UI modal.

### Funnel stage → growth lever mapping
| Stage | Lever | Where to implement |
|-------|-------|-------------------|
| Template deploy | Template marketplace listing copy | Railway template metadata |
| Setup completion | Wizard UX friction reduction | `setup.html`, `setup-app.js` |
| Gateway ready | Time-to-ready improvement | `waitForGatewayReady` polling |
| Channel connected | Invite loop trigger | `src/server.js` post-onboard handler |
| First agent task | Referral hook | Post-activation success screen |

## Pitfall
Railway free-tier resource limits (memory, CPU, volume size) constrain what growth mechanics are viable for entry-level users. Don't design loops that require persistent background processes or large log storage — they'll degrade the experience on free plans before the user ever hits an upgrade moment.