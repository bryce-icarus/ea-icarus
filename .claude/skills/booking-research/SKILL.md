---
name: booking-research
description: Runs on a schedule to catch newly booked "Strategy Session with Icarus" calls on Google Calendar, researches the lead/company via Perplexity, drafts a personal pre-call text in Bryce's voice, and emails Bryce one email with both before the call. Requires Perplexity MCP to be connected — see Prerequisites.
---

# Booking Research

Detects new discovery-call bookings and gets Bryce a research brief on the lead, plus a personal pre-call text draft, before he's on the call with them. Built 2026-08-14 per Bryce's request, pre-call text draft added 2026-08-15.

**This fills a real gap, not an existing SOP.** `references/sops/follow-up-masterclass.md` and the rest of the SOP set script the post-call sequence (Day 1 recap/text/voice-note, Days 2-3, Day 4+) and the automated confirmation-page/reminder cadence (`confirmation-page-best-practices.md`, marketing-owned, not something Bryce sends). Nothing covers a personal touch from Bryce himself in the window between booking and the call. This step is an interpretive addition using the same voice rules as the rest of the SOP set (Section 1 and Section 11 of the follow-up masterclass: conviction-based, direct, personal, never "just checking in" or generic), not sourced from the original material.

**Delivery is email, not Slack.** Slack was the original plan but Icarus's workspace is company-owned (per `context/team.md`) and Bryce isn't the admin, so connecting a Slack app connector needs someone else's approval. Gmail is already connected and fully his, so the brief goes to his own inbox instead. Revisit Slack later if workspace admin approval comes through.

## Prerequisites (check every run)

This skill needs **Perplexity MCP**, which was **not connected** when it was built, for the research step.

At the start of each run, check whether the Perplexity tool is available (`ToolSearch` with a query like `select:` a known tool name, or check the deferred-tools listing in context). If it's still missing:
- Do the parts you can (calendar scan, Close lookup) and log the new booking(s) as "detected, pending research" in `context/booking-research-log.md` rather than silently skipping.
- Note that once Perplexity is connected, the skill should be re-run to backfill.
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

## Step 4: Compose the research section

Format per `.claude/rules/communication-style.md` (bullets, no em dashes, no emojis, casual internal tone). Keep it scannable, Bryce is reading this on his phone before a call. This becomes the "Research" block in the Step 6 email, covering: company/product summary, size/stage signal, recent news (if any), compliance/vertical fit note (if relevant), and a one-line suggested angle only if something concrete surfaced.

## Step 5: Draft a personal pre-call text (Bryce's voice, not sent)

Draft a short text Bryce could send from his own personal phone, same spirit as the Day-1 personal text in the follow-up masterclass but adapted for before the call instead of after. Never actually send it, texts are never automated per that SOP, this is a draft for Bryce to copy and send himself.

**Voice rules (per follow-up-masterclass.md Sections 1 and 11):**
- Direct, personal, conviction-based. No corporate softness.
- Never generic. Must reference something *specific* from the booking notes, the company, or the research, proof Bryce actually looked at their situation, not a template with the name swapped in.
- No "just checking in," no "looking forward to connecting," no filler. Say something real in one or two sentences and stop.
- Casual, like a text a person actually sends. Contractions, short sentences, no "Dear" or sign-off block.

**Shape (adapt every time, don't reuse verbatim):**
> Hey [Name], this is Bryce with Icarus. Saw you locked in [day/time] — [one specific, real line: what they said they need, what their company does, or something the research turned up]. [One line of genuine excitement/conviction about the call, tied to that specific thing]. Talk soon.

**Worked example (from the ArkonLabs demo run):**
> Hey Mohamed, this is Bryce with Icarus. Saw you're the ArkonLabs guy for Monday at 11 — read what you sent over about needing someone who's actually handled restricted/regulated brands without tanking the ad account. That's the whole reason we exist, excited to get into specifics with you Monday.

If nothing specific surfaced (thin booking notes, no research findings), say so rather than forcing a generic line, flag it in the email instead of drafting a hollow text.

## Step 6: Email the brief

Send the brief via `mcp__claude_ai_Gmail__send_message` to Bryce's own address (brycestrange3@gmail.com), subject line like "Booking research: [Lead] — [Company]", so it lands in his inbox before the call. Include the Step 5 text draft in its own clearly-labeled section at the bottom, tried as a separate Gmail draft first, but Bryce prefers everything in one email:

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

---
Pre-call text draft (send from your own phone, not sent automatically):
[the drafted text from Step 5]
```

## Step 7: Update the log

Append each processed booking to `context/booking-research-log.md` (event identifier, lead name, company, call time, date researched) so it's never re-sent on the next scheduled run.
