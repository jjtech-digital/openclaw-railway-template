# Distribution

## When to use
When deciding how and where to deliver lifecycle messages: in-app UI, email, Railway deployment notifications, or bot-channel messages.

## Patterns

**In-app is the primary channel**
Because users interact with `/setup`, `/logs`, and `/openclaw` via browser, in-page messaging (setup wizard steps, status banners, loading states) reaches users at the highest-intent moment. Prioritize in-app copy before adding out-of-band email.

**Bot channel as a confirmation channel**
Once a channel (Telegram/Discord/Slack) is connected, the bot itself can deliver a first message confirming setup success. This doubles as a live proof-of-function. Keep this message short: confirm the channel is active, invite the first real query.

**Railway deployment hooks for redeploy messaging**
Redeploy events are detectable via version bumps in logs. A lightweight in-app banner on first load after redeploy ("Config and pairings restored from volume — no action needed") prevents unnecessary re-setup attempts. This can be surfaced via `/setup/api/debug` response data on page load.

## Pitfalls
Do not rely on email as the sole delivery path. Many Railway template users deploy for personal or team use without configuring transactional email. If a message is critical (e.g., setup required), it must be surfaced in-app at the `/setup` redirect, not only via email.