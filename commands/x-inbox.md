---
description: Read the X DM inbox — list conversations and replies, then draft responses to warm leads. Reads cost credits; drafting replies is free.
argument-hint: (optional) <handle or "requests" tab>
allowed-tools: Bash(curl:*), Bash(jq:*), Read, WebFetch
---

Work the reply inbox: see who responded, and draft the next message for each warm lead. Reading
conversations spends credits (10 each); drafting the replies is free.

Load the `x-outreach` skill. Read `references/dm-playbook.md` (handling replies) and
`references/api-reference.md`.

1. **Pre-flight (free).** `GET /accounts` for the account id. Confirm a key + credits (`/x-setup` if
   unsure).
2. **List conversations.** `GET /dm/conversations?account_id=<id>&tab=all` (or `tab=requests` for
   message requests / `tab=hidden`). **This costs 10 credits.** Say so before calling if the user is
   cost-sensitive. `$ARGUMENTS` can scope it to a tab or a specific handle.
3. **Read the ones that matter.** For a conversation you need the full thread of, `GET
   /dm/conversations/{id}?account_id=<id>` (**10 credits each**) — only open the ones with a real
   reply worth acting on, not every thread, to keep spend down.
4. **Triage.** Sort replies into: **interested** (drive toward the call / next step), **question**
   (answer it, then a soft ask), **not interested / negative** (thank them, stop — add to DNC), and
   **auto-reply/noise** (ignore). Anyone who replied should be **removed from any active DM sequence**.
5. **Draft responses (free).** For each warm reply, write a short, specific next message per the
   playbook — answer what they actually said, keep momentum, and make the ask concrete (a specific
   time, a link, a yes/no question). No pushy multi-paragraph pitches.
6. **Sending a reply** is a DM like any other: `POST /dm/send` with `{account_id, recipient_username,
   text}` + a fresh `Idempotency-Key` (**10 credits**). Confirm before sending. You can hand the drafts
   back to the user to send instead — their call.

Deliver: a table of conversations (handle · their last message · your suggested reply · stage), plus
credits spent reading and remaining. Talk about conversations and booked calls — never DM counts.

On errors, surface the message and stop; only back off + retry once on `429`. Never fabricate a reply
that wasn't in a real conversation response.
