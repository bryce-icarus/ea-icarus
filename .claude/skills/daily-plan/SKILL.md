---
name: daily-plan
description: Use whenever Bryce asks for his plan for the day, what's on his plate, what he needs to handle, or his task list. Confirms today's date, pulls today's calls from Google Calendar, and refreshes the pending-email-responses list from a live Gmail scan.
---

# Daily Plan

Builds Bryce's "what do I need to handle today" view from two live sources: Google Calendar and Gmail. Always re-run the scans below — don't just recite the last saved snapshot in `context/pending-responses.md`, since it goes stale fast.

## Step 1: Confirm the date

Read the `currentDate` value injected into context (system reminder at the top of the session). State the actual weekday + date at the top of the reply (e.g. "Thursday, Aug 13, 2026") — don't assume or guess from email timestamps, since the inbox contains dated correspondence that can be misleading.

## Step 2: Pull today's calendar

Use `mcp__claude_ai_Google_Calendar__list_events` with `startTime` = today 00:00:00 and `endTime` = today 23:59:59 (ISO 8601, today's date from Step 1). Omit `timeZone` so it resolves to the calendar's own default (Bryce's calendar runs Central Time historically, but don't hardcode it — let the API resolve it).

For each event, pull: time, title, attendees (who the call is with — cross-reference against Close if the name isn't obviously a lead/client). Flag anything that's a "Strategy Session" / discovery call as a lead call worth a pre-call brief mention if one exists in ClickUp/email.

**This tells you what's scheduled, not what actually happened.** A calendar event doesn't confirm a call took place, it could be a no-show, a reschedule, or Bryce could take a call that was never on the calendar at all. Step 3 is what confirms a call actually ran.

## Step 3: Refresh pending email responses + confirm which calls actually happened

Re-scan Gmail rather than trusting the stored file. Use `mcp__claude_ai_Gmail__search_threads` with a query like:

```
in:inbox newer_than:14d -from:notifications@tasks.clickup.com -from:notifications@calendly.com -from:team@mail.clickup.com -from:teamzoom@e.zoom.us -from:teamcalendly@send.calendly.com -from:support@close.com -from:support+notifications@close.com -from:no-reply@accounts.google.com -from:notifications-noreply@linkedin.com -from:messages-noreply@linkedin.com -from:noreply@skool.com -from:membersuccess@alignable.com -from:help@clickup.com -from:mailer-daemon@googlemail.com -from:communications@b2binfo.verizonwireless.com -from:simon@icarusdigitalmarketing.com
```

Note `gemini-notes@google.com` is deliberately **not** excluded here anymore (it used to be), those emails are now the confirmation signal, see below.

Add any new noise senders you spot (newsletters, other automated tool notifications) to that exclusion list as you find them — the list only needs to grow. Don't add Gemini notes or read.ai back to it.

For each remaining thread, read the message order and classify:
- **Needs a reply — high priority**: last message is from an external lead/client/partner, addressed to Bryce, with a question or ask he hasn't answered. Also include anything where Bryce said "let me get back to you" / "let me confirm" and never followed up, even if it's not the literal last message.
- **Stalled — nudge needed**: Bryce replied last, but it's been several days of silence from the other side, especially if they'd expressed urgency (e.g. "want to launch this week").
- **Low priority / optional**: cold vendor pitches, partnership fishing with no real stakes, podcast/guest-spot asks.
- **FYI / action trigger, not a reply**: contract fully signed, payment received, etc. — things that need an internal next step but not an email reply.
- **Call confirmed (new bucket)**: sender is `gemini-notes@google.com`, or sender/subject indicates a read.ai meeting report/summary/transcript. Each of these confirms a specific call actually happened. Match it to a calendar event from Step 2 by attendee name/company and rough timing, this is what upgrades a calendar entry from "scheduled" to "confirmed, needs follow-up."

Ignore pure internal ClickUp/Close/Calendly automated notifications unless they contain a real human question in the body.

## Step 4: Update the snapshot file

Overwrite `context/pending-responses.md` with the fresh results, same section structure as above, and update the date in the `_Snapshot from Gmail scan: YYYY-MM-DD_` line at the top.

## Step 5: Follow-up sequence check

- **Only treat a call as ready for Day-1 follow-up once Step 3 confirms it via a Gemini notes / read.ai email.** A calendar event alone isn't enough, per the note in Step 2. If today's (or yesterday's, if it ran late) calendar has a non-close call type with no matching transcript email yet, don't run follow-up on it and don't flag it as a no-show either, transcripts can lag behind the call by a bit. Just note it's scheduled and awaiting confirmation, it'll get picked up on the next daily-plan run once the transcript lands.
- For calls confirmed via Step 3, note each one needs the Day-1 follow-up sequence (recap email + personal text + voice note) run per `references/sops/follow-up-masterclass.md`, and that Bryce can say "run follow-up" to have it drafted.
- If a calendar event from a day or more ago (not just today) still has no transcript, flag it once as a **likely no-show**, "no transcript found for [lead/call], worth checking if it happened" rather than silently dropping it or silently treating it as done. This is what feeds the `lead-follow-up` skill's no-show handling (confirm with Bryce, then offer to log the Close status and enroll in `Icarus No-Show Re-Engagement`), don't take any Close action from daily-plan itself, just surface it here.
- **Run the `lead-follow-up` skill's Step 5 pipeline check every time daily-plan runs**, not just when Bryce asks, specifically the **Day 2-3 touches** and **Day 3 → Day 4+ handoff** buckets. With his call volume, both are easy to lose under a heavy day, worth the extra Close.io query every morning even though the rest of this step is date-based only. Surface named leads with a day-count for each.
- Check today's date against the end of the calendar month. If today falls in the **last 7 days of the month**, flag that the end-of-month squeeze window is open (full-pipeline re-offer blast, see SOP Section 7) and ask if he wants it drafted via `lead-follow-up`. Don't assume it hasn't been sent yet, just surface the window and let Bryce confirm.

## Step 6: Check reminders and outbound asks

Read `context/waiting-on.md`. It's persistent (items stay until Bryce explicitly says they're done, not auto-cleared by a scan), so just surface what's currently there, don't try to resolve or clear anything. Two kinds of line show up in that file, treat them differently:
- **Blocked on someone else** (the normal case): already asked, waiting on their reply. Mention it lightly, don't repeat it in full every single day, that gets noisy, a short one-line mention is enough.
- **Not sent yet** (flagged explicitly in the file, e.g. "Not sent yet"): this is Bryce's own outbound action still pending, surface it as a clear reminder, not a background item, since these are easy to let slip. Once Bryce says he's sent it, update the file to drop the "not sent yet" flag and treat it as a normal blocked-on-someone-else item going forward.

## Step 7: Present the plan

Format per `.claude/rules/communication-style.md` (bullets, no em dashes, no emojis, casual internal tone). Structure:

1. **Date** (from Step 1)
2. **Today's calls** — time-ordered list from Step 2 (scheduled), marking which ones Step 3 has already confirmed happened vs. still awaiting a transcript
3. **Needs a reply today** — high-priority items from Step 3
4. **Worth a nudge** — stalled items from Step 3
5. **Confirmed calls needing follow-up** — from Step 5, calls with a transcript in hand, reminder to run follow-up on each
6. **Day 2-3 follow-up touches due** — named leads with a day-count from Step 5, don't bury this one
7. **Ready to move to the Close automation** — named leads at day 4+ from Step 5, ask before enrolling
8. **Likely no-shows** — named leads with no transcript found from Step 5, ask before logging or enrolling in re-engagement
9. **End-of-month squeeze** — only mention if the window is open per Step 5
10. **Reminders / outbound asks** — from Step 6, lead with anything flagged "not sent yet," mention normal blocked-on-someone-else items briefly
11. Skip low-priority/FYI items in the main reply unless Bryce asks for the full list — just mention "N low-priority items sitting in pending-responses.md if you want them."
