---
description: Draft a personalized first-DM for each lead from their real profile and recent tweets. Free — drafting spends no credits.
argument-hint: <lead list, or a handle to draft for>
allowed-tools: Bash(curl:*), Bash(jq:*), Read, WebFetch
---

Write a short, specific first-DM for each lead in `$ARGUMENTS`. **Drafting is free** — writing the
messages spends no credits. (The only thing that could cost credits here is optionally pulling a
lead's recent tweets for personalization, `GET /users/{handle}/tweets` at 5 credits each — do that
only if you don't already have their bio/tweets and only after saying so.)

Load the `x-outreach` skill. Read `references/dm-playbook.md` for openers, depth, and tone.

1. **Use what you already have.** If the lead rows from `/find-x-leads` already include bio and recent
   activity, draft from that — no new API calls, no spend. Only fetch more (their tweets) when a lead
   is high-value and you genuinely lack a personalization hook; if so, say "this will cost 5 credits
   per profile to pull tweets — want me to?" first.
2. **Draft one DM per lead** following the playbook:
   - Open with something **true and specific** about them (a line from their bio, a recent post, what
     they're building) — not "Hey {name}, love your content".
   - One clear reason you're reaching out that connects their situation to the offer.
   - A low-friction ask (a question or a soft "worth a look?"), never a hard "book a demo" in message #1.
   - Short — 2–4 sentences. No links in the first DM unless asked. No hashtags, no emoji spam.
3. **Vary the openers** across the list so it never reads as a template blast. If you can't find a
   genuine personalization hook for a lead, say so and suggest dropping them rather than sending
   generic filler — a weak DM hurts the account and wastes a send.
4. **Draft the follow-ups too** (optional): 1–2 short follow-ups spaced a few days apart, each adding
   a new angle, all set to stop the instant the lead replies.

Deliver a table: handle · the personalization hook you used · the drafted DM (+ optional follow-ups).
Flag any lead you couldn't personalize well. Nothing here sends anything — hand the approved drafts to
`/x-dm-campaign`, which will confirm the total credit cost before sending.

Keep it human: no em-dashes-as-crutch, no "I hope this finds you well", no fake flattery. This is a
message a real person would be glad to receive, not spam.
