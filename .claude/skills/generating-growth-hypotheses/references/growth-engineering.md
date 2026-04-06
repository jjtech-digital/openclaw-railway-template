# Growth Engineering

## When to use
When implementing closed-loop growth mechanics that require code changes to `src/server.js`, `src/public/`, or the Railway template configuration.

## Patterns

### Reactivation via gateway health webhook
If the gateway process exits (tracked via `gatewayProc` in `src/server.js`), emit a webhook or bot message to prompt the user to redeploy or re-run setup. This turns churn events into re-engagement touchpoints. Implement in the `gatewayProc.on("exit")` handler.

### Post-setup referral token generation
After `openclaw onboard` completes and `openclaw.json` is written, generate a referral token (`crypto.randomBytes(8).toString("hex")`) and embed it in the success screen. Store it alongside `gateway.token` in `STATE_DIR` for persistence across redeploys.

### Growth loop wiring checklist
Before shipping any loop mechanic:
- [ ] Loop entry point is reachable after activation (not during setup)
- [ ] Loop message/link is sent at most once per instance (use a flag file in `STATE_DIR`)
- [ ] Referral tokens are stable across redeploys (persisted to volume, not regenerated)
- [ ] No secrets or internal URLs are embedded in user-facing referral links

## Pitfall
Growth mechanics that fire during setup (before gateway is ready) will confuse users who haven't activated yet. Gate all loop triggers on confirmed gateway readiness — check `isConfigured()` and gateway status before emitting any referral or invite content.