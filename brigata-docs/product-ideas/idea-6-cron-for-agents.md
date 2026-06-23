---
brigata_id: 514bc4de-802e-4b7f-929b-389262897252
title: "Idea 6: Cron-for-Agents"
---

# Cron-for-Agents

## The Problem
"Run this prompt every weekday at 9am and email me the result" is a 3-line idea that takes a developer 2 days to build right (scheduler, retries, dedup, secrets, output handling, observability). Non-developers can't do it at all.

## The Wedge
Hosted scheduler that fires prompts at LLM providers on a schedule. Smaller and sharper than full agent runtime — explicitly *just* "cron + LLM + nice log UI." Natural wedge into observability (#1) and full runtime (#2).

## Core Features (MVP)
- **Schedule a prompt**: cron expression or natural language ("every weekday 9am ET")
- **Bring-your-own API key** for Anthropic / OpenAI (or use ours with markup)
- **Tool access**: web search, web fetch, send email, post to Slack, write to Google Sheet — that's it for MVP
- **Output destinations**: email, Slack, webhook, doc, Sheet row
- **Run log** — every fire, full transcript, replayable
- **Retry policy** — configurable, idempotency keys

## Phase 2
- Triggers beyond cron: webhook, email-in, RSS, "new row in Sheet"
- Multi-step workflows (chain prompts, pass outputs)
- Marketplace of public schedules ("daily AI news summary")
- Team workspaces

## Tech Stack (suggested)
- Postgres + a real job queue (River, Oban-style, or Temporal)
- Next.js dashboard
- Modal/Inngest-style execution if you don't want to run your own workers

## Distribution
- Free tier: 30 runs/mo
- Paid: $19/mo for 1,000 runs, $99 for 10,000
- Target: indie hackers, ops people, solo founders, marketers
- Distribution: Product Hunt, X, "show me your cron" templates

## Build Estimate
- 1–2 engineers, ~6 weeks to MVP
- Cheapest, fastest of the six to ship
- Best candidate for "validate fast, expand or kill"

## Risks
- Easy for Zapier/Make to add ("AI step")
- Easy for OpenAI/Anthropic to ship natively
- Differentiation has to be on UX + log clarity, not features

## Notes
Lowest risk, lowest ceiling. Good first ship to test the audience and seed it into the bigger plays above. Could be free-forever as a top-of-funnel for #1 or #2.
