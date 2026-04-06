# Content & Copy

## When to use
Apply when testing headline text, instructional labels, error messages, or CTA wording in the setup wizard or loading states.

## Patterns

### Wizard Step Labels
Variants live in `src/public/setup.html`. Change label text only; leave form field names and API payloads identical so instrumentation remains stable across variants.

### Error Message Clarity
Test actionable vs. generic error strings returned from `/setup/api/run`. Primary metric: retry rate (second POST to `/setup/api/run` within the same session). Lower retry rate signals the message resolved confusion.

### Loading State Messaging
`loading.html` is shown while `waitForGatewayReady` polls. Test "Starting your gateway…" vs. step-by-step progress copy. Guardrail: actual gateway startup duration must not change — this is a perception test only.

## Pitfalls
- Copy changes that alter element IDs or `data-*` attributes will break `setup-app.js` selectors — verify with `npm run lint` and a manual setup flow after any HTML edit.
- Avoid testing copy in `/logs` or `/tui` pages; traffic there is too low for significance.