---
name: booking-research
description: Runs on a schedule to catch newly booked "Strategy Session with Icarus" calls on Google Calendar, researches the lead/company via Perplexity, and sends Bryce a Slack DM with the brief before the call. Requires Perplexity MCP and Slack MCP to be connected — see Prerequisites.
---

# Booking Research

Detects new discovery-call bookings and gets Bryce a research brief on the lead before he's on the call with them. Built 2026-08-14 per Bryce's request.

## Prerequisites (check every run)

This skill needs two MCP servers that were **not connected** when it was built:

1. **Perplexity MCP** — for the research step.
2. **Slack MCP** — for the delivery step (Bryce chose Slack DM over email/SMS).

At the start of each run, check whether the required tools are available (`ToolSearch` with a query like `select:` a known tool name, or check the deferred-tools listing in context). If either is still missing:
- Do the parts you can (calendar scan, Close lookup) and log the new booking(s) as "detected, pending research" in `context/booking-research-log.md` rather than silently skipping.
- Note that once the missing server is connected, the skill should be re-run to backfill.
- Don't fail the whole run for a missing dependency, degrade gracefully.

## Step 1: Find new bookings

Use `mcp__claude_ai_Google_Calendar__list_events` (or `search_events`) for a window covering **now to 14 days out**, plus the last 24 hours (to catch same-day bookings for near-term calls). Filter to events titled like "Strategy Session with Icarus" (the Calendly booking title per `context/work.md`) — don't process other event types (onboarding calls, internal syncs).

Read `context/booking-research-log.md` for the list of already-processed events (matched by event ID, or by attendee + call time if ID isn't stable). Any calendar match not already in that log is a **new booking**.

If there are no new bookings, stop here, nothing to report.

## Step 2: Pull what's already known

For each new booking, identify the lead from the calendar invite (name, email, and company if the invite/Calendly booking includes it).

Check Close.io first (`mcp__claude_ai_Close__lead_search`) for an existing lead record — company, vertical, source, any notes. This is faster than external research and gives Perplexity a sharper query (e.g. "Kylo Peptides" instead of just a first name).

## Step 3: Research via Perplexity

Use the Perplexity MCP tool (exact tool name depends on how it's connected — check the deferred-tools listing) to research the lead's company:
- What the company does, product/vertical (especially relevant since Icarus only serves high-risk niches — THC, CBD, peptides, adult, etc, per `context/work.md`)
- Company size/stage signals (funding, team size, other agencies used, if findable)
- Recent news or public activity (launches, press, social presence)
- Anything relevant to the compliance/high-risk angle Icarus sells against (are they already running ads elsewhere, have they been shut down by Meta before, etc, if publicly findable)

Keep it factual, don't speculate beyond what's found. If Perplexity finds nothing useful (thin web presence), say so rather than padding the brief.

## Step 4: Compose the brief

Format per `.claude/rules/communication-style.md` (bullets, no em dashes, no emojis, casual internal tone). Keep it scannable, Bryce is reading this on his phone before a call:

```
New booking: [Lead name] — [Company]
Call: [date/time]

Close.io: [existing lead status/notes, or "not in Close yet"]

Research:
- [company/product summary]
- [size/stage signal]
- [recent news, if any]
- [compliance/vertical fit note, if relevant]

[One-line suggested angle for the call, only if something concrete surfaced]
```

## Step 5: Send the Slack DM

Send the brief as a direct message to Bryce via the Slack MCP tool. If Bryce is in multiple Slack workspaces or the tool needs a user ID/channel, use his own DM channel (self-DM or "Slackbot" style) — confirm the right target with Bryce the first time this runs successfully, then it's a known quantity for future runs.

## Step 6: Update the log

Append each processed booking to `context/booking-research-log.md` (event identifier, lead name, company, call time, date researched) so it's never re-sent on the next scheduled run.
