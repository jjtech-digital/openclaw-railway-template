# Conversion Optimization

## When to use
When release copy needs to move operators from "aware of the release" to "deployed on Railway." Apply when writing rollout checklists, upgrade prompts, or CTA copy in release notes.

## Patterns

**Reduce friction at the deploy step**
Lead the rollout checklist with the one action that blocks everything else (volume mount, `SETUP_PASSWORD` set). Operators who hit an avoidable error mid-deploy rarely retry immediately.

**Anchor on outcome, not version number**
"Stable device pairings across redeploys" converts better than "v2.1.0 token persistence." Use the outcome as the headline; put the version in a subhead or badge.

**One clear next action per section**
Each checklist section (Pre-deploy / Deploy / Post-deploy) ends with a single verifiable state (e.g., `GET /healthz returns 200`), not an open-ended task. Checkboxes with concrete pass/fail criteria reduce decision fatigue.

## Pitfalls
Avoid listing every changed file or function in operator-facing copy. Implementation detail in release notes raises anxiety ("what broke?") rather than confidence. Stick to user-facing outcomes and migration steps only.