---
name: day1-followup-auto
description: Runs on a schedule (hourly, business hours, same as booking-research) to catch new Gemini notes / read.ai call transcripts in Gmail and automatically run the Day-1 follow-up sequence for each newly confirmed non-close call — recap email drafted in Gmail, personal text + voice note script staged on a high-priority Close task, confirmation note logged. Companion automation to booking-research.
---

# Day-1 Follow-Up (Auto)

Automates what `lead-follow-up`'s Step 2-4 already does manually, triggered by a new call transcript landing in Gmail instead of Bryce asking for it. Built 2026-08-15, same day as the Nic Nac manual walkthrough this mirrors exactly.

**Still never sends anything.** Same automation boundary as `lead-follow-up` and the whole SOP set: recap email is a Gmail draft, not sent; text and voice note are copy staged on a Close task, Bryce sends/records them himself from his personal phone. This routine only adds automatic *triggering* and *drafting*, not automatic *sending*.

**No missing dependencies.** Uses Gmail, Google Calendar, and Close, all three already attached to the `booking-research-business-hours` routine's connectors, reused here on the same routine setup, no new connector needed.

## Step 1: Find new transcripts

Search Gmail for recent transcript emails, per the same pattern `lead-follow-up` Step 2 already uses: `from:gemini-notes@google.com OR "meeting notes" OR "meeting report"`, scoped to roughly the last 3 hours (routine runs hourly, so this window comfortably overlaps the previous run with no gaps).

Read `context/day1-followup-log.md` for already-processed message IDs. Skip anything already logged.

If nothing new, stop here.

## Step 2: Match transcript to a real external sales call

For each new transcript, identify the attendees and match against the calendar event and Close. Skip (don't log, don't process) if:
- All attendees are `@icarusdigitalmarketing.com` (internal meeting, not a sales call)
- No matching Close lead can be found with reasonable confidence

If the lead match is low-confidence, flag it in the summary rather than guessing (same rule as `lead-follow-up`).

## Step 3: Skip closed-won and already-processed leads

Check the lead's Close status. Skip closed-won calls entirely, those get onboarding, not the follow-up sequence, per `context/work.md`.

Also check the lead's notes/activity for an existing "Day-1 follow-up sequence generated" note (this routine and manual `lead-follow-up` runs both leave one). If found, skip, it's already been done, don't draft a second time.

## Step 4: Pull Close context

Same as `lead-follow-up` Step 2: company, vertical, opportunity status, existing notes, any objections logged, and whether a proposal has already gone out.

**Stage 1 proposal timing — default assumption is NOT sent.** Per `context/work.md`, the paid-media plan takes a while to build, it's not a same-day deliverable by default. Check Gmail sent mail for an actual "Your Icarus paid-media plan" email to this lead's address. Only treat it as sent if that email is actually found (the Nic Nac run confirmed one this way, that's the exception, not the default). If not found, the recap should mention the plan is coming, not imply it's already out or invent a delivery date.

## Step 5: Draft the Day-1 triad

Use `references/sops/follow-up-masterclass.md` Section 2 for structure and Section 1/11 for voice (conviction-based, direct, contextualized to the lead's actual call content from the transcript, never generic).

1. **Recap email** — Gmail draft (`mcp__claude_ai_Gmail__create_draft`), addressed to the lead, NOT sent. Warm open referencing something real from the transcript, quick recap of what was discussed, `[program overview video, not yet built]` placeholder, mention of the Stage 1 plan per Step 4 (already-sent reference only if confirmed, otherwise phrased as coming), conviction close, ask for a follow-up time if none is already booked.
2. **Personal text** — plain copy, references something specific.
3. **Voice note script** — plain copy, pure conviction, references something specific from the call.

## Step 6: Stage the Close task

`mcp__claude_ai_Close__create_task`, `priority: high`, assigned to Bryce, text/voice note copy pasted directly into the task body, same format as the Nic Nac example:

> **Send Day-1 follow-up (text + voice note): [Lead name]**
> Text: "[drafted text]"
> Voice note script: "[drafted script]"
> Recap email drafted in Gmail (not sent). [Any other action items surfaced in the transcript's "next steps" section, e.g. compliance checks, deck sends.]

## Step 7: Log the confirmation note

`mcp__claude_ai_Close__create_note` on the lead: "Day-1 follow-up sequence generated (recap email Gmail draft, personal text + voice note staged as a task). Call confirmed via [Gemini notes/read.ai], [date]." Also note any data-quality issues spotted in passing (duplicate lead records, stale status not reflecting the call/proposal), same as the Nic Nac run, don't fix them silently, just flag.

## Step 8: Update the log

Append to `context/day1-followup-log.md`: transcript message ID, lead name, company, call date, date processed. This is what Step 1 checks to avoid reprocessing.

## Step 9: Summary

If run interactively (not the scheduled routine), summarize per `.claude/rules/communication-style.md`: which leads got the sequence, any skipped (closed-won, already-processed, low-confidence match), and any data-quality flags. The scheduled routine's run log serves as the record when unattended, no separate report needed.
