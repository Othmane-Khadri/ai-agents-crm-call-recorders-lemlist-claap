---
name: lemlist-claap-outlier-check
description: Use when the user asks "was this deal an outlier", "should I keep going after this list", "do I need a new list", "what did my closed deals have in common", or wants to compare a won deal against the rest of the lemlist campaign that sourced it.
version: 1.2.0
---

# 3. Check whether the deal I closed was an outlier

Either the buyer shares a context with the rest of the list (keep going) or they were a lucky outlier (build a new list). Tool names in `CLAUDE.md`.

## Workflow

### Step 1 — The won deal and its campaign
1. Ask which deal. To list candidates: Claap `search_deals` with `filters: {stage: "won"}`. 0 deals, or `list_deal_stages` errors "Crm not connected" → no CRM is connected to Claap; ask the user for the company and buyer email.
2. lemlist `search_campaign_leads` with `email` = the buyer's email → which campaign(s) they were in. Not in any → "This deal did not come from lemlist outbound", stop.
3. Several campaigns → ask which one sourced it.
4. **Did the campaign actually reach them?** `search_campaign_leads` with `email` and `include: ["activities"]` → the buyer's first `emailsSent` in that campaign. None, or it is after the first Claap call with them → "{company} was not sourced by {campaign}: {it never emailed them | they were already talking to you}." Stop: a list cannot be judged by a deal it did not produce. Offer to run Steps 2–4 anyway as a lookalike check, labelled as such.

### Step 2 — The winner's context, from the calls
Find the calls: `get_recordings` with `filters.contactEmail` (`filters.dealId` only if the deal exists in Claap). `get_recording` to locate what matters, `get_recording_transcript` for quotes: summaries are paraphrase, never quote them. Extract, each with a verbatim quote and the recording link:
- **Trigger**: why now (hire, funding, tool change, failed attempt, deadline)
- **Stack**: tools they named
- **Team**: who owns the problem, team size
- **Pain**: in their words
- **Buyer**: title and seniority

Missing element → `not said on the calls`. Never fill it from general knowledge about the company.

### Step 3 — Does the rest of the list share it?
1. `search_campaign_leads` on that campaign with `fields: ["companySize","companyIndustry","companyDomain"]`: title and firmographics in one call.
2. Check only what the data shows: title, seniority, company size, industry. Empty field → `unknown` for that lead; shares are over leads where the field is known. Trigger and stack need outside research: ask before doing it and give the number of look-ups.

| Context element | Winner | Leads that match | Share |
|---|---|---|---|

### Step 4 — Verdict
Count only elements known for at least half the list.
- **Not enough data**: fewer than 2 such elements. Say which data is missing and how to get it.
- **Shared**: 2+ elements, each matching 30%+ of the list. Keep going on the not-yet-contacted leads, lead with the shared elements.
- **Outlier**: 2+ elements, and the trigger or stack (if unknown: most known elements) matches under 30%. Write the new list definition in plain filters, ready to paste into a lead search.
- Anything else → **Mixed**: say which elements are shared and which are rare, the user decides.

Also flag it if the winner is outside `docs/icp.md`: a win outside the ICP is a positioning question.

### Step 5 — Save `reports/outlier-{company}-{date}.md`

## End state
- [ ] Deal traced to exactly one campaign (or stopped with the reason)
- [ ] Every context element has a quote and a link, or says `not said on the calls`
- [ ] Attribution checked before any comparison (Step 1.4)
- [ ] Verdict is one of shared / outlier / mixed / not enough data, with the counts behind it
