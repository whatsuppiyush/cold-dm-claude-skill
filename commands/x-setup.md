---
description: Validate the XAutoDM API key and show plan, credit balance, and connected X accounts — the gate before any outreach
argument-hint: (no arguments)
allowed-tools: Bash(curl:*), Bash(jq:*), Read, WebFetch
---

Set up and verify XAutoDM before any outreach happens. This is the gate — nothing spends credits
until it passes.

Load the `x-outreach` skill. Read `references/api-reference.md` for auth and error shapes.

1. **Reachability (free).** `GET https://api.xautodm.com/v1/health` — no auth, no charge. If it
   fails, the API is unreachable; stop and report.
2. **Key present?** Check that `XAUTODM_API_KEY` is set in the environment. If it is missing:
   - Explain, honestly, that XAutoDM is a paid, metered API (no free trial) and that the user needs
     their own key. Point them to **https://api.xautodm.com/?ref=claude-skill** to sign up.
   - Note the two key types: `xdm_live_…` performs real actions and spends credits; `xdm_test_…` is
     a dry-run that validates the setup and spends nothing (good for a first check, but it sends
     nothing real).
   - Stop here — do not attempt any authed call.
3. **Account + plan (free).** `GET /account` with `Authorization: Bearer $XAUTODM_API_KEY`. Report
   the plan name, **credit balance**, and seat usage. If the response is `invalid_key`, tell them to
   recheck the key. If the plan is the default **$0 / 0-credit** plan, explain they must subscribe to
   a plan (and connect an X account) before anything can run, and link the signup.
4. **Connected accounts (free).** `GET /accounts`. List each connected X account (handle, id,
   status). If none are connected, explain that they must connect one before sending: it is
   **API-only** via `POST /accounts/connect` and costs **50 credits** — confirm before doing it.
   If any account shows a needs-reconnect status, flag it and point to `POST /accounts/{id}/reconnect`.
5. **Usage (optional, free).** `GET /usage` for a recent spend ledger if the user wants to see where
   credits went.

Present a short readiness summary: key OK? · plan + credits · connected account(s) · what's blocking,
if anything. End with the next step (`/x-icp` if they're ready to target, or the signup link if not).

Example curl (never print the key back to the user):

```bash
curl -s https://api.xautodm.com/v1/account \
  -H "Authorization: Bearer $XAUTODM_API_KEY" | jq '{plan, credits_remaining, seats}'
```

Do not fabricate a balance or an account — every number here comes from a real call you made.
