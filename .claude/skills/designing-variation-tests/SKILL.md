---
name: designing-variation-tests
description: Plans A/B experiments and measurement plans for the OpenClaw Railway Template, mapping hypotheses to UI surfaces, setup wizard steps, and onboarding flows.
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Designing-variation-tests Skill

Plans A/B experiments and measurement plans grounded in the OpenClaw Railway Template's onboarding funnel, setup wizard, and gateway lifecycle. Given a hypothesis or area of interest, this skill produces a structured experiment brief: variant definitions, traffic split, success metrics, guardrail metrics, and a measurement plan tied to observable product events.

## Quick Start

1. State the hypothesis and the surface to test (e.g., `/setup` wizard copy, step order, channel selection UI).
2. Identify the control (current behaviour) and one or more variants.
3. Define the primary metric and minimum detectable effect.
4. Specify guardrail metrics that must not regress.
5. Map instrumentation hooks to existing log categories (`log.info("setup", ...)`, `log.info("gateway", ...)`).
6. Set experiment duration based on estimated daily unique visits and required sample size.

## Key Concepts

- **Hypothesis** — A falsifiable statement: "Changing X will increase Y by Z% because W."
- **Variants** — Control (A) is always current production behaviour; each variant (B, C…) changes exactly one variable.
- **Primary metric** — Single north-star outcome (e.g., setup completion rate, time-to-gateway-ready).
- **Guardrail metrics** — Metrics that must not degrade: gateway error rate, proxy 502/503 rate, session drop rate.
- **Instrumentation surface** — Log events emitted by `server.js` categories (`setup`, `gateway`, `proxy`, `tui`) are the primary observability layer; add structured fields to distinguish variant cohorts.
- **Sample size** — Calculate using power analysis (typically 80% power, α=0.05) before launching.
- **Exposure logging** — Log variant assignment at the earliest moment the user experiences the difference, not at conversion.

## Common Patterns

### Setup Wizard Copy / Step Order Test
- **Control**: current `setup.html` step sequence and labels.
- **Variant**: reordered steps or rewritten CTA text in `src/public/setup.html` and `src/public/setup-app.js`.
- **Primary metric**: fraction of `/setup/api/run` calls that return `ok: true` within a session.
- **Instrumentation**: add `variant` field to the existing `log.info("setup", "onboard complete")` line in `server.js`.

### Channel Selection Defaulting Test
- **Control**: no channel pre-selected.
- **Variant**: Telegram pre-selected as default on page load.
- **Primary metric**: channel configuration completion rate (presence of channel key in `openclaw.json` post-setup).
- **Guardrail**: overall setup error rate (HTTP 500s from `/setup/api/run`).

### Gateway Readiness Feedback Test
- **Control**: generic loading spinner (`loading.html`).
- **Variant**: progress steps shown while polling gateway health.
- **Primary metric**: user-perceived time-to-first-interaction (client-side timing logged via `/setup/api/debug` beacon).
- **Guardrail**: gateway startup duration (server-side, logged in `waitForGatewayReady`).

### Measurement Plan Template

| Field | Value |
|---|---|
| Hypothesis | [statement] |
| Surface | [file / route] |
| Control | [description] |
| Variant(s) | [description] |
| Primary metric | [name + calculation] |
| Guardrail metrics | [list] |
| Minimum detectable effect | [%] |
| Required sample size | [N per variant] |
| Estimated duration | [days] |
| Exposure log point | [server.js line / event name] |
| Analysis method | [chi-squared / t-test / Mann-Whitney] |
| Ship threshold | [p < 0.05 + guardrails green] |