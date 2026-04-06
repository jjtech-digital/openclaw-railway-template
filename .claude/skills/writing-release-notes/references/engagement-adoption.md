# Engagement & Adoption Release Notes

## When to use
Reference this when drafting notes for changes that affect repeat usage: the `/logs` viewer, `/tui` terminal, debug console commands, or config editor improvements.

## Patterns

**New debug console command:**
```
### New debug command: `openclaw.<command>`
Available in `/setup` → Tools → Debug Console. Use it to [action] without restarting the gateway.
```

**TUI session behavior changed:**
```
### TUI sessions now time out after [N] minutes of inactivity
Configurable via `TUI_IDLE_TIMEOUT_MS`. Previously sessions persisted indefinitely, consuming resources on idle deployments.
```

**Logs viewer improvement:**
```
### Live logs now buffer the last 1000 lines
The `/logs` endpoint serves from an in-memory ring buffer — no disk I/O required. Older entries are discarded automatically.
```

## Pitfalls
- Avoid framing TUI or logs improvements as "developer tools" — Railway deployers use these for day-to-day monitoring, not development.
- Session timeout changes affect users mid-session. Include the new default and the env var override in the note.