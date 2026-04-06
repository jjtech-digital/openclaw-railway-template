# Content & Copy

## When to use
When writing UI text, error messages, empty states, or onboarding prompts inside `src/public/` files.

## Patterns

### Action-first labels
Wizard CTAs and button labels should describe the outcome, not the action. Prefer "Connect Telegram Bot" over "Next" at channel-config steps in `setup.html`.

### Error copy as recovery instructions
Error strings returned from `/setup/api/run` and displayed in the wizard UI should tell the user what to do next, not just what went wrong:
- Bad: `"Gateway failed to start"`
- Good: `"Gateway didn't start — check your token in Step 2 or view logs at /logs"`

### Gateway-ready confirmation copy
The message shown after gateway startup (`loading.html` or redirect) sets expectations for what "success" looks like. Include a concrete next step: "Your OpenClaw gateway is live. Open /openclaw to run your first task."

## Pitfall
Copy changes to `setup.html` are static — they take effect on next deploy. Don't use server-side rendered strings for wizard UI text unless the content is dynamic (e.g., user-specific URLs). Keep static copy in HTML, dynamic values in `setup-app.js`.