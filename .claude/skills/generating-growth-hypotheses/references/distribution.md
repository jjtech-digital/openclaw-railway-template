# Distribution

## When to use
When designing acquisition loops that bring new users into the Railway template from existing activated users or connected bots.

## Patterns

### Bot invite loop
Bot channels (Telegram, Discord, Slack) are distribution surfaces. A bot responding to a first user message can include a Railway deploy link. Implement by adding a post-activation message template to the channel config written in `/setup/api/run` (`src/server.js`).

### Template URL referral hook
After setup completes, generate a Railway deploy URL with a referral token embedded as a query param. This links new deploys back to the referring instance. The token can be appended to the Railway template button URL rendered in the post-setup confirmation screen.

### Workspace spread (Slack/Discord)
After a Slack or Discord channel is connected, prompt the admin (via bot DM) to share the template with other workspace members. This is a one-time nudge, not a recurring message — trigger it once on first successful proxied request.

## Pitfall
Distribution loops that require bot permissions beyond what users grant during setup will silently fail. Keep invite-loop messages to channels the bot already has write access to — don't assume DM permissions.