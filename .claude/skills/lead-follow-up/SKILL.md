---
name: lead-follow-up
description: Use after Bryce takes a non-close call, when he asks to "run follow-up," "send recaps," or "do the follow-up sequence," or when checking whether the pipeline is due for a Day 2-3 touch, a Day 4+ handoff, a cold reactivation, or the end-of-month squeeze. Drafts everything (recap email, personal text, voice note script) for Bryce to review and send himself. Nothing sends automatically.
---

# Lead Follow-Up

Implements `references/sops/follow-up-masterclass.md` for Icarus. Read that file first if it's not already in context, it's the source of truth for tone, structure, and cadence timing.

**Automation boundary (per Bryce, confirmed Aug 2026):** Days 1-3 stay draft-and-review, this skill never sends an email itself and never sends a text (texts must come from Bryce's personal phone per the SOP). Two Close.io workflows exist for what happens after:

- `Icarus Follow-Up: Day 4+ Sequence` (`seq_5ihhG5wGdldYXDVSjbbEHR`) — took the call, didn't close, went quiet. 4 emails at ~day 4/8/14/21. Draft status, `lead-manual` trigger: only fires once Bryce (a) switches it to Active in Close and (b) manually enrolls a specific lead.
- `Icarus No-Show Re-Engagement (Auto)` (`seq_5SUflaFnSekcrpnNBYyyvK`) — booked a discovery call and never showed. 3 emails at ~1hr/day3/day7 after enrollment, each linking Bryce's real Calendly scheduling link to rebook. Draft status, but **`lead-event` trigger** filtered to the "No Show" lead status (`stat_yTkv6sYexeEqRhUeJ9GojDr5Ro1rfW1M2z9FzBqOAlG`), per Bryce's choice, this one only needs (a) switched to Active, then it fires automatically the moment a lead's Close status is set to No Show, no separate enrollment step. That means **updating a lead's status to No Show is itself the trigger** once the workflow is Active, treat that status change with the same care as enrolling in a workflow directly, always get Bryce's explicit go-ahead first (see Step 5), never set it speculatively or for record-keeping only.
  - There's also a now-unused older version, `Icarus No-Show Re-Engagement` (`seq_2ZmnyaXkv9JIGYQwe8Clis`, `lead-manual` trigger, still Draft), superseded by the auto version above. It can't be deleted via the available Close tools (no delete/update workflow action exists), Bryce needs to delete it manually in the Close UI. Don't reference or use it.

Never switch either live workflow to Active, change its trigger, or build another live-sending workflow without checking with Bryce first.

## Step 1: Figure out what's being asked

Three possible entry points:
1. **Day-1 sequence for specific call(s)** — Bryce says something like "run follow-up for today's calls" or names a lead. Go to Step 2.
2. **Pipeline cadence check** — Bryce asks "who's due for follow-up," "what am I missing," or this is triggered from the daily-plan flag (Day 2-3 touches, end-of-month squeeze). Go to Step 5.
3. **Both** — default when he just says "run follow-up" with no qualifier. Do Step 2 first, then Step 5.

Run the Step 5 pipeline check by default every time this skill is invoked, even when Bryce only asked about today's calls — with call volume this high, Day 2-3 leads are the ones most likely to silently fall through, and a quick check costs little.

## Step 2: Identify the call(s) and pull lead context

- If not already known, use `mcp__claude_ai_Google_Calendar__list_events` for today (or the date Bryce specifies) to find the scheduled call(s) and attendee names.
- **Confirm each one actually happened before drafting anything.** A calendar event only means it was scheduled, not that it ran (no-shows and reschedules happen). Check Gmail for a Gemini notes or read.ai transcript/summary email matching that lead/company and roughly that time (same logic as `daily-plan` Step 3's "Call confirmed" bucket) — `mcp__claude_ai_Gmail__search_threads` for something like `from:gemini-notes@google.com OR "meeting notes" OR "meeting report"` scoped to the last day or two. If Bryce names the call directly ("run follow-up for the call with X") or explicitly says it happened, that's confirmation enough, skip the transcript check.
- If a scheduled call has no transcript yet and Bryce didn't confirm it directly, don't draft the sequence for it, tell him it's still awaiting confirmation instead.
- For each confirmed call, look up the attendee in Close.io (`mcp__claude_ai_Close__lead_search` or `find_contact_custom_fields`/`fetch_lead`) to get: company, vertical, opportunity/lead status, notes from past activity, any objections logged.
- If a lead isn't in Close yet or the match is low-confidence, flag it to Bryce rather than guessing.
- **Skip closed-won calls** — those don't get the follow-up sequence, they get onboarding (see `context/work.md` delivery process).

## Step 3: Draft the Day-1 sequence assets

For each non-close call, produce all three pieces using Section 2 of the SOP:

**1. Recap email** (Gmail draft via `mcp__claude_ai_Gmail__create_draft`, addressed to the lead, do NOT send):
- Subject: "Recap from our call" or "Recap: what [outcome] looks like for you"
- Warm open → quick recap of what was discussed → program overview video → confirm the next call date/time → conviction close.
- **No program overview video exists yet** (confirmed with Bryce Aug 2026). Leave a `[program overview video, not yet built]` placeholder rather than a fake or broken link, and mention it in your summary to Bryce so it doesn't get missed as a to-build item.
- Contextualize to the lead's actual vertical/objections from Close, don't send a generic template.

**2. Personal text** — plain copy, not sent by Claude (no SMS integration; SOP requires it come from Bryce's real phone anyway). Short, personal-number intro line per Section 2 Step 2.

**3. Voice note script** — written script per the Section 2 Step 3 framework, adapted with the lead's name, call time, next call day, and their specific goal/offer. Pure conviction, no feature-selling, per the SOP's rules.

## Step 4: Stage the manual steps in Close

For each lead, create a Close.io task (`mcp__claude_ai_Close__create_task` or `create_call_task`) due today, assigned to Bryce, with the text copy and voice note script pasted into the task body so he can act on them without hunting for this conversation. Something like:

> **Send Day-1 follow-up (text + voice note): [Lead name]**
> Text: "[drafted text]"
> Voice note script: "[drafted script]"

Also drop a short Close note on the lead confirming the Day-1 sequence assets were generated today, so there's a record even before Bryce sends anything.

## Step 5: Pipeline cadence check

Check five timing triggers. Use `mcp__claude_ai_Close__activity_search` or `lead_search` filtered by last-activity date rather than assuming a custom field exists — if Bryce wants a dedicated "cadence stage" field built into Close later, flag it as an option rather than building it unprompted (schema changes are a shared-CRM change).

- **Day 2-3 leads (the easy-to-forget window)**: non-close calls from 1-3 days ago. The SOP is explicit that Days 1-3 are fully manual, but only defines Day 1's content (the recap/text/voice-note triad) and what happens from Day 4+ (the Close workflow). It doesn't script Day 2 or Day 3 specifically, so this skill fills that gap: a **light personal-phone text touch**, one per day, each a different angle so nothing repeats (a quick value-add tied to something from the call, a proof point, a light nudge on the upcoming follow-up call). Never the Day-1 recap again, never "just checking in." For each lead in this window who hasn't had a touch logged today, draft the text copy and flag it to Bryce by name and day-count ("Day 2, texted him about X yesterday" style), same staging pattern as Step 4 (Close task with the copy, not sent by Claude). This bucket is the one most likely to get missed on a heavy call day, surface it every time this skill runs, not just when asked.
- **Ready to move from Day 3 to the Close automation**: non-close calls that just crossed out of the Day 2-3 manual window (day 4+) with no response yet, and not already enrolled in `Icarus Follow-Up: Day 4+ Sequence`. This is a handoff point, not automatic, explicitly call it out to Bryce by name ("[Lead] is at day 4, ready to move into the Close sequence") and ask whether to enroll them (only relevant once he's switched that workflow to Active) or draft a one-off Gmail follow-up instead. Surface this every time the pipeline check runs, including from `daily-plan`, not just on request.
- **Likely no-shows**: a lead had a discovery call scheduled per the calendar, but `daily-plan` Step 3 found no Gemini notes / read.ai transcript for it by the next check. That's a signal, not proof, confirm with Bryce before acting ("looks like [Lead] no-showed the [day] call, no transcript came through, want me to log it as a no-show?"). On confirmation: update the lead's Close status to **No Show** (`stat_yTkv6sYexeEqRhUeJ9GojDr5Ro1rfW1M2z9FzBqOAlG`). That status change is now the trigger for `Icarus No-Show Re-Engagement (Auto)` (`seq_5SUflaFnSekcrpnNBYyyvK`), no separate enrollment step, so make sure Bryce is confirming both "yes it's a no-show" and "yes, start the re-engagement sequence" in that one go, don't treat the status update as a lesser action just because it looks like a label change. Still only fires for real once he's switched that workflow to Active.
- **Cold reactivation (2-3 months)**: leads with no activity in 60-90+ days who haven't opted out. Draft fresh-angle re-engagement copy per Section 5 of the SOP (changed offer, split-pay mention, pattern-interrupt humor), never the same message twice on the same lead.
- **End-of-month squeeze**: only relevant in the last ~7 days of the month or end of quarter. Check today's date. If in window, tell Bryce it's open and offer to draft the squeeze email (Section 7: re-offer, 8-Mile the resistance, Sales Competition Frame, direct booking link) for a batch send, or the third-party/manager check-in variant if it's end of quarter and hasn't been used this quarter.

Don't run a full-pipeline blast draft unprompted, this list can be hundreds of leads. Surface the count and ask whether Bryce wants it drafted now or batched.

## Step 6: Present the summary

Per `.claude/rules/communication-style.md` (bullets, no em dashes, no emojis, casual internal tone):

1. **Day-1 sequences drafted today** — which leads, link to Gmail drafts, note that Close tasks are staged with the text/voice note copy.
2. **Day 2-3 touches due** — named leads with a day-count, lead this section since it's the easiest to lose track of, not buried at the bottom.
3. **Ready to move to the Close automation** — named leads at day 4+, ask before enrolling.
4. **Likely no-shows** — named leads with no transcript found, ask before logging as No Show or enrolling in re-engagement.
5. **Anything flagged** — missing/low-confidence Close matches, the video placeholder reminder.
6. **Pipeline cadence** — cold reactivation count, end-of-month squeeze window status (open/not open, sent this month or not).
7. Ask before drafting a full-pipeline batch send.
