# Conversion Optimization

## When to use
When improving lifecycle message sequences to increase the rate of users completing key activation steps: visiting `/setup`, finishing the wizard, and sending a first message via a connected channel.

## Patterns

**Reduce friction at the wizard gate**
The `/setup` wizard is the single highest-drop point. Welcome messages should set a concrete time expectation ("~2 minutes") and link directly to the URL. Avoid vague CTAs like "get started" — use "Open your setup wizard at `your-app.up.railway.app/setup`."

**Anchor CTAs to observable outcomes**
Every message CTA should point to something the user can verify: gateway health via `/logs`, a test message in their Telegram/Discord/Slack channel, or the `/openclaw` UI responding. Abstract success ("you're all set") converts worse than concrete confirmation ("send `/ping` to your bot — it should reply within 5 seconds").

**Sequence timing around gateway warm-up**
Gateway startup takes 30–90 seconds after wizard completion. A "hang tight" message sent immediately after setup prevents support contacts caused by users assuming failure. Follow up with a "you're live" trigger once `/healthz` reports ready.

## Pitfalls
Do not ask users to take action before the gateway is ready. Sending a "try it now" CTA during the gateway startup window causes confusion and erodes trust. Gate that message on the health check passing.