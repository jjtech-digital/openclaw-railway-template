# Measurement & Testing

## When to use
When defining metrics, baselines, and test conditions for growth hypotheses before building anything.

## Patterns

### Core activation event
The single activation metric is the **first successful proxied request** through the gateway. All funnel metrics lead here. Instrument by logging the first non-`/setup`, non-`/healthz` request that returns a non-502 from the proxy handler in `src/server.js`.

### Baseline sourcing from existing code
| Signal | Source |
|--------|--------|
| Setup drop-off | Step transition logic in `src/public/setup-app.js` |
| Channel config errors | Error paths in `/setup/api/run` handler |
| Gateway readiness latency | Polling loop in `waitForGatewayReady` |
| Proxy error rate | `proxy.on("error")` handler log output |

### Hypothesis card test conditions
Every experiment needs a falsifiable condition. Use this structure:
```
Baseline: current completion rate (from logs or manual audit)
Success threshold: X% improvement over 2-week window
Rollback: revert setup.html/setup-app.js change if metric regresses
```

## Pitfall
Gateway logs are in-memory (ring buffer, last 1000 lines). Don't rely on them as a persistent metric store — export key events to an external sink if you need longitudinal data for A/B comparisons.