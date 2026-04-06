# Measurement & Testing

## When to use
When defining success metrics for lifecycle message sequences or A/B testing copy and timing changes.

## Patterns

**Activation funnel metrics**
Track conversion at each lifecycle gate:
1. Deploy → `/setup` visited (healthz hit + setup page load)
2. Setup visited → wizard completed (`openclaw.json` written)
3. Wizard completed → gateway ready (health check passes)
4. Gateway ready → first proxied request to `/openclaw`
5. First request → channel message received (channel-specific)

**Log-based event proxies**
The wrapper's ring buffer (last 1000 log lines, `/logs` endpoint) provides observable signals: `[gateway] ready`, `[gateway] starting`, `[onboard] complete`. These can serve as event proxies for funnel stage transitions when a formal analytics pipeline is absent.

**Copy test structure**
When testing subject lines or CTAs, change one variable per variant. The lowest-effort test: send two wizard-completion message variants to alternating deployments and measure time-to-first-proxied-request as the outcome metric.

## Pitfalls
Gateway startup time (30–90s) creates noise in time-based funnel analysis. A user who completes the wizard and loads `/openclaw` 45 seconds later is not churning — they are waiting for the gateway. Segment funnel data by gateway-ready state, not by clock time from wizard submission.