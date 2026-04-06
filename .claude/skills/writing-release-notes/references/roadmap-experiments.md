# Roadmap & Experiments Release Notes

## When to use
Reference this when drafting notes for feature flags, opt-in behavior (`ENABLE_WEB_TUI`), experimental env vars, or changes gated behind non-default configuration.

## Patterns

**New opt-in feature:**
```
### Web TUI available behind `ENABLE_WEB_TUI=true` (experimental)
Exposes a browser-based terminal at `/tui`. Requires setup auth. Not recommended for production deployments with shared access — session isolation is limited.
```

**Env var promoted to stable:**
```
### `OPENCLAW_GATEWAY_TOKEN` is now the recommended token source
Previously documented as optional. Now the preferred way to ensure token stability across redeploys. Set via Railway Variables using `${{ secret() }}`.
```

**Behavior gated on env var removed:**
```
### `LEGACY_PROXY_MODE` env var removed
The fallback proxy implementation it controlled has been deleted. Remove the variable from Railway Variables — it is silently ignored in this release.
```

## Pitfalls
- Experimental features still need migration notes if they change behavior in a future release. Flag them with "(experimental)" and document the stabilization plan if known.
- Removing an env var is a breaking change for users who have it set — always call this out explicitly even if the var was undocumented.