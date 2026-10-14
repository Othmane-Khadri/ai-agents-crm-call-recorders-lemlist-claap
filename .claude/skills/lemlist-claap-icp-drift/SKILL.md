---
name: lemlist-claap-icp-drift
description: Use when the user asks "is my targeting still aligned", "check the ICP of my last campaigns", "ICP drift", "who did I actually target", or wants to compare the leads in recent lemlist campaigns against their current positioning.
version: 1.2.0
---

# 1. Check the ICP I went after

Targeting drifts one campaign at a time. This reads who was actually loaded into the last campaigns and scores them against `docs/icp.md`. Tool names in `CLAUDE.md`.

## Workflow

### Step 1 — Inputs
1. Read `docs/icp.md`. Missing → run `lemlist-claap-setup`.
2. Campaigns named by the user → find them with `get_campaigns` (`search`). Otherwise ask how many recent ones (default 5) and whether drafts count (default: yes, drift starts at the list), and page `get_campaigns`.
3. `get_campaign_details` for `createdAt`, then order oldest to newest. Show date and time when two share a day.

### Step 2 — Pull the leads and their titles
1. `search_campaign_leads` per campaign with `fields: ["companySize","companyIndustry","companyDomain"]`, paged with `offset` until `hasMore` is false. One call gives title and firmographics. Do not call lemlist `search_contacts` or `search_companies` for this.
2. Count leads with an empty title, per campaign. Strictly over 50% in a campaign → stop and tell the user: "{n} of {m} leads have no job title in lemlist, scoring them would be guessing." Offer `bulk_enrich_leads` with the credit cost stated, or continue with them as `unknown`.

### Step 3 — Score each lead
- `in`: buyer or user title AND firmographics in
- `adjacent`: one of the two
- `out`: neither, or matches a firmographic listed as out (freelancer, agency, too small...). A title that says it outright ("Freelance...", "Agency owner") counts as out.
- `unknown`: title missing. Never infer a title from a name, an email or a LinkedIn slug.
- Firmographics missing but title known → score on the title alone (`in*`, `adjacent*`, `out*`). Never infer size or industry from a company name. Report how many leads were title-only.
- lemlist only carries `companySize` and `companyIndustry`. Map them to the ICP plainly ("Software Development" ≈ software; size against the employee range). ICP criteria lemlist cannot see (stage, region, business model) are not checked: say which ones in the report.

### Step 4 — The trend
| Campaign | Created | Leads | in % | adjacent % | out % | unknown % | Titles not in the ICP |
|---|---|---|---|---|---|---|---|

Percentages are over leads with a known title, and starred scores count in their bucket (write `50% (2/4, 1 title-only)`); show `unknown %` over all leads. Then 3 lines:
1. Is `in %` going up, flat or down?
2. Which titles or segments crept in that are not in `docs/icp.md`? Name them.
3. Was that on purpose? Ask. Yes → offer to update `docs/icp.md`. No → that is the drift.

### Step 5 — Optional Claap cross-check
First `get_campaigns_stats`: a campaign with `messageMetrics.sent: 0` has nothing attributable: say so, skip it, and offer the join check from `CLAUDE.md` if the user wants to test the plumbing. Otherwise, for `out` and `adjacent` leads, run the join (`CLAUDE.md`). Out-of-ICP leads booking calls the campaign can claim → either the ICP is too narrow or the offer pulls the wrong crowd. The user decides which.

### Step 6 — Save `reports/icp-drift-{date}.md`

## End state
- [ ] Table with one row per campaign, oldest first, counts from this run
- [ ] Creeping titles named, verbatim
- [ ] No lead scored without a known title
- [ ] The user was asked whether the drift was on purpose
