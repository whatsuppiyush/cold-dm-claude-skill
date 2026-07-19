# 📩 X Outreach — a Claude skill

**Run cold outreach on X (Twitter) the right way — find DM-able leads, write DMs worth replying to,
and send paced, cost-transparent campaigns — all from your agent, using your own API key.**

Your agent finds followers/engagers who can actually be DMed, drafts a personalized first message from
each person's real profile, and sends at a safe, human pace through the **XAutoDM API** — showing you
the exact credit cost before every spend and asking first.

It's opinionated: it optimizes for **replies and conversations started**, warms up accounts, and
**refuses spray-and-pray**. Targeted outreach, not spam.

---

## Install

With the [`skills` CLI](https://github.com/obra/skills):

```bash
npx skills add whatsuppiyush/cold-dm-claude-skill
```

Or clone it into your skills folder manually:

```bash
git clone https://github.com/whatsuppiyush/cold-dm-claude-skill \
  ~/.claude/skills/x-outreach
```

Then set your API key so the skill can act:

```bash
export XAUTODM_API_KEY="xdm_live_..."   # or xdm_test_... to dry-run the setup, spending nothing
```

If the slash commands don't auto-register, copy them once:

```bash
mkdir -p .claude/commands && cp ~/.claude/skills/x-outreach/commands/*.md .claude/commands/
```

Compatible with Claude Code and any agent that loads Anthropic-style skills (a `SKILL.md` with
`commands/` and `references/`).

---

## How the API key works

- **You bring your own key.** Get one, self-serve, at
  **[api.xautodm.com](https://api.xautodm.com/?ref=claude-skill)**. Set it as `XAUTODM_API_KEY`. The
  skill never ships a key and never sees XAutoDM's internal credentials — every action runs on *your*
  key and *your* connected X account.
- **Paid and metered, honestly.** XAutoDM is a paid product — **there's no free trial.** New accounts
  start on a $0 / 0-credit plan and need a subscription plus a connected X account before anything
  sends. Check your current plan and pricing at [api.xautodm.com](https://api.xautodm.com/?ref=claude-skill).
- **Cost-transparent by design.** Every action has a known credit price, and every response tells you
  exactly what it charged and what's left. The skill estimates the cost and **asks before spending** —
  every time.
- **Dry-run first.** A `xdm_test_…` key validates your whole setup and returns real response shapes
  while spending nothing and sending nothing — use it to verify before you go live.

---

## Usage

| Command | What you get | Spends credits? |
|---|---|---|
| `/x-setup` | Validate your key; see plan, credit balance, and connected accounts. Run this first. | No |
| `/x-icp <product>` | A tight ideal-customer profile: bio keywords, follower bands, buying signals, seed accounts to mine. | No |
| `/find-x-leads <query>` | Search users/tweets, pull followers of a seed account, filter to DM-able, dedupe → a lead list. Shows cost first. | Yes |
| `/draft-x-dms <leads>` | A personalized first-DM per lead, written from their real bio + tweets. | No |
| `/x-dm-campaign <leads + msg>` | Confirms the total cost, then sends at a safe pace and reports replies started. | Yes |
| `/x-inbox` | List conversations + replies; draft responses to warm leads. | Yes |
| `/x-engage <handles>` | Warm up prospects (follow / like / reply) a day or two before the DM. | Yes |

No slash commands? Just ask in plain language — *"find X leads who follow @levelsio and can be
DMed"* or *"draft cold DMs for these 20 founders"* — the skill triggers on intent.

### Credit costs at a glance

1 credit = $0.005. The per-action prices are stable, so your agent can price a campaign before it runs:

| Action | Credits | ≈ USD |
|---|---:|---:|
| Read a profile / tweet | 5 | $0.025 |
| Lead row (follower / search / list member) | 5 | $0.025 |
| Premium lead (verified follower / tweet engager) | 20 | $0.10 |
| Send a DM | 10 | $0.05 |
| Read a DM conversation | 10 | $0.05 |
| Follow / like / reply / tweet | 5 | $0.025 |
| Connect / reconnect an X account | 50 | $0.25 |
| Health · account · usage · list accounts | free | — |

Plan (subscription) prices change over time — check the live plans at
[api.xautodm.com](https://api.xautodm.com/?ref=claude-skill). The per-action costs above are what the
skill quotes for estimates.

### Worked example

```
/find-x-leads followers of @levelsio who can DM
```

→ the skill estimates the cost ("pulling ~500 followers ≈ 2,500 credits ≈ $12.50 — proceed?"), waits
for your yes, then pulls the list, keeps only `can_dm: true` profiles, dedupes them, and hands back a
clean lead table with each handle, follower count, and bio snippet. From there `/draft-x-dms` writes a
personalized opener for each (free), and `/x-dm-campaign` confirms the send cost (`120 × 10 = 1,200
credits ≈ $6`) and sends them paced over a few days — reporting replies, not send counts.

---

## What's inside

```
cold-dm-claude-skill/
├── SKILL.md                 # the methodology + credit table + safety rails + how the API gate works
├── commands/                # 7 slash commands (setup, icp, find, draft, campaign, inbox, engage)
└── references/
    ├── dm-playbook.md        # openers, personalization depth, follow-up timing, reply handling
    ├── account-safety.md     # what gets accounts flagged + safe daily pacing + warm-up-first
    └── api-reference.md      # endpoints, auth, idempotency, errors, credit costs
```

---

## What it won't do

- **It never touches X directly.** No cookies, no scraping x.com, no browser automation — every action
  goes through the XAutoDM API with your key and your connected account.
- **It won't spam.** Mass-blast, spray-and-pray, buy-a-list-and-hammer-it — the skill declines and
  steers you back to a tighter list. Untargeted volume gets accounts banned and wastes your credits.
- **It won't spend silently.** No credit leaves your balance without an estimate and your explicit yes.
- **It won't brag about send counts.** Success here is measured in replies, conversations, and booked
  calls — not "N DMs sent".

---

## The product behind the skill

The skill is genuinely useful on its own — the ICP work and DM drafting are free and keyless. The parts
that touch X (finding leads, sending, the inbox) run on the **[XAutoDM API](https://xautodm.com)**, a
managed layer that holds your X session alive, sends reliably, and bills in simple credits. Get a key
at **[api.xautodm.com](https://api.xautodm.com/?ref=claude-skill)** · full docs at
**[xautodm.com/api/docs](https://xautodm.com/api/docs)**.

---

## Notes & disclaimer

This skill calls a **paid, metered API using your own API key** — real actions on X spend real credits
from your balance. Costs shown are estimates based on stable per-action prices; the skill reads the
actual `credits_charged` and `credits_remaining` back from each live response and never fabricates a
lead, a `can_dm` flag, or a credit number. You are responsible for your own outreach staying within
X's terms and applicable anti-spam law — the skill's safety rails (targeting, pacing, warm-up,
stop-on-reply) are built to keep you on the right side of both.

Built by [Piyush](https://github.com/whatsuppiyush) · Cold DMs on X that start conversations, powered
by [XAutoDM](https://xautodm.com).

## License

[MIT](./LICENSE) — use it, fork it, ship it.
