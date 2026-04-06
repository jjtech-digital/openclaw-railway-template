# In-App Guidance Release Notes

## When to use
Reference this when drafting notes for changes to setup wizard copy, inline help text, status messages, error feedback in the browser UI, or the loading/redirect pages.

## Patterns

**Error message improved:**
```
### Setup wizard now shows actionable error on gateway timeout
Instead of a blank screen, users see a specific message with next steps when the gateway fails to start within 60 seconds.
```

**Status indicator added:**
```
### Gateway readiness indicator on `/setup`
The wizard now polls and displays live gateway status before redirecting to `/openclaw`, reducing confusion during slow cold starts.
```

**Redirect behavior changed:**
```
### Unconfigured deployments now redirect to `/setup` immediately
All non-setup routes redirect on first request instead of returning a 502. Affects users who bookmark `/openclaw` before completing setup.
```

## Pitfalls
- Loading and redirect pages (`loading.html`) are shown during gateway startup — changes here affect the perceived startup time even if backend timing is unchanged. Note both the UI change and any timing dependency.
- Do not reference internal route names (`/setup/api/run`) in user-facing notes.