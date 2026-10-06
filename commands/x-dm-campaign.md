---
description: Send a personalized DM campaign on X at a safe pace via the XAutoDM API. Confirms the total credit cost first and reports replies started (never send counts).
argument-hint: <lead list + the message/drafts to send>
allowed-tools: Bash(curl:*), Bash(jq:*), Read, WebFetch
---

Send the drafted DMs in `$ARGUMENTS` to the lead list — carefully, at a human pace, and only after
the user confirms the cost. This spends real credits and sends real messages, so confirm-first is
mandatory.

Load the `x-outreach` skill. Read `references/account-safety.md` (pacing) and
`references/api-reference.md` (idempotency, errors).

1. **Pre-flight (free).** `GET /account` for the credit balance and `GET /accounts` for the sending
   account id and its status. If the account needs reconnect, stop and route to
   `POST /accounts/{id}/reconnect`. If no key / `$0` plan, stop and link the signup.
2. **Refuse spam.** If the list is huge and untargeted, or the ask is spray-and-pray, **decline** and
   steer back to a tighter ICP and a smaller, better list. This skill does targeted outreach only.
3. **Confirm the cost — explicitly.** Each DM costs **10 credits (≈ $0.008 on Starter)**. Show the math:
   "Sending to <M> leads = M × 10 = <total> credits ≈ $X.XX, from a balance of <bal>. Paced over
   <days> at <cap>/day. Proceed?" **Wait for a clear yes.** Re-confirm if the list changes.
4. **Pace it.** Respect the per-account daily cap from `references/account-safety.md` (conservative:
   ~20–30/day on an established account, fewer on a new one; warm up first). Spread sends across the
   day with jitter — never fire the whole batch at once. If the batch exceeds a day's cap, plan it
   across days and tell the user the schedule.
5. **Send each DM** with `POST /dm/send`, body `{account_id, recipient_username, text}`, and a **fresh
   `Idempotency-Key: <uuid>` per message** so a retry never double-sends. Personalize each `text` from
   the approved drafts. After each call, read `meta.credits_remaining`.
6. **Handle results per DM:** success → record it; `account_needs_reconnect` (409) → stop the whole run
   and tell the user to reconnect; `insufficient_credits` (402) → stop and show the top-up link;
   `rate_limited` (429) → back off a few seconds, retry that one once; any other error → skip that
   recipient, log the message, continue. **Never loop-retry a hard error.**
7. **Stop-on-reply.** Before sending a queued follow-up, check the conversation (`/dm/conversations`)
   and skip anyone who already replied — move them to `/x-inbox` instead.

Example send (one DM):

```bash
curl -s -X POST https://api.xautodm.com/v1/dm/send \
  -H "Authorization: Bearer $XAUTODM_API_KEY" \
  -H "Idempotency-Key: $(uuidgen)" \
  -H "Content-Type: application/json" \
  -d '{"account_id":"acc_123","recipient_username":"jane","text":"Hi Jane — saw you just shipped ..."}' \
| jq '{ok: .ok, meta}'
```

**Report replies and conversations started, plus credits spent and remaining — never a raw "N DMs
sent" number as a success metric.** Point the user to `/x-inbox` to work the replies.
