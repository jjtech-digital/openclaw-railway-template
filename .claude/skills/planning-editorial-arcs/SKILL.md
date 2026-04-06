---
name: planning-editorial-arcs
description: Defines content themes, briefs, and editorial cadence for the OpenClaw Railway Template project
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Planning-editorial-arcs Skill

This skill helps define content themes, structured briefs, and publishing cadence for documentation, release notes, onboarding copy, and user-facing messaging within the OpenClaw Railway Template project. It operates across `src/public/`, `docs/`, and root-level markdown files to ensure consistent voice and timely content delivery aligned with feature releases.

## Quick Start

1. Identify the content surface: setup wizard copy (`src/public/setup.html`, `src/public/setup-app.js`), docs (`docs/`), or release-facing markdown (`README.md`, `CONTRIBUTING.md`).
2. Define the target audience segment: new deployers, returning users upgrading from MoltBot, or developers contributing to the template.
3. Draft a brief with theme, goal, key messages, and publish timing relative to the release cycle.
4. Review existing copy for tone consistency before authoring new content.

## Key Concepts

- **Editorial theme**: A focused message tied to a feature area (e.g., "zero-SSH onboarding", "stable token persistence across redeploys", "browser-first setup").
- **Content brief**: A structured outline specifying audience, goal, key messages, call to action, and target surface (UI copy, docs page, release note).
- **Cadence**: Content is tied to the release cycle — new docs and UI copy ship with the feature; release notes publish at tag time; onboarding copy is reviewed on every setup wizard change.
- **Voice**: Direct, technical, low-friction. Matches the minimal setup philosophy of the template.

## Common Patterns

- **Feature launch brief**: When a new capability ships (e.g., device pairing, TUI endpoint), draft a brief covering what changed, why it matters to the deployer, and which surfaces need updated copy (`setup.html`, `docs/`, `README.md`).
- **Onboarding arc**: Map the user journey from first deploy → `/setup` wizard → gateway ready → first agent message. Identify copy gaps at each stage and assign briefs per step.
- **Cadence table**:

  | Content Type        | Trigger                        | Surface                        |
  |---------------------|-------------------------------|-------------------------------|
  | UI microcopy        | Setup wizard change            | `src/public/setup.html`       |
  | Release note        | Git tag / version bump         | `docs/` or `CHANGELOG`        |
  | Migration guide     | Breaking change or rename      | `docs/MIGRATION_FROM_*.md`    |
  | Startup improvement | Gateway lifecycle change       | `docs/STARTUP-IMPROVEMENTS.md`|

- **Audience segmentation**: Tailor briefs for (a) first-time Railway deployers unfamiliar with OpenClaw, (b) MoltBot migrators (reference `docs/MIGRATION_FROM_MOLTBOT.md`), and (c) contributors reading `CONTRIBUTING.md`.