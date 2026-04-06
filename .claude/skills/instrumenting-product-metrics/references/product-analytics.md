# Product Analytics Reference

## When to use
Use this reference when adding, modifying, or querying any structured metric event in the OpenClaw Railway Template. All analytics are log-based — no external SDK is introduced.

## Patterns

**Standard event schema**
```js
log.info("metrics", JSON.stringify({
  event: "category.action",     // dot-namespaced, snake_case
  durationMs: number,           // include for any timed operation
  source: "env" | "disk" | "generated",  // include for token/config origin
  // ...additional context props
}));
```

**Token source instrumentation** (pairing stability signal)
```js
// src/server.js — after gateway token is resolved
log.info("metrics", JSON.stringify({
  event: "gateway_token.resolved",
  source: tokenSource,  // "env" | "disk" | "generated"
}));
```

**Querying the ring buffer**
```bash
# All distinct event types and counts
curl -u admin:$SETUP_PASSWORD http://localhost:8080/logs \
  | grep '"metrics"' \
  | jq -r '.event' \
  | sort | uniq -c | sort -rn
```

## Pitfalls
- The ring buffer holds the last 1 000 log lines across **all** categories. High-frequency proxy traffic logs can evict metric events before they are queried. Keep `proxy.*` events off by default; emit them only under `OPENCLAW_TEMPLATE_DEBUG=true`.
- Never log `OPENCLAW_GATEWAY_TOKEN` or `SETUP_PASSWORD` inside a metrics payload. Run all metric objects through `redactSecrets()` before emitting if they include request headers or config blobs.