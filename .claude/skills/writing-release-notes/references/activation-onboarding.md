# Activation & Onboarding Release Notes

## When to use
Reference this when drafting notes for changes to the `/setup` wizard, onboarding flow, `openclaw onboard --non-interactive`, or initial channel configuration writes.

## Patterns

**New setup step added:**
```
### [Step name] added to setup wizard
Users now configure [X] during initial setup at `/setup`. Previously this required manual edits to `openclaw.json`.
```

**Onboarding failure mode fixed:**
```
### Setup wizard no longer stalls on [auth provider] auth
Fixed a race condition in the onboard handshake that caused `/setup` to hang indefinitely when [condition]. Users must re-run setup if previously affected.
```

**Channel config write changed:**
```
### Channel configuration now written via `config set --json`
Replaces the unreliable `channels add` CLI call. Affects Telegram, Discord, and Slack setup paths.
```

## Pitfalls
- Do not describe CLI flags (`--non-interactive`, `--auth-provider`) in user-facing notes — frame changes in terms of what the user sees in the browser, not what the wrapper calls internally.
- If `openclaw.json` schema changes, always include a migration note — existing installs will break silently on redeploy.