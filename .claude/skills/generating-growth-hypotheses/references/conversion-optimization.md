# Conversion Optimization

## When to use
When analyzing drop-off between funnel stages: template deploy → `/setup` completion → gateway ready → first proxied request.

## Patterns

### Funnel stage mapping
Audit step transitions in `src/public/setup-app.js` to identify where users abandon the wizard. Each wizard step that requires external action (creating a bot, copying a token) is a potential drop-off point.

### Inline guidance injection
Add contextual help text directly into `src/public/setup.html` at high-friction steps (e.g., bot token fields for Telegram/Discord). Hypothesis card format:
```
Channel: web
Hypothesis: If we add inline token-fetch instructions at step 3, setup completion rate increases because users won't leave to find docs
Metric: setup completion rate
Test: Add collapsible "How to get this token" section in setup.html
Expected lift: 15% reduction in wizard abandonment
```

### Error message clarity
Check error response paths in the `/setup/api/run` handler (`src/server.js`). Vague errors ("setup failed") cause silent drop-off; specific errors ("Invalid bot token — check it has no extra spaces") enable self-recovery.

## Pitfall
Don't optimize later funnel steps before fixing the first one with measurable drop-off. Audit `setup-app.js` step transitions before writing new UI copy.