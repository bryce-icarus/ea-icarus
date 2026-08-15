---
name: booking-research
description: Runs on a schedule to catch newly booked "Strategy Session with Icarus" calls on Google Calendar, does a quick web-search pass on the lead's company, and emails Bryce a tailored personal text to send them before the call. Deliberately lightweight — ClickUp's Discovery Call Prep Assistant already handles the full pre-call brief.
---

# Booking Research

Detects new discovery-call bookings and gets Bryce a tailored text to send the lead before the call. Built 2026-08-14, simplified 2026-08-15 to drop Perplexity and the full research brief.

**Scope, deliberately narrow:** this is NOT a pre-call brief generator, that's already ClickUp's job (`context/work.md`: Simon assigns the ClickUp task and a "Discovery Call Prep Assistant" auto-generates a pre-call brief there). This skill exists purely to produce one thing: a short, specific, tailored text Bryce sends the lead right after they book, in his own voice. Research is a means to that end, kept as brief as the text itself, not a standalone deliverable.

**No Perplexity.** Originally planned, but its MCP server needs an API key and claude.ai's web connector flow can't take one (OAuth-only), so it can't be attached to this cloud routine. Bryce decided it's not worth chasing, use `WebSearch` instead (already available, no setup). One or two quick searches, skimmable, not a deep dive. If nothing useful turns up (thin/newer company, no web presence), that's fine, fall back to what's in the booking notes themselves, which are often enough on their own (see the ArkonLabs example below, where the lead's own Calendly submission was the best material either way).

**This fills a real gap, not an existing SOP.** `references/sops/follow-up-masterclass.md` and the rest of the SOP set script the post-call sequence (Day 1 recap/text/voice-note, Days 2-3, Day 4+) and the automated confirmation-page/reminder cadence (`confirmation-page-best-practices.md` — note: this page **does not exist yet** for Icarus, it's still a to-build item on Bryce's side, not live automation, don't treat it as already running). Nothing in the SOP set covers a personal touch from Bryce himself in the window between booking and the call. This step is an interpretive addition using the same voice rules as the rest of the SOP set (Section 1 and Section 11 of the follow-up masterclass: conviction-based, direct, personal, never "just checking in" or generic), not sourced from the original material.

**Delivery is email, not Slack.** Slack was the original plan but Icarus's workspace is company-owned (per `context/team.md`) and Bryce isn't the admin, so connecting a Slack app connector needs someone else's approval. Gmail is already connected and fully his, so the text lands in his own inbox instead. Revisit Slack later if workspace admin approval comes through.

## Step 1: Find new bookings

Use `mcp__claude_ai_Google_Calendar__list_events` (or `search_events`) for a window covering **now to 14 days out**, plus the last 24 hours (to catch same-day bookings for near-term calls). Filter to events titled like "Strategy Session with Icarus" (the Calendly booking title per `context/work.md`) — don't process other event types (onboarding calls, internal syncs).

Read `context/booking-research-log.md` for the list of already-processed events (matched by event ID, or by attendee + call time if ID isn't stable). Any calendar match not already in that log is a **new booking**.

If there are no new bookings, stop here, nothing to report.

## Step 2: Pull what's already in the booking + Close

For each new booking, pull whatever's already sitting there before searching anything:
- The calendar invite / Calendly description itself — company name, phone, and anything they wrote in the "anything that will help prepare" field. This is frequently the best material available (see the ArkonLabs example, where the lead's own words gave a sharper angle than web search did).
- A quick Close.io check (`mcp__claude_ai_Close__lead_search`) for an existing lead record — company, vertical, notes. Just enough to sharpen a search query, not a full pull.

## Step 3: One or two quick WebSearch passes

Only if the booking notes don't already give enough to write a specific line. Keep it to what the company does and maybe one recent/notable thing, skimmable in a few seconds, not a report. If nothing useful turns up, don't force it, that's a normal outcome for newer/smaller companies and the booking notes are usually enough on their own.

## Step 4: Draft the tailored personal text (Bryce's voice)

Draft a short text Bryce sends the lead himself from his personal phone, right after they book. Never sent automatically, drafted only.

**Voice rules (per follow-up-masterclass.md Sections 1 and 11):**
- Direct, personal, conviction-based. No corporate softness.
- Never generic. Must reference something *specific*, from the booking notes, the company, or a quick search, proof Bryce actually looked, not a template with the name swapped in.
- No "just checking in," no "looking forward to connecting," no filler. Say something real in one or two sentences and stop.
- Casual, like a text a person actually sends. Contractions, short sentences, no "Dear" or sign-off block.

**Shape (adapt every time, don't reuse verbatim):**
> Hey [Name], this is Bryce with Icarus. Saw you locked in [day/time] — [one specific, real line pulled from their booking notes or a quick search]. [One line of genuine excitement/conviction, tied to that specific thing]. Talk soon.

**Worked example (from the ArkonLabs demo run, sourced entirely from the booking notes, no search needed):**
> Hey Mohamed, this is Bryce with Icarus. Saw you're the ArkonLabs guy for Monday at 11 — read what you sent over about needing someone who's actually handled restricted/regulated brands without tanking the ad account. That's the whole reason we exist, excited to get into specifics with you Monday.

If truly nothing specific is available anywhere (empty booking notes, no Close history, nothing findable), say so in the email rather than forcing a hollow generic text.

## Step 5: Email it

Send via `mcp__claude_ai_Gmail__send_message` to Bryce's own address (brycestrange3@gmail.com). Subject: "Text for [Lead] — [Company]". Keep the email itself short, this isn't a brief:

```
New booking: [Lead name] — [Company], call [date/time]

[One line max: what the text below is based on, e.g. "from their booking notes" or "quick search, thin web presence otherwise"]

Text to send:
[the drafted text from Step 4]
```

## Step 6: Update the log

Append each processed booking to `context/booking-research-log.md` (event identifier, lead name, company, call time, date researched) so it's never re-sent on the next scheduled run.
