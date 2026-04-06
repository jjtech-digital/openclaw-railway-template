# Measurement & Testing

## When to use
Apply when defining instrumentation, choosing a statistical test, or calculating required sample size before launching an experiment.

## Patterns

### Instrumentation via Log Categories
Add a `variant` field to existing log calls rather than new log lines:
```js
log.info("setup", "onboard complete", { variant: req.cookies["openclaw-variant"] });
```
Query the ring buffer at `/logs` or parse stdout to compute per-variant event counts.

### Statistical Test Selection
- **Binary outcomes** (completed / did not complete): chi-squared test.
- **Continuous outcomes** (startup duration, session length): Mann-Whitney U (non-parametric; avoids assuming normal distribution in small samples).
- **Multiple variants**: apply Bonferroni correction to α before declaring significance.

### Sample Size Estimation
Minimum inputs: baseline rate, minimum detectable effect (MDE), power (0.80), α (0.05). Use a standard power calculator. For setup completion rates typically around 60%, detecting a 10 pp lift requires ~200 sessions per variant.

## Pitfalls
- The ring buffer holds only the last 1000 log lines. For experiments running longer than a few hours, pipe logs to an external sink or increase buffer size before launch.
- Novelty effect inflates early variant performance. Run experiments for at least one full week before reading results.