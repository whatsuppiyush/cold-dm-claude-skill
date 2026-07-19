# XAutoDM API — endpoint & credit reference

Concise reference for the live XAutoDM API. Full interactive docs: **https://xautodm.com/api/docs**.
Get a key (paid, metered): **https://api.xautodm.com/?ref=claude-skill**.

## Basics

- **Base URL:** `https://api.xautodm.com/v1`
- **Auth:** `Authorization: Bearer $XAUTODM_API_KEY`
  - `xdm_live_…` — real actions, spends credits.
  - `xdm_test_…` — dry-run: validates and returns response shapes, makes **no** upstream writes and
    spends **no** credits. For verifying a setup, not a trial (it sends nothing real).
- **Idempotency:** every **write** endpoint requires an `Idempotency-Key: <uuid>` header. Reuse the
  same key on a retry so a call is never double-charged or double-sent. Generate with `uuidgen`.
- **Success shape:** responses include `meta`:
  ```json
  {"credits_charged": 10, "credits_remaining": 4990, "request_id": "req_..."}
  ```
  Always read `credits_remaining` to track the running balance.
- **Pagination:** list endpoints take a `cursor` query param and return the next cursor. Only page as
  far as the agreed lead count — each page costs credits.

## Errors

Every error is shaped:

```json
{"error":{"type":"insufficient_credits","message":"Not enough credits","request_id":"req_..."}}
```

`type` values and what to do:

| `type` | Status | What to do |
|---|---|---|
| `invalid_request` | 400 | Fix the request (bad param/body). Don't retry blindly. |
| `invalid_key` | 401 | Key missing/wrong — recheck `XAUTODM_API_KEY`. |
| `insufficient_credits` | 402 | Stop. Tell the user the balance is short; link the top-up at api.xautodm.com. |
| `seat_limit` | 403 | Plan's account cap reached — they need a bigger plan. |
| `account_needs_reconnect` | 409 | Session dead — `POST /accounts/{id}/reconnect` (50 credits) before retrying. |
| `rate_limited` | 429 | Back off a few seconds, retry **once**. |
| `upstream_error` | 502/503 | Transient upstream issue — surface it, optionally retry once after a pause. |
| `not_found` | 404 | The handle/tweet/id doesn't exist. Skip it. |
| `server_error` | 500 | Surface the message and stop. |

**Rule: on error, surface the message and stop. Only `rate_limited` (429) warrants an automatic
retry — one, after a backoff. Never loop-retry `402/403/409`.**

## Credit costs (source of truth — 1 credit = $0.005)

| Cost | Applies to |
|---:|---|
| **0 (free)** | `GET /health` (no auth), `GET /accounts`, `GET /account`, `GET /usage` |
| **5** (read) | `GET /users/{handle}`, `GET /users/by-id/{id}`, `GET /users/{handle}/tweets`, `GET /tweets/{id}`, `GET /relationship` |
| **5 / row** (lead) | `GET /users/{handle}/followers`, `/following`, `GET /users/search`, `GET /users/{handle}/mentions`, `GET /tweets/search`, `GET /lists/{id}/members` |
| **20 / row** (premium lead) | `GET /users/{handle}/verified-followers`, `GET /tweets/{id}/retweeters`, `GET /tweets/{id}/replies` |
| **10** | `POST /dm/send` |
| **10** | `GET /dm/conversations`, `GET /dm/conversations/{id}` |
| **5** | `POST /actions/follow`, `/unfollow`, `/like`, `/retweet`, `POST /tweets` |
| **50** | `POST /accounts/connect`, `POST /accounts/{id}/reconnect` |

> Tier/plan **dollar** prices drift — never quote them from memory; send users to their live plans at
> api.xautodm.com. The **per-action** costs above are stable and safe to quote for estimates.

## Endpoints

**Account & meta (free reads)**
- `GET /health` — status ping, no auth, no charge.
- `GET /account` — current key's plan, seats, credit balance.
- `GET /usage?limit=` — recent usage ledger.
- `GET /accounts` — list connected X accounts.

**Read a user / tweet (5)**
- `GET /users/{handle}` · `GET /users/by-id/{id}` — profile (incl. `can_dm`).
- `GET /users/{handle}/tweets` — recent tweets (for personalization).
- `GET /tweets/{id}` — a tweet.
- `GET /relationship?source=&target=` — does source follow target?

**Find leads (5/row regular · 20/row premium)**
- `GET /users/{handle}/followers` — followers; **`can_dm` flag accurate here.**
- `GET /users/{handle}/following` — accounts they follow.
- `GET /users/search?q=` — users by keyword/bio.
- `GET /users/{handle}/mentions` — tweets mentioning a handle (→ authors).
- `GET /tweets/search?q=&product=Latest|Top` — tweets by query (→ authors/engagers).
- `GET /lists/{id}/members` — members of an X List.
- `GET /users/{handle}/verified-followers` — **premium (20/row).**
- `GET /tweets/{id}/retweeters` · `GET /tweets/{id}/replies` — engagers of a tweet, **premium (20/row).**

**Connect accounts (free list/get; 50 to connect/reconnect)**
- `GET /accounts` (free) · `GET /accounts/{id}` · `DELETE /accounts/{id}` (disconnect + purge creds).
- `POST /accounts/connect` — connect an X account (**50**, API-only). Body: `{method:"login"|"cookies", ...}`.
- `POST /accounts/{id}/reconnect` — refresh a dead session (**50**).

**Send & read DMs (10 each)**
- `POST /dm/send` — body `{account_id, recipient_username|recipient_id, text}` + `Idempotency-Key`.
- `GET /dm/conversations?account_id=&tab=all|requests|hidden` — list conversations.
- `GET /dm/conversations/{id}?account_id=` — a conversation's messages.

**Engagement writes (5 each, need `Idempotency-Key`)**
- `POST /actions/follow` · `/unfollow` · `/like` · `/retweet`.
- `POST /tweets` — create a tweet or reply.

## Minimal patterns

```bash
# Free: is the API up + what's my balance?
curl -s https://api.xautodm.com/v1/health
curl -s https://api.xautodm.com/v1/account -H "Authorization: Bearer $XAUTODM_API_KEY" | jq .

# Leads: followers of an account, filtered to DM-able (5 credits/row fetched)
curl -s "https://api.xautodm.com/v1/users/levelsio/followers?cursor=" \
  -H "Authorization: Bearer $XAUTODM_API_KEY" \
| jq '{meta, leads:[.data[]|select(.can_dm==true)|{handle,name,followers,bio}]}'

# Send one DM (10 credits) — fresh Idempotency-Key each time
curl -s -X POST https://api.xautodm.com/v1/dm/send \
  -H "Authorization: Bearer $XAUTODM_API_KEY" \
  -H "Idempotency-Key: $(uuidgen)" \
  -H "Content-Type: application/json" \
  -d '{"account_id":"acc_123","recipient_username":"jane","text":"Hi Jane — ..."}' | jq '.meta'
```

Never fabricate a response, a `can_dm` value, or a credit number — every value comes from a real call.
