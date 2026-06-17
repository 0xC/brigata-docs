---
brigata_id: 8255c7c2-251a-412f-bf11-1a70bf33a1fc
title: "brigata-pricing-brief-for-luca"
---

# Brigata Studio — Pricing Planning Brief

**Prepared by:** Cosimo
**For:** Luca, with Chris's review
**Status:** Working document — decisions in §1 are locked, everything in §2 is open

---

## 0. One-paragraph product context

Brigata Studio is a closed-beta multi-tenant agentic workspace platform. A subscriber gets a personal workspace, can spin up AI agents in it, talk to them in Slack-style channels or a floating-window Studio mode, and have those agents create and edit shared documents. Two tiers exist today:

- **Standard agents** run in the central Brigata process against the user's own Anthropic credential. The user provides an OAuth subscription token (uses their Claude Pro/Max plan) or an Anthropic API key.
- **Pro agents** get a dedicated VPS (Hetzner or DigitalOcean, or the user's own server via BYOVPS) with shell access, cron, browser automation, and the ability to host/build small web apps.

As of 2026-05-28 the platform also supports **multiple workspaces per subscriber**, with **invite-based shared workspaces** (Slack-model collaboration). This reframes the platform from "one person + their agents" toward "people + their agents collaborating with other people + their agents."

We are currently free for closed-beta invitees. There is no billing infrastructure yet. We are roughly 3–4 weeks from being able to flip on paid public launch *if* pricing and Stripe integration are sorted in parallel.

---

## 1. Decisions already locked in

These do not need re-debating in pricing planning. Treat as constraints.

### 1.1 We never resell or mark up LLM calls
Every user runs their own agents against their own Anthropic account. We are not in the model billing chain. This is a deliberate positioning choice — "your account, your bill" — and a differentiator vs platforms that proxy / meter / mark up tokens.

### 1.2 We never mark up VPS costs
Pro-tier VPS is billed by the provider (DO, Hetzner) directly to the user's provider account. The provisioner just orchestrates. BYOVPS is free for the platform side because the user already owns the VPS.

### 1.3 Workspace creator pays compute, multi-workspace membership is free
When a workspace has multiple human members, the **workspace owner's Anthropic credential** funds all agent activity in that workspace. Members ride along without their own billing relationship to us. Subscribers can be members of N shared workspaces with no per-workspace charge — Slack model. Don't gate collaboration on cost; it's the growth vector.

### 1.4 The billable unit is the seat, not the workspace
The subscriber pays us for tier access (a seat), not per-workspace. This keeps the math simple and matches the Slack/Notion mental model users already have.

### 1.5 Closed beta is invite-only and free
Currently every signed-in user is on Chris's allowlist (env + DB allowlist table, enforced at Google OAuth callback). We extend this until paid public launch.

---

## 2. What needs deciding

### 2.1 Tier shape

Three rough shapes are on the table; each implies different pricing and product packaging.

**Option A — Two tiers (Standard, Pro) sold separately.**
- Standard subscription: flat $X/mo. Pro subscription: flat $Y/mo (which *includes* the orchestration of one VPS, but the VPS bill still goes to the user).
- A single subscriber can be on Standard or Pro.
- Easy to message. Closest to ChatGPT/Claude Pro/Team pattern.

**Option B — One subscription tier with Pro as an a-la-carte add-on per agent.**
- One $X/mo seat fee.
- Each Pro agent on the seat costs +$Z/mo on top of the base, regardless of whether it's our Hetzner provision, DO, or BYOVPS.
- Lets a subscriber have a mix of Standard and Pro agents at any number.
- Closer to what the platform actually IS (multi-agent).

**Option C — Slack-style team plans.**
- "Personal" tier (cheaper, capped agents).
- "Team" tier (more expensive per seat, unlimited shared workspaces, admin tooling).
- Education tier (heavily discounted, post-pilot).
- Most flexible long-term but most complex to design and communicate at launch.

**My instinct:** Start with A or B for launch, evolve toward C as we get usage data.

### 2.2 Numbers — anchors and starting points

Comparable products' current pricing for orientation:

| Product            | Plan          | Price          | Notes                                |
|--------------------|---------------|----------------|--------------------------------------|
| Claude Pro         | Individual    | $20/mo         | What our users likely already pay    |
| Claude Max         | Individual    | $100-200/mo    | Heavy users                          |
| ChatGPT Plus       | Individual    | $20/mo         |                                      |
| ChatGPT Team       | Per seat      | $30/mo         | 2+ seats                             |
| Notion AI          | Per seat      | $10/mo addon   |                                      |
| Slack              | Pro per seat  | $7.25/mo       | Standard collaboration anchor        |
| Cursor             | Pro           | $20/mo         | Engineer tooling                     |

Our value proposition is "platform + UX + multi-agent orchestration on top of the Claude account you already pay for." The Claude Pro user is essentially adding Brigata Studio *to* their existing $20/mo Claude bill.

**My instinct:**
- Standard seat: **$10–15/mo** (a modest addition to Claude Pro that buys real orchestration, channels, documents, collaboration).
- Pro seat or add-on: **+$15–25/mo** (the orchestration of a VPS, the bridge, the auto-provisioning, monitoring).
- Annual discount: 2 months free (roughly 17%).

These should be pressure-tested against actual unit economics — see §3.

### 2.3 Trial / free path

The current closed-beta is effectively an unlimited free tier behind invite gating. Once we launch paid, we still want a way for new users to experience the product without immediate commitment.

Options:
- **Time-bounded trial.** 14 days full access, then must convert or stop.
- **Feature-bounded free tier.** Permanent free tier, capped to 1 Standard agent, 1 workspace, no Pro, no shared workspaces. Upgrade unlocks the rest.
- **Capped-usage free tier.** Permanent free, N turns/month, then prompt to upgrade.

**My instinct:** Feature-bounded free tier. Best for SEO/word-of-mouth/sharing — anyone can sign up, anyone can taste it forever, paying unlocks the full experience. Trial countdowns create friction at exactly the wrong moment.

### 2.4 Onboarding-to-paid funnel

Currently onboarding is a 4-question wizard that seeds a workspace + Connect Claude. After that, the user is in.

To set up for paid:
- Onboarding probably needs a "Choose your plan" step at the end (with Free / Standard / Pro options), even if Standard/Pro initially route through the same closed-beta gate.
- Need clear messaging about what Pro adds — and ideally a Pro agent demo built into the onboarding so the user understands the value before being asked to pay.
- The Connect Claude wall (currently mandatory) should NOT happen during onboarding — it's a friction barrier the user will hit *after* they've seen the value. Maybe offer the wizard a "try it without your own Claude account, get N free turns" path. (But that costs us, so this needs careful thought.)

### 2.5 Upgrade paths

- **Free → Standard.** Stripe checkout, immediate unlock.
- **Standard → Pro (whole subscription).** Stripe upgrade, charge prorated.
- **Add Pro agent (under Option B).** In-app "Upgrade this agent to Pro" already exists; just needs billing wired.
- **Cancel / downgrade.** Pro agent VPS should be destroyed (or moved to BYOVPS) before downgrade is allowed.

### 2.6 Education licensing (post-pilot, parked)

Chris has flagged this as a growth lever:
- Free tier for `.edu`-verified students.
- Paid seat for instructor (or per-classroom subscription).
- Sweet spot: high-school AP CS, college intro-to-AI.
- Not pre-launch. Pilot with 1–2 friendly professors *after* paid launch.

---

## 3. What you (Luca) can help with

These are the things we genuinely need outside-of-Cosimo's-head input on:

1. **Unit economics sanity check.** At $10–15/mo Standard, what's our gross margin after Hetzner per-agent infrastructure costs for shared workspace owners who actually use it heavily? Where does that break even?

2. **Anchor research.** Pull current pricing for the comparables above and any others we should consider (Replit AI, GitHub Copilot Enterprise, Anthropic API direct, Cursor Teams, Linear AI). Surface anything in the $5–30/mo Slack/Notion-adjacent space we should know about.

3. **Trial-vs-freemium tradeoffs.** Which model do SaaS products in our shape actually convert better with? Trial countdowns vs feature-gated freemium — bring back the conventional wisdom and any contrarian counterexamples.

4. **Education licensing — where to find the friendly professors.** If we pilot post-launch, what's the path to identifying 2–3 instructors in CS / AI ethics / business who would be willing to use Brigata for a semester?

5. **Pricing psychology.** Do we land odd ($9.99) or even ($10)? Strikethrough launch-promo pricing yes/no? Annual upfront discount typical depth in this space?

---

## 4. What Cosimo will do next (engineering)

Independent of pricing decisions, the engineering work needed before paid launch:

- Stripe integration — Customer, Subscription, webhook handling, payment-method capture.
- Plan-aware enforcement (cap Standard count, gate Pro features, etc.).
- Usage metering display (turns this month, by agent) so users can see what their Claude account is doing.
- Cost dashboard ("you're on Standard, paid through 2026-07-01").
- Graceful soft-stop / dunning when payment fails.
- Provisioner reliability pass for Pro tier (it has known silent-failure modes).

None of this blocks pricing planning — pricing can happen in parallel and lock in numbers Cosimo will then wire into Stripe.

---

## 5. Working hypothesis (subject to your input)

If we had to ship pricing tomorrow with no further analysis, my recommendation would be:

- **Free tier:** 1 personal workspace, 1 Standard agent, no shared workspaces, BYO Anthropic credential. Permanent.
- **Standard:** $12/mo per seat. Unlimited Standard agents, unlimited shared workspace participation, can create up to 3 shared workspaces, BYO Anthropic credential. Annual: $120/yr (2 months free).
- **Pro:** $35/mo per seat. Everything in Standard, plus unlimited Pro agents (each requires user's own VPS billing or BYOVPS), priority support. Annual: $350/yr.
- **No usage caps from us** — Anthropic costs are between the user and Anthropic.

This is a starting point, not a commitment. Numbers especially are placeholders to argue against.

---

## 6. Open question for Chris

Before Luca dives in, one clarifying decision from Chris:

**Is the Pro tier a separate subscription, or an add-on on top of Standard?** (Option A vs Option B in §2.1.) This shapes how the comparable pricing maps to ours and changes the Stripe data model. My slight preference is Option B (add-on) because it matches what the platform actually does, but I can see the case for A.

---

*Prepared 2026-05-28 by Cosimo. Revise freely.*
