---
description: Warm up prospects before DMing — follow, like, and reply to a few of their posts via the XAutoDM API. Confirms cost first; paced and human.
argument-hint: <handles or a lead list to warm up>
allowed-tools: Bash(curl:*), Bash(jq:*), Read, WebFetch
---

Warm up the prospects in `$ARGUMENTS` before the cold DM so the DM lands from a familiar name, not a
stranger. Every action here spends credits, so confirm first and keep it small and genuine.

Load the `x-outreach` skill. Read `references/account-safety.md` (warm-up-first, pacing) and
`references/api-reference.md`.

1. **Pre-flight (free).** `GET /accounts` for the sending account id + status; confirm key + credits.
2. **Why warm up.** Following and engaging with 1–2 real posts a day or two before a DM lifts reply
   rates and looks human. It is optional and best reserved for higher-value prospects — it costs
   **5 credits per action** and adds account activity, so don't over-do it.
3. **Plan the touches per prospect** (keep it light and real):
   - **Follow** → `POST /actions/follow` (5 credits).
   - **Like** a recent, relevant post → `POST /actions/like` (5 credits). Pick a post you'd genuinely
     endorse, not the newest thing blindly.
   - **Reply** with a short, substantive comment → `POST /tweets` as a reply (5 credits). Only when you
     have something real to add — a throwaway "great post!" hurts more than it helps.
   Typical warm-up = 1 follow + 1 like (± 1 reply) per prospect. That's ~10–15 credits each.
4. **Estimate and confirm.** "Warming up <N> prospects with <plan> ≈ <total> credits ≈ $X.XX. Proceed?"
   Wait for a yes.
5. **Execute, paced.** One prospect at a time, with a **fresh `Idempotency-Key` per write**, spread out
   with jitter — never a rapid-fire burst of follows/likes (that's the fastest way to get an account
   flagged; see `references/account-safety.md`). Respect the account's daily action budget.
6. **Then wait, then DM.** Warm-up should precede the DM by a day or two — don't follow and DM in the
   same minute. Hand off to `/x-dm-campaign` after a natural gap.

Example (follow):

```bash
curl -s -X POST https://api.xautodm.com/v1/actions/follow \
  -H "Authorization: Bearer $XAUTODM_API_KEY" \
  -H "Idempotency-Key: $(uuidgen)" \
  -H "Content-Type: application/json" \
  -d '{"account_id":"acc_123","username":"jane"}' | jq '{ok, meta}'
```

On errors, surface and stop (retry once only on `429`). Report actions taken, credits spent, and
remaining. Keep engagement authentic — refuse to mass-follow/mass-like for vanity; that's spam and it
gets accounts banned.
