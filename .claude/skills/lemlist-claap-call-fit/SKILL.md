---
name: lemlist-claap-call-fit
description: Use when the user asks "who turned up on the calls", "did this campaign bring good calls", "reply rate vs call quality", "which campaign brings the good conversations", or wants to see whether the people a lemlist campaign booked were anywhere near the ICP.
version: 1.0.0
---

# 2. Look at who turned up on the calls

A good reply rate on a message that attracts the wrong people is a cost. This skill follows a campaign past the reply, into the Claap calls it produced, and scores those calls.

## When This Skill Applies
"who showed up on the calls from campaign X", "call quality by campaign", "which campaign brings good conversations", "replies vs calls"

## Workflow

### Step 1 — Inputs
1. Read `docs/icp.md` and `docs/connection.md`.
2. Ask which campaigns (default: every campaign with at least 5 replies in the last 90 days).

### Step 2 — From replies to calls
1. Per campaign, pull leads who replied (activities `emailsReplied` / `linkedinReplied`, or `GET /api/activities?type=emailsReplied&campaignId={id}`).
2. Find each one's calls in Claap using the join in `CLAUDE.md` (email → domain, after first email, within 60 days of the reply). Tools: `search_contacts`, `search_companies`, `get_recordings`, `get_recording`.
3. Report the match rate per campaign before anything else.

### Step 3 — Score each call (read the call, not the title)
For each matched call, `get_recording` with the summary, and `get_recording_transcript` when the summary is thin. Score:

| Signal | Question |
|---|---|
| Person fit | Is the attendee a buyer or user title from `docs/icp.md`? |
| Company fit | Firmographics in? |
| Pain | Did they describe the problem you solve, in their own words? Quote it. |
| Next step | Was a concrete next step agreed (date, second call, trial, proposal)? |
| Why they replied | What did they say brought them? (the offer, the problem, curiosity, a freebie) |

Call label: `good` (person + company fit + pain + next step), `ok` (fit, no next step), `wrong-crowd` (no fit), `freebie-hunter` (came for the offer, no pain).

### Step 4 — The comparison that matters
| Campaign | Reply rate | Replies | Calls found | good | ok | wrong-crowd | freebie-hunter | Good calls per 100 leads |
|---|---|---|---|---|---|---|---|---|

Rank by **good calls per 100 leads**, not by reply rate. Then say in 2 lines which campaign looks best on replies and which one is actually best on calls. If they are the same campaign, say that too.

### Step 5 — Save
`reports/call-fit-{date}.md`: the table, the 2 lines, and one quoted line per `good` call showing the pain in the buyer's words (skill 5 reuses these).

## Key File Paths
| File | Role |
|---|---|
| `docs/icp.md`, `docs/connection.md` | inputs |
| `reports/call-fit-{date}.md` | output, read by `lemlist-claap-opener-from-calls` |
