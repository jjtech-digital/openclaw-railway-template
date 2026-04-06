# Content Copy

## When to use
When drafting the actual text of launch narratives, feature highlights, changelog entries, or upgrade guides. Use alongside git log output and CLAUDE.md context to ground copy in real changes.

## Patterns

**Launch narrative structure**
1. Problem sentence — what friction existed before this release
2. Change sentence — what the wrapper now does differently
3. Deployment note — what the operator needs to know before going live

Keep to 3–5 sentences. Technical audiences scan; they do not read.

**Feature highlight formula**
`[User-facing outcome] — [one-line mechanism]`
Example: "Device pairings survive redeploys — gateway token now persists to volume and is stable across Railway deployments."
Avoid passive voice and nominalisations ("improvement to token handling").

**Breaking-change warning block**
```
> **Before you deploy:** [describe the breaking change in one sentence].
> Required: [env var or volume action]. See docs/MIGRATION_FROM_MOLTBOT.md for full steps.
```
Place this block at the top of the rollout checklist, not buried in feature highlights.

## Pitfalls
Do not copy-paste commit messages directly into release copy. Commit messages are written for contributors; release copy is written for operators. Translate, do not transcribe.