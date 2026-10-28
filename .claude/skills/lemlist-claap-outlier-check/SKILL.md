---
name: lemlist-claap-outlier-check
description: Use when the user asks "was this deal an outlier", "should I keep going after this list", "do I need a new list", "what did my closed deals have in common", or wants to compare a won deal against the rest of the lemlist campaign that sourced it.
version: 1.3.0
---

# 3. Check whether the deal I closed was an outlier

Either the buyer shares a context with the rest of the list (keep going, and here is who to contact first) or they were an outlier (here is the next list, already sized). Tool names in `CLAUDE.md`.

## Workflow

### Step 1: The won deal and its campaign
1. Ask which deal, or deals: 2-3 wins from the same campaign give a pattern, one gives a hint. To list candidates: Claap `search_deals` with `filters: {stage: "won"}`. 0 deals, or `list_deal_stages` errors "Crm not connected" → no CRM is connected to Claap; ask the user for the company and buyer email.
2. lemlist `search_campaign_leads` with `email` = the buyer's email → the campaign(s) they were in, and their `createdAt`. Not in any → "This deal did not come from lemlist outbound", stop.
3. Several campaigns → ask which one sourced it.
4. **Did the campaign actually reach them?** The email lookup returns no activities: use "One lead's activities" in `CLAUDE.md` to get the buyer's first `emailsSent` line. None, or it is after the first Claap call with them → "{company} was not sourced by {campaign}: {it never emailed them | they were already talking to you}." Stop: a list cannot be judged by a deal it did not produce. Offer to run Steps 2-5 anyway as a lookalike check, labelled as such.

### Step 2: The winner's context, from the calls
`get_recordings` with `filters.contactEmail` and `limit: 100` (`filters.dealId` only if the deal exists in Claap). `get_recording` to locate what matters, `get_recording_transcript` for quotes (`CLAUDE.md`, "Quoting a prospect"). Extract, each with a verbatim quote and the recording link:
- **Trigger**: why now (hire, funding, tool change, failed attempt, deadline)
- **Stack**: tools they named
- **Team**: who owns the problem, team size
- **Pain**: in their words
- **Buyer**: title and seniority
- **Result**: anything measured on a later call (before/after). Skill 4 reuses it as proof.

Missing element → `not said on the calls`. Never fill it from general knowledge about the company.

### Step 3: Does the rest of the list share it?
1. `search_campaign_leads` on that campaign with `fields: ["companySize","companyIndustry","companyDomain","companyLocation","companyTechnologies","companyEmployeeCount"]`.
2. Compare on what the data shows: title family, seniority, company size, industry, and **stack** (`companyTechnologies`) and **region** (`companyLocation`) when filled. Empty field → `unknown` for that lead; shares are over leads where the field is known.
3. Trigger needs outside research (job posts, leadership changes, funding): ask before doing it and give the number of look-ups.

| Context element | Winner | Leads that match | Share |
|---|---|---|---|

### Step 4: Verdict
Count only elements known for at least half the list.
- **Not enough data**: fewer than 2 such elements. Say which data is missing and how to get it.
- **Shared**: 2+ elements, each matching 30%+ of the list.
- **Outlier**: 2+ elements, and the trigger or stack (if unknown: most known elements) matches under 30%.
- Anything else → **Mixed**: say which elements are shared and which are rare, the user decides.

One deal → write **directional** next to the verdict. Also flag it if the winner is outside `docs/icp.md`: a win outside the ICP is a positioning question.

### Step 5: What to do with it
- **Shared** → rank the leads of that campaign who have not replied yet by how many winner elements they match. Show the top 10 (name, title, company, elements matched) as "contact these first", and suggest a follow-up that leads with the shared elements.
- **Outlier** → write the next list in plain filters (titles, industry, size, region, stack, trigger, exclusions). Then size it: `get_lemleads_filters` for the filter IDs, `people_database_search_count` for the number of people who match (a count, no leads pulled, no credits). Say if it is too small (under 200) or too broad (over 20,000) and which filter to change.
- **Mixed** → both, smaller.
- Offer to add the winner's profile to `docs/icp.md` as a new segment (or to `Recent wins`), so the next ICP check scores against it. Write only after a yes.

### Step 6: Save `reports/outlier-{company}-{date}.md`

## End state
- [ ] Deal traced to exactly one campaign (or stopped with the reason)
- [ ] Attribution checked with the buyer's real first email date before any comparison
- [ ] Every context element has a quote and a link, or says `not said on the calls`
- [ ] Verdict is one of shared / outlier / mixed / not enough data, with the counts behind it, marked directional if built on one deal
- [ ] A next action: the leads to contact first, or the next list with its size
