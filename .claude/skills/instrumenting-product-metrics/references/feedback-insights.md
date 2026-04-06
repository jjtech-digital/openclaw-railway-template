# Feedback & Insights Reference

## When to use
Capture explicit user signals (debug console usage, reset actions, error patterns) and surface them as product insights to prioritise fixes and roadmap items.

## Patterns

**Setup reset signal**
```js
// src/server.js — inside the /setup/api/reset handler
log.info("metrics", JSON.stringify({
  event: "setup.reset",
  hadGatewayConfig: !!existingConfig,
  hadChannels: channelCount > 0,
}));
```

**Debug console command usage**
```js
// src/server.js — after each allowed console command runs
log.info("metrics", JSON.stringify({
  event: "debug_console.command_run",
  command: sanitizedCommand,   // never log args that may contain secrets
  exitCode: result.code,
}));
```

**Gateway readiness failure**
```js
// src/server.js — when waitForGatewayReady exhausts retries
log.info("metrics", JSON.stringify({
  event: "gateway.ready_timeout",
  elapsedMs: Date.now() - gatewayStartTime,
  lastEndpointTried: lastEndpoint,
}));
```

## Pitfalls
- `setup.reset` is a strong negative signal — a user resetting after completing onboard indicates a broken state, not exploration. Distinguish `reset_before_gateway` from `reset_after_gateway_ready` using the `hadGatewayConfig` flag to separate intent from frustration.
- Debug console command arguments may contain channel tokens or config values. Always log the command name only (`command: "openclaw.config.get"`) and never include `args` in the metric payload — pass args through `redactSecrets()` if the full invocation must be recorded.