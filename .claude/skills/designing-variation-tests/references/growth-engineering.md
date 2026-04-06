# Growth Engineering

## When to use
Apply when an experiment requires code changes to `server.js`, `setup-app.js`, or route handlers — not just copy edits — to implement a variant.

## Patterns

### Feature-Flag via Environment Variable
Gate variant behaviour with a Railway Variable:
```js
const VARIANT_CHANNEL_DEFAULT = process.env.VARIANT_CHANNEL_DEFAULT || "none";
```
Ship both code paths; flip the variable to activate. Rollback is a Variable change, not a redeploy.

### Onboarding Step Injection
Add or reorder steps by modifying `buildOnboardArgs()` in `server.js`. Keep the control path behind a flag so both variants run in production simultaneously. Log which path executed at the `openclaw onboard` call site.

### Gateway Readiness UX Hook
`waitForGatewayReady` emits log events on each poll. Expose a `/setup/api/status` SSE stream to push progress to the browser. Variants can display richer feedback without changing gateway startup logic.

## Pitfalls
- Never put variant logic inside `proxyReq` or `proxyReqWs` handlers — latency-sensitive path; any added overhead directly impacts perceived performance and contaminates timing metrics.
- Test both variant and control paths with `npm run lint` and a full Docker build before deploying; a broken variant corrupts the experiment and wastes sample.