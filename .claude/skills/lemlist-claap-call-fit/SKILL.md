---
name: lemlist-claap-call-fit
description: Use when the user asks "who turned up on the calls", "did this campaign bring good calls", "reply rate vs call quality", "which campaign brings the good conversations", or wants to see whether the people a lemlist campaign booked were anywhere near the ICP.
version: 1.2.0
---

# 2. Look at who turned up on the calls

A high reply rate on a message that attracts the wrong people is a cost. This follows each campaign past the reply into the Claap calls it produced. Tool names and the join in `CLAUDE.md`.

## Workflow

### Step 1 — Inputs
1. Read `docs/icp.md` and `docs/connection.md`.
2. Campaigns named by the user win. Otherwise default: `get_campaigns` running / paused / ended, keep those with `leadMetrics.answered` ≥ 5 in `get_campaigns_stats`.
3. A named campaign with `messageMetrics.sent: 0` → "{campaign} has never sent, so no call can come from it." None qualifies → say "No campaign has 5 replies yet, there is nothing to compare." If the user wants, run a join check on one campaign (see `CLAUDE.md`, "Campaign never sent") using the join labels and match-rate format from `CLAUDE.md`, and stop there.

### Step 2 — From replies to calls
1. `get_campaigns_stats` per campaign → contacted = `leadMetrics.reached`, replies = `leadMetrics.answered`, reply rate = answered / reached.
2. `search_campaign_leads` with `include: ["activities"]` (≤50 per page) → replied leads, the date of their first `emailsSent` and of their reply.
3. Join each replied lead to Claap with the full join in `CLAUDE.md` (junk-domain skip, email, company ID, paging, time window). Prior relationships go in their own list.
4. Print the match rate per campaign before any analysis.

### Step 3 — Score each attributable call
`get_recording` first (summary). `get_recording_transcript` when the summary does not answer the 5 questions, and always for the pain quote. A `domain-match` call is scored on the person who was actually on it:

| Signal | Question |
|---|---|
| Person fit | Attendee is a buyer or user title from `docs/icp.md`? |
| Company fit | Firmographics in? |
| Pain | Did they describe the problem you solve? Quote them verbatim from the transcript, never from the summary. |
| Next step | Concrete next step agreed (date, second call, trial, proposal)? |
| Why they replied | What brought them: the problem, the offer, curiosity, a freebie? |

Label: `good` (fit + pain + next step), `ok` (fit, no next step), `wrong-crowd` (no fit), `freebie-hunter` (came for the offer, no pain).

### Step 4 — The comparison
| Campaign | Contacted | Reply rate | Replies | Attributable calls | good | ok | wrong-crowd | freebie-hunter | Good calls / 100 contacted |
|---|---|---|---|---|---|---|---|---|---|

Rank by **good calls per 100 contacted**. 2 lines: best campaign on replies, best on calls. Same campaign → say it.

### Step 5 — Save `reports/call-fit-{date}.md`
Include one verbatim pain quote per `good` call (skill 5 reuses them).

## End state
- [ ] Match rate per campaign printed before the table
- [ ] No call counted that happened before the lead's first email
- [ ] Ranking by good calls per 100 contacted, not reply rate
- [ ] Every `good` call has a verbatim quote and its recording link
