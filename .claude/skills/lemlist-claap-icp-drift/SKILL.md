---
name: lemlist-claap-icp-drift
description: Use when the user asks "is my targeting still aligned", "check the ICP of my last campaigns", "ICP drift", "who did I actually target", or wants to compare the leads in recent lemlist campaigns against their current positioning.
version: 1.3.0
---

# 1. Check the ICP I went after

Targeting drifts one list at a time. This reads who was actually loaded into the last campaigns, scores them against `docs/icp.md`, finds the import that let the drift in, and says what it cost. Tool names in `CLAUDE.md`.

## Workflow

### Step 1: Inputs
1. Read `docs/icp.md`. Missing → run `lemlist-claap-setup`.
2. Campaigns named by the user → find them with `get_campaigns` (`search`). Otherwise ask how many recent ones (default 5) and whether drafts count (default: yes, drift starts at the list), and page `get_campaigns`.
3. `get_campaign_details` for `createdAt`, then order oldest to newest. Show date and time when two share a day.

### Step 2: Pull the leads
1. `search_campaign_leads` per campaign with `fields: ["companySize","companyIndustry","companyDomain","companyLocation","location"]`, paged with `offset` until `hasMore` is false. One call gives title, firmographics and the date each lead was added (`createdAt`). Do not call lemlist `search_contacts` or `search_companies` for this.
2. Count leads with an empty title, per campaign. Strictly over 50% in a campaign → stop and tell the user: "{n} of {m} leads have no job title in lemlist, scoring them would be guessing." Offer `bulk_enrich_leads` with the lead count and the fact that it costs credits, or continue with them as `unknown`.

### Step 3: Score each lead
- `in`: buyer or user title AND firmographics in (for the best-fitting segment if `docs/icp.md` has several)
- `adjacent`: one of the two
- `out`: neither, or matches a firmographic listed as out (freelancer, agency, too small...). A title that says it outright ("Freelance...", "Agency owner") counts as out.
- `unknown`: title missing. Never infer a title from a name, an email or a LinkedIn slug.
- Firmographics missing but title known → score on the title alone (`in*`, `adjacent*`, `out*`). Never infer size or industry from a company name. Report how many leads were title-only.
- Geography: if `docs/icp.md` names regions and the lead has `companyLocation` or `location`, check it. Otherwise say region was not checked.
- Map lemlist values to the ICP plainly ("Software Development" ≈ software; size against the employee range). ICP criteria lemlist cannot see (stage, business model) are not checked: say which ones in the report.

### Step 4: The trend
| Campaign | Created | Leads | in % | adjacent % | out % | unknown % | Titles not in the ICP |
|---|---|---|---|---|---|---|---|

Percentages are over leads with a known title, and starred scores count in their bucket (write `50% (2/4, 1 title-only)`); show `unknown %` over all leads. Then:
1. Is `in %` going up, flat or down?
2. **What crept in**, grouped by reason (agencies, too small, wrong function, junior titles, out-of-region...), with counts and the titles verbatim.

### Step 5: Where the drift came in
Leads arrive in imports: group each campaign's leads by the minute they were added (`createdAt`). For every import of 5+ leads: date, size, `in %`, `out %`. Name the import that brought the most out-of-ICP leads: "The {date} import of {n} leads brought {k} of the {m} out-of-ICP leads." That points at the list source or search that went wrong. One import per campaign → say so and skip.

### Step 6: What it cost
`search_campaign_leads` with `include: ["activities"]` on the same campaigns (≤50 per page). Per score bucket: leads contacted (`emailsSent` line), leads who replied (`emailsReplied` / `linkedinReplied`).

| Score | Leads | Contacted | Replied | Share of all replies |
|---|---|---|---|---|

Say it plainly: "{x}% of your replies came from people outside your ICP." Then list the out-of-ICP leads **not contacted yet** in running campaigns: the user can still pull them. Never remove or pause a lead yourself.

### Step 7: Was it on purpose?
Ask. Yes → offer to add the new segment to `docs/icp.md`. No → that is the drift.

### Step 8: Optional Claap cross-check
Skip campaigns with `messageMetrics.sent: 0` (say so). For `out` and `adjacent` leads who replied, run the join (`CLAUDE.md`). Out-of-ICP leads booking calls the campaign can claim → either the ICP is too narrow or the offer pulls the wrong crowd. Say what the calls suggest; the user decides.

### Step 9: Save `reports/icp-drift-{date}.md`

## End state
- [ ] Table with one row per campaign, oldest first, counts from this run
- [ ] Creeping titles named verbatim, grouped by reason
- [ ] The import that let the drift in is named (or "one import per campaign")
- [ ] The share of replies that came from outside the ICP
- [ ] No lead scored without a known title
- [ ] The user was asked whether the drift was on purpose
