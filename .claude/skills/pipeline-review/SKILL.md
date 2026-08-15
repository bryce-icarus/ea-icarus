---
name: pipeline-review
description: Use when Bryce asks about pipeline health, forecast accuracy, stale deals, what needs a daily score, or wants a Close.io pipeline audit. Implements the daily/weekly habits from the Pipeline Management SOP. Read-only by default, only touches Close records when Bryce confirms.
---

# Pipeline Review

Implements `references/sops/pipeline-management-masterclass.md`, distilled into a published artifact ("Pipeline Management SOP") for Icarus's actual Close.io setup: one pipeline, **Sales** (`pipe_4ODy8tRPVolEiyYrRPtCwk`), stages New Lead, Qualified, Discovery Completed, Proposal Sent, Won, Lost.

**Tooling constraint:** the available Close.io tools can create pipelines, statuses, and workflows, but there is no create/update tool for custom fields or smart views. Never claim to build one, if a new field or saved view is genuinely needed, tell Bryce the exact name/type/choices to add himself in Close Settings, this takes him about two minutes.

## Step 1: Figure out what's being asked

- **"What needs a score today" / daily habit** → Step 2.
- **"Stale deals" / "what's falling through" / weekly hygiene** → Step 3.
- **"How's the pipeline looking" / forecast / win rate** → Step 4.
- **General "run the pipeline review"** → do Steps 2-4 in order.

## Step 2: Daily scoring pass

- Pull open opportunities likely to move this week. `mcp__claude_ai_Close__find_opportunities` with `status_type: "active"`, sorted `closing_soonest` or filtered by `close_at` if Bryce has real close dates set.
- There is no separate 1-10 field. The scoring habit maps onto Close's built-in **confidence** field: score x 10 = confidence percent. Present each deal with its current confidence, ask Bryce for today's score (or take it if he gives it directly), and update via `mcp__claude_ai_Close__update_opportunity` (confidence field) on confirmation. Don't set confidence values without Bryce's input, this is his call, not an inference.
- Note which of yesterday's high scores (70%+) actually moved stage, and which didn't, that's the calibration signal from the SOP. Use `mcp__claude_ai_Close__activity_search` with `activity_types: ["activity.opportunity_status_change"]` to check.

## Step 3: Stale-deal sweep

- Use `mcp__claude_ai_Close__find_opportunities` with `status_type: "active"` and `needs_attention: true` first, that's Close's own stalled-deal signal.
- Cross-check with `updated_at` filtered to more than 30 days ago for anything `needs_attention` might miss.
- Present the list by name, current stage, and days since last touch. Ask Bryce what to do with each cluster, don't auto-move stages or archive anything. Remind him per the SOP: moving a deal out of active forecasting doesn't mean stop following up, it stays in the long-term nurture cadence from `follow-up-masterclass.md`.

## Step 4: Forecast and hygiene snapshot

- Use `mcp__claude_ai_Close__aggregation` with `include_types: ["opportunity"]`, grouped by `status_id`, summing `value` and counting, to get pipeline value and count by stage.
- Flag hygiene gaps plainly rather than presenting a forecast as if it's reliable: e.g. if most opportunities have `value: 0` or flat/default confidence, say so and note the forecast isn't meaningful until that's fixed, don't let a technically-correct-but-garbage-in number pass as a real forecast.
- **The known backlog** (as of 2026-08-14): 368 active opportunities from a bulk import, all sampled records sitting in New Lead with $0 value and flat 50% confidence, real details buried in free-text notes instead of structured fields. Mention this is still open until Bryce says otherwise, and offer to help triage it in batches (e.g. "want me to pull the next 20 and help you go through them") rather than trying to fix it in one silent pass. This is real deal data, treat it with the same care as the earlier decision not to bulk-modify leads without review.

## Step 5: Present the summary

Per `.claude/rules/communication-style.md` (bullets, no em dashes, no emojis, casual internal tone):

1. **Today's scoring** (if Step 2 ran) — deals asked about, scores set.
2. **Stale deals** (if Step 3 ran) — named, with days since touch, grouped if there's a clear pattern (e.g. "6 of these are from the same import batch").
3. **Pipeline snapshot** (if Step 4 ran) — value/count by stage, with the hygiene caveat front and center if the data isn't clean yet.
4. Any Close-side gap that needs manual setup (custom field, smart view) gets called out explicitly with the exact spec to create it, not glossed over.
