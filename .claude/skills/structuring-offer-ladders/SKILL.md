---
name: structuring-offer-ladders
description: Frames plan tiers, value ladders, and upgrade logic for the OpenClaw Railway Template
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Structuring-offer-ladders Skill

This skill guides the design and implementation of plan tiers, value ladders, and upgrade logic within the OpenClaw Railway Template. It helps frame how deployment tiers (e.g., free, pro, team) map to gateway features, channel integrations, and access controls exposed through the `/setup` wizard and server configuration.

## Quick Start

1. Identify which features gate on tier: channel count, TUI access (`ENABLE_WEB_TUI`), pairing, log retention.
2. Map each tier to environment variables and `openclaw.json` config keys that enforce limits.
3. Surface upgrade prompts in `setup.html` / `setup-app.js` at the point of friction (e.g., when a user tries to add a fourth channel).
4. Implement server-side enforcement in `src/server.js` route handlers before proxying to the gateway.

## Key Concepts

- **Value ladder**: Each tier unlocks a capability the layer below cannot access — free gets one channel, pro gets multi-channel + TUI, team gets pairing + shared tokens.
- **Gate point**: The earliest server-side check that rejects or redirects an under-tier request. Prefer `src/server.js` middleware over client-only checks.
- **Upgrade trigger**: UI moment where a user hits a limit and is shown a path forward. Implemented in `setup-app.js` as a conditional render after a failed API call.
- **Config as source of truth**: Tier entitlements are reflected in `openclaw.json` via `openclaw config set --json`. The wrapper reads these at startup and enforces them throughout the session.

## Common Patterns

**Tier check in middleware**
```javascript
function requireTier(minTier) {
  return (req, res, next) => {
    const tier = resolveCurrentTier(); // read from openclaw.json or env
    if (tierRank(tier) < tierRank(minTier)) {
      return res.status(402).json({ ok: false, error: "upgrade_required", minTier });
    }
    next();
  };
}
app.use("/tui", requireSetupAuth, requireTier("pro"), tuiHandler);
```

**Upgrade prompt in setup-app.js**
```javascript
if (response.status === 402) {
  showUpgradeBanner(data.minTier); // render inline CTA, not a redirect
  return;
}
```

**Persisting tier to config**
```javascript
await runCmd(OPENCLAW_NODE, clawArgs([
  "config", "set", "--json",
  JSON.stringify({ "plan.tier": selectedTier })
]));
```

**Ladder definition (add near top of server.js)**
```javascript
const TIER_RANK = { free: 0, pro: 1, team: 2 };
const tierRank = (t) => TIER_RANK[t] ?? -1;
const resolveCurrentTier = () => loadedConfig?.plan?.tier ?? "free";
```