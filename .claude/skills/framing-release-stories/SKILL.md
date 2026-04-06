---
name: framing-release-stories
description: Builds launch narratives, assets, and rollout checklists for OpenClaw Railway Template releases
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Framing-release-stories Skill

Generates launch narratives, release assets, and rollout checklists for OpenClaw Railway Template releases. Pulls context from git history, CLAUDE.md, and source files to produce audience-ready copy and structured go-live checklists grounded in actual changes.

## Quick Start

1. Run `/framing-release-stories` with an optional version or tag (e.g., `/framing-release-stories v2.1.0`)
2. The skill reads recent commits, CHANGELOG entries, and key source files to surface what changed
3. Outputs: one-paragraph launch narrative, bullet-point feature highlights, and a deployment rollout checklist

## Key Concepts

**Launch narrative** — A 3–5 sentence story framing the release for a technical audience: what problem it solves, what changed, and why it matters for Railway deployments.

**Feature highlights** — Concrete, user-facing bullets drawn from git log and diff summaries. Avoids implementation jargon; maps changes to user outcomes (e.g., "stable device pairings across redeploys" not "token persistence refactor").

**Rollout checklist** — Ordered steps covering pre-deploy verification, Railway variable checks (`SETUP_PASSWORD`, `OPENCLAW_GATEWAY_TOKEN`), volume mount confirmation, health endpoint validation (`/healthz`), and post-deploy smoke tests (`/setup`, `/openclaw`, `/logs`).

**Audience tiers** — Release assets may target: (1) operators deploying the template on Railway, (2) contributors reviewing the PR, or (3) end users of the OpenClaw Control UI. Tailor tone accordingly.

## Common Patterns

**Single-feature release**
Focus the narrative on one capability. Lead with the user outcome, follow with the mechanism, close with the deployment note.

**Multi-feature release**
Group highlights by surface area: gateway/proxy changes, setup wizard changes, auth changes, Docker/infra changes. Keep each bullet to one line.

**Breaking-change release**
Open the narrative with the migration impact. Reference `docs/MIGRATION_FROM_MOLTBOT.md` pattern for structure. Add a "Before you deploy" warning block to the rollout checklist covering any required env var additions or volume re-initialisation steps.

**Rollout checklist template**
```
Pre-deploy
- [ ] `npm run lint` passes on src/server.js
- [ ] Docker build completes without errors
- [ ] SETUP_PASSWORD set in Railway Variables
- [ ] OPENCLAW_GATEWAY_TOKEN set (use Railway secret() generator)
- [ ] Volume mounted at /data

Deploy
- [ ] Trigger Railway redeploy
- [ ] Monitor build logs for Homebrew and OpenClaw install steps

Post-deploy smoke tests
- [ ] GET /healthz returns 200
- [ ] GET /setup returns Basic auth challenge
- [ ] Complete /setup wizard end-to-end
- [ ] Verify /openclaw Control UI loads
- [ ] Verify /logs streams output
- [ ] Confirm device pairings intact (token stable across redeploy)
```