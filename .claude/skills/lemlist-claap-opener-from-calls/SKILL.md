---
name: lemlist-claap-opener-from-calls
description: Use when the user asks "write my next opener from my calls", "first line from call transcripts", "use buyer language in my campaign", "what words do my buyers use", or wants the next lemlist campaign's opening line built from Claap calls that went somewhere.
version: 1.2.0
---

# 5. Write the next campaign's first line from the calls that went somewhere

The best opener is a sentence a buyer already said out loud. Tool names in `CLAUDE.md`.

## Workflow

### Step 1 — Which calls count
"Went somewhere" = a prospect call (not a client, partner or internal call) that led to a second call with the same company, a deal moving forward, or an agreed next step in the summary. Sources, in order:
1. Latest `reports/call-fit-*.md` (if `reports/` exists): its `good` calls.
2. Claap `search_deals` with `filters: {}` sorted by `lastInteraction` → keep deals whose stage the user calls won or advancing (ask, stage names differ per CRM) → `get_recordings` with `filters.dealId`. Skip if there are 0 deals (no CRM).
3. Claap `get_recordings` with `hasExternalSpeaker: true` and `createdAt.gte` = 6 months ago, `sort: [{field: "createdAt", order: "desc"}]`, `limit: 100`. If `total` > 100, split the date range into smaller windows instead of following `nextCursor` (it can return an empty page). Then filter:
   - drop calls with no company, or where every external attendee is on a free-mailbox domain: these are usually team or internal calls
   - group by company **name and domain** (Claap can hold the same company under two IDs), keep companies with 2+ calls
   - show that list (company, number of calls, first and last date, titles) and **ask the user to untick clients, partners and anyone who is not a prospect**. Do not guess it from titles.

If the user names a company that is not on the list, look it up (`get_recordings` with `companyName`) and add it if its calls qualify.

Fewer than 5 calls left → stop: "{n} calls is an anecdote, not a pattern."

### Step 2 — Mine the prospect's words
1. Write the query from the `Problem` section of `docs/icp.md` (no such section → use `Offer`, and suggest re-running setup).
2. `search_recording_transcripts`, `type: semantic`, `filters.recordingId` = the chosen calls, `limit: 100`. Do not filter on `tag`: the Claap tags are often empty and the filter then returns nothing.
3. A chosen call with no chunks, or whose chunks hold no prospect line → read it with `get_recording_transcript`. Semantic search can drop calls silently.
4. Keep only what the **prospect** said. Chunks mix speakers: take `speakers[].isExternal` from the `get_recordings` results you already have, drop internal speakers, and drop lines attributed to `Unknown`. A team member on a personal email can show as external, so ask the user for their team's names once and cross-check.
5. Per call, extract: the problem, the moment it hurt, what they had tried, the outcome they wanted.
6. Transcript in another language than the campaign → keep the original quote, add your translation, and mark it `translated`. The translation is not verbatim.

| Theme | Companies (of n) | Strongest quote, verbatim | Recording (+ timestamp when the search chunk gives one) |
|---|---|---|---|

Rank themes by the number of different companies, then by the most recent call.

### Step 3 — Compare with what you send today
Ask which campaign the opener is for (these calls often came from referrals, not from a campaign). `get_campaigns` to confirm it by name, then `get_campaign_sequences` → current first email. Show its first line next to the top theme, and point out where your words differ from theirs.

### Step 4 — Write 5 openers
One per top theme, 2 for the strongest. Each one:
- reuses the buyer's wording, never their identity, company or a detail specific enough to recognise them (a number of accounts, a named tool they built)
- is about the reader, under 25 words, in the campaign's language
- no "hope this finds you well", no feature, does not open with a question
- carries a note: theme, number of companies, source quote

### Step 5 — Optional: put one in lemlist
Only if the user picks one and says so:
1. `get_campaign_sequences` → `sequenceId` and the first email step's `stepId`.
2. Build variant B = variant A's body with **only the opening line replaced** by the new opener. Keep greeting, call to action and sign-off. Keep the subject unless asked.
3. Show the full variant B body and the campaign name. If the step already has a variant B, show it too and say it will be replaced. Get a yes, then `set_ab_variant`. Never overwrite variant A, never start the campaign.
4. Re-read with `get_campaign_sequences` and show variant B as saved.

### Step 6 — Save `reports/openers-{date}.md`

## End state
- [ ] At least 5 prospect calls, confirmed by the user, each with its link
- [ ] Every quote from the prospect, verbatim or marked `translated`
- [ ] 5 openers, each traced to a theme and a quote, no identifying detail
- [ ] lemlist only changed as variant B (first line swapped), after a yes, verified by re-reading
