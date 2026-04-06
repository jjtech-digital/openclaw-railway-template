# Distribution

## When to use
When deciding where release assets go and in what form — Railway template marketplace listing, GitHub release notes, PR description, or CHANGELOG entry. Each surface has different length and format constraints.

## Patterns

**Surface-to-format mapping**

| Surface | Max length | Format |
|---|---|---|
| GitHub Release title | ~72 chars | Plain text, outcome-first |
| PR description | No limit | Narrative + checklist |
| CHANGELOG entry | 3–5 bullets | Past tense, grouped by area |
| Railway template description | ~300 chars | One paragraph, operator-focused |

**Layered distribution**
Write one canonical narrative (PR/release), then derive shorter variants for CHANGELOG and Railway listing. Do not write each surface independently — they drift and create inconsistency.

**Audience routing for breaking changes**
- Contributors: full diff context in PR description
- Operators: migration steps in GitHub Release notes with "Before you deploy" block
- End users: nothing (Control UI changes are transparent; setup wizard handles auth)

## Pitfalls
Railway template marketplace descriptions index on keywords. Avoid abstract copy ("improved reliability"). Use concrete nouns: "Railway volume", "bearer token", "setup wizard", "/healthz endpoint". These surface in operator searches.