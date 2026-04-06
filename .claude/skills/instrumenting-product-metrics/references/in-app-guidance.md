# In-App Guidance Metrics

## When to use
Measure whether setup wizard tooltips, inline help text, and contextual prompts are seen and acted upon — key signals for reducing time-to-activation.

## Patterns

**Guidance step viewed (client → server)**
```js
// src/public/setup-app.js — when a help tooltip or inline hint is shown
await fetch("/setup/api/metrics", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    event: "guidance.step_viewed",
    step: "channel_config",   // e.g. "auth_provider" | "channel_config" | "token_stable"
    wizardPage: currentPage,
  }),
});
```

**Guidance dismissed vs. acted upon**
```js
// Track whether users follow the hint or skip it
await fetch("/setup/api/metrics", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    event: "guidance.cta_clicked",
    step: "discord_intent_reminder",
    outcome: "opened_developer_portal",  // or "dismissed"
  }),
});
```

**Server-side aggregation query**
```bash
curl -u admin:$SETUP_PASSWORD http://localhost:8080/logs \
  | grep '"event":"guidance\.' \
  | jq -r '[.step, .outcome] | @tsv' \
  | sort | uniq -c
```

## Pitfalls
- Guidance metrics fire in the browser; if the user navigates away before the `fetch` completes, events are lost. Use `navigator.sendBeacon` for dismissal/close events where reliability matters.
- Do not instrument every tooltip render — focus on steps where drop-off is known (e.g., Discord MESSAGE CONTENT INTENT, Railway volume mount). Over-instrumentation saturates the 1 000-line ring buffer with low-signal noise.