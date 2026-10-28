---
name: lemlist-claap-call-fit
description: Use when the user asks "who turned up on the calls", "did this campaign bring good calls", "reply rate vs call quality", "which campaign brings the good conversations", or wants to see whether the people a lemlist campaign booked were anywhere near the ICP.
version: 1.3.0
---

# 2. Look at who turned up on the calls

A high reply rate on a message that attracts the wrong people is a cost. This follows each campaign past the reply into the Claap calls it produced, and ranks campaigns by good calls, not replies. Tool names and the join in `CLAUDE.md`.

## Workflow

### Step 1: Inputs
1. Read `docs/icp.md` and `docs/connection.md`.
2. Campaigns named by the user win. Otherwise default: `get_campaigns` running / paused / ended, keep those with `leadMetrics.answered` ≥ 5 in `get_campaigns_stats`.
3. A named campaign with `messageMetrics.sent: 0` → "{campaign} has never sent, so no call can come from it." A named campaign with `leadMetrics.answered: 0` → "{campaign} has no replies yet, so no call can be tied to it." Do not run the join on its leads unless the user asks. None qualifies → "No campaign has 5 replies yet, there is nothing to compare." If the user wants, run a join check on one campaign (`CLAUDE.md`, "Campaign never sent") and stop there.

### Step 2: From replies to calls
1. `get_campaigns_stats` per campaign → contacted = `leadMetrics.reached`, replies = `leadMetrics.answered`, reply rate = answered / reached.
2. `search_campaign_leads` with `include: ["activities"]` (≤50 per page) → replied leads, the day of their first `emailsSent` and of their reply.
3. Reply tone: `get_inbox_conversations` (`listId: "teamConversations"`, `campaignFilter: {in: [campaignId]}`) → `get_inbox_conversation` per replied lead → `aiLeadInterest` of their reply (`positive` / `neutral` / `negative` / `not scored`).
4. Join each replied lead to Claap with the full join in `CLAUDE.md`. Prior relationships go in their own list.
5. Print the match rate per campaign before any analysis.

### Step 3: Score each attributable call
`get_recording` first (summary). `get_recording_transcript` when the summary does not answer the 5 questions, and always for the pain quote (rules in `CLAUDE.md`, "Quoting a prospect"). A `domain-match` call is scored on the person who was actually on it.

| Signal | Question |
|---|---|
| Person fit | Attendee is a buyer or user title from `docs/icp.md`? |
| Company fit | Firmographics in? |
| Pain | Did they describe the problem you solve? Quote them verbatim. |
| Next step | Concrete next step agreed (date, second call, trial, proposal)? A second call with the same company in Claap counts. |
| Why they replied | What brought them: the problem, the offer, curiosity, a freebie? |

Label: `good` (fit + pain + next step), `ok` (fit, no next step), `wrong-crowd` (no fit), `freebie-hunter` (came for the offer, no pain).

### Step 4: The funnel
| Campaign | Contacted | Replied | Positive replies | Calls | Good | OK | Wrong crowd | Freebie | **Good calls / 100 contacted** | Days reply → call (median) |
|---|---|---|---|---|---|---|---|---|---|---|

Rank by **good calls per 100 contacted**. A campaign with fewer than 5 calls is marked **directional**. Then 3 lines:
1. Best campaign on replies, best on good calls. Same campaign → say it.
2. Where each campaign leaks: the step with the biggest drop (reply → call, or call → good call).
3. Does lemlist's reply tone predict the call? Count `positive` replies that became `good` calls vs `neutral`/`negative` ones. If tone and call quality disagree, say it: your inbox triage is sorting on the wrong signal.

### Step 5: Call by call
One line per call: person, title, company, label, verbatim quote, next step, recording link. Group by campaign.

### Step 6: What to do next
One recommendation per campaign, from the numbers: keep, rewrite the offer, tighten the list (point to skill 1), or stop. Never change a campaign yourself.

### Step 7: Save `reports/call-fit-{date}.md`
Include every `good` call's verbatim pain quote and link (skill 5 reuses them).

## End state
- [ ] Match rate per campaign printed before the table
- [ ] No call counted that happened before the lead's first email
- [ ] Ranking by good calls per 100 contacted, small campaigns marked directional
- [ ] Reply tone compared with call quality
- [ ] Every `good` call has a verbatim prospect quote and its recording link
