---
description: Define an ideal-customer profile for X outreach — bio keywords, follower bands, engagement signals, and seed accounts to mine. Keyless, no spend.
argument-hint: <product or who you sell to>
allowed-tools: Read, WebFetch
---

Build a tight ideal-customer profile (ICP) for cold DMs about `$ARGUMENTS`. This is free, keyless
work — no API key and no credits. It is also the highest-leverage step: a sharp ICP is the
difference between a 30% and a 3% reply rate.

Load the `x-outreach` skill.

1. **Understand the offer.** If `$ARGUMENTS` is a product/URL, fetch it and figure out what it does,
   who it's for, and the one problem it removes. Ask the user for anything critical that's missing
   (price band, the outcome they sell, who has actually bought).
2. **Describe the person, not just the company.** Define the ICP on X in terms the API can actually
   target:
   - **Bio keywords / phrases** they'd have ("founder", "indie hacker", "head of growth", a stack, a
     niche) — these feed `/users/search` and post-filtering.
   - **Follower band** (e.g. 500–20k) — big enough to be real, small enough to still read DMs.
   - **Buying signal / trigger** — what recent behavior means they need this now (complaining about a
     problem, launching something, hiring, using a competitor).
   - **DM-ability** — the API's `can_dm` flag is the hard filter; the ICP should assume you only keep
     `can_dm: true` profiles.
3. **Pick the seed sources** — the concrete places to mine leads next:
   - **Seed accounts** whose *followers* or *following* are your audience (competitors, tools they
     use, communities' figureheads) → `/users/{handle}/followers`.
   - **Keyword/phrase searches** → `/users/search` (people) and `/tweets/search` (recent posts about
     the problem, then their authors/engagers).
   - **Specific tweets** whose *repliers/retweeters* self-selected as interested → `/tweets/{id}/replies`
     (note: these are premium, 20 credits/row — use sparingly on the highest-signal tweets).
   - **X Lists** someone already curated → `/lists/{id}/members`.
4. **Anti-signals.** List who to exclude (wrong seniority, agencies if you sell to end users, obvious
   bots/giveaway accounts, non-buyers) so the lead-finding step can filter them out.

Deliver: a one-paragraph ICP statement, a table of bio keywords / follower band / signals, and a
ranked list of 3–6 concrete seed sources with the exact endpoint each maps to and its per-row credit
cost (regular 5 / premium 20). End by pointing to `/find-x-leads` to pull the first list — and remind
that that step spends credits and will show the estimate first.

Never invent audience data or engagement stats — keep this grounded in the offer and the user's input.
