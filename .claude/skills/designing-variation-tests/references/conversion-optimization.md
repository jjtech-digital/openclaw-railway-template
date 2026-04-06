# Conversion Optimization

## When to use
Apply when a hypothesis targets setup completion rate, channel configuration rate, or time-to-gateway-ready — any metric where a UI or flow change is expected to move the needle.

## Patterns

### Setup Completion Funnel
Track drop-off at each wizard step by logging step transitions in `setup-app.js`. The primary conversion event is a successful `/setup/api/run` response (`ok: true`). Compare variant completion rates using chi-squared test.

### CTA Copy Variants
Change button labels or instructional text in `src/public/setup.html`. Keep all other variables constant. Log the variant cohort at page load so exposure is captured before any interaction.

### Progressive Disclosure
Test revealing advanced options (channels, token config) only after basic setup completes. Measure whether reduced initial cognitive load improves `openclaw.json` write success rate.

## Pitfalls
- Do not conflate gateway startup errors with user drop-off — filter `log.error("gateway", ...)` events before computing completion rate.
- Avoid running copy and step-order tests simultaneously; changes confound each other.