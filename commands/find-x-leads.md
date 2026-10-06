---
description: Find DM-able X leads — search users/tweets, pull followers of seed accounts, filter to can_dm, dedupe into a lead list. Shows the credit cost before spending.
argument-hint: <query, e.g. "followers of @levelsio who can DM">
allowed-tools: Bash(curl:*), Bash(jq:*), Read, WebFetch
---

Build a deduplicated list of DM-able leads for `$ARGUMENTS`. This spends credits, so estimate first
and get an explicit yes before any call that charges.

Load the `x-outreach` skill. Read `references/api-reference.md`.

1. **Pre-flight (free).** Confirm `XAUTODM_API_KEY` is set and there's a plan with credits (run
   `/x-setup` if unsure). No key or `$0` plan → stop and route to
   https://api.xautodm.com/?ref=claude-skill.
2. **Pick the source** from the request / ICP and map it to an endpoint + per-row cost:
   - Followers of a seed account → `GET /users/{handle}/followers` — **5/row**
   - People X follows → `GET /users/{handle}/following` — **5/row**
   - Keyword/bio user search → `GET /users/search?q=` — **5/row**
   - Recent posts about a topic → `GET /tweets/search?q=&product=Latest` — **5/row** (then keep authors)
   - Members of an X List → `GET /lists/{id}/members` — **5/row**
   - Verified followers → `GET /users/{handle}/verified-followers` — **10/row** (premium)
   - Repliers / retweeters of a tweet → `GET /tweets/{id}/replies` or `/retweeters` — **10/row** (premium)
3. **Estimate and confirm.** State plainly: "Pulling up to N rows from <source> ≈ N × <cost> credits
   ≈ $X.XX. Proceed?" Wait for a yes. Prefer regular (5) sources unless the ICP truly needs premium.
   Note that you're charged per profile **fetched**, not per profile that survives the `can_dm` filter,
   so pull in reasonable page sizes and stop when you hit the target count.
4. **Fetch + paginate.** Call the endpoint, follow the `cursor` for more pages only up to the agreed
   count. After each call read `meta.credits_remaining` and keep a running total; if it approaches the
   balance, stop and report.
5. **Filter.** Keep only `can_dm: true`. Apply the ICP's follower band, bio keywords, and anti-signals.
   Drop obvious bots/giveaway accounts.
6. **Dedupe.** Remove duplicates by handle/id, and remove anyone already contacted or on the user's DNC
   list if that context exists.
7. **Deliver** a clean lead table: handle · name · followers · bio snippet · `can_dm` · id. Report how
   many rows were fetched, how many survived filtering, total credits spent, and credits remaining.

Example (followers, filtered to DM-able):

```bash
curl -s "https://api.xautodm.com/v1/users/levelsio/followers?cursor=" \
  -H "Authorization: Bearer $XAUTODM_API_KEY" \
| jq '{meta, leads: [.data[] | select(.can_dm==true) | {handle, name, followers, bio}]}'
```

On `insufficient_credits` (402) stop and show the top-up link; on `rate_limited` (429) back off once;
on any other error, surface the message and stop. Never fabricate leads or a `can_dm` value — every
row comes from a real response. Next step: `/draft-x-dms` (free) on this list.
