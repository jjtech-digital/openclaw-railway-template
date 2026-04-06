---
name: engineering-referral-loops
description: Designs referral or partner loop mechanics for the OpenClaw Railway Template, including invite flows, token generation, and tracking hooks.
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Engineering-referral-loops Skill

Implements referral and partner loop mechanics within the OpenClaw Railway Template wrapper. This skill covers generating stable referral tokens, tracking invite attribution, wiring partner callbacks into the Express layer, and persisting referral state alongside existing gateway configuration in the Railway volume.

## Quick Start

1. Identify the entry point in `src/server.js` where new routes should be registered (after existing `/setup` and proxy middleware).
2. Generate a referral token using the same `crypto.randomBytes(32).toString("hex")` pattern already used for `OPENCLAW_GATEWAY_TOKEN`.
3. Persist referral state to `OPENCLAW_STATE_DIR` (e.g., `referrals.json`) with mode `0o600`, matching the pattern used for `gateway.token`.
4. Expose a lightweight `/referral` endpoint (Basic auth via `requireSetupAuth`) for invite link generation and partner callbacks.
5. Log all referral events with `log.info("referral", "message")` — never log tokens directly.

## Key Concepts

- **Stable tokens**: Referral codes must survive redeploys. Persist to the Railway volume (`/data`) and check env vars first, disk second, generate third — same priority chain as `OPENCLAW_GATEWAY_TOKEN`.
- **Auth layering**: Referral management endpoints sit behind `requireSetupAuth` (Basic auth). Partner webhook callbacks use a separate HMAC signature check, not the gateway bearer token.
- **State isolation**: Referral data lives in its own file under `STATE_DIR`, not mixed into `openclaw.json`, to avoid corrupting gateway config.
- **Token injection parity**: If referral context needs to flow through to the proxied gateway, inject via `proxy.on("proxyReq")` — not direct `req.headers` mutation — to cover WebSocket upgrades too.

## Common Patterns

**Generate and persist a referral code:**
```javascript
const referralCodePath = path.join(STATE_DIR, "referral.token");
let referralCode;
try {
  referralCode = fs.readFileSync(referralCodePath, "utf8").trim();
} catch {
  referralCode = crypto.randomBytes(16).toString("hex");
  fs.writeFileSync(referralCodePath, referralCode, { mode: 0o600 });
}
```

**Register an invite endpoint:**
```javascript
app.get("/referral/invite", requireSetupAuth, (req, res) => {
  const base = process.env.RAILWAY_PUBLIC_DOMAIN
    ? `https://${process.env.RAILWAY_PUBLIC_DOMAIN}`
    : `http://localhost:${PORT}`;
  res.json({ url: `${base}/referral/join?ref=${referralCode}` });
});
```

**Track a partner callback with HMAC verification:**
```javascript
app.post("/referral/callback", express.json(), (req, res) => {
  const sig = req.headers["x-partner-signature"] ?? "";
  const expected = crypto
    .createHmac("sha256", referralCode)
    .update(JSON.stringify(req.body))
    .digest("hex");
  if (!crypto.timingSafeEqual(Buffer.from(sig), Buffer.from(expected))) {
    return res.status(401).json({ ok: false, error: "invalid signature" });
  }
  log.info("referral", `partner callback: ${req.body.event}`);
  // persist attribution to referrals.json
  res.json({ ok: true });
});
```

**Append referral origin to proxied requests (if gateway needs it):**
```javascript
proxy.on("proxyReq", (proxyReq, req) => {
  if (req.query.ref) {
    proxyReq.setHeader("X-Referral-Code", redactSecrets(req.query.ref));
  }
});
```