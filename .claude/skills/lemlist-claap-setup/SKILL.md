---
name: lemlist-claap-setup
description: Use when setting up the lemlist x Claap playbook, when the user says "check my lemlist and Claap connection", "set up the playbook", "is lemlist connected", "is Claap connected", or when docs/icp.md or docs/connection.md is missing.
version: 1.2.0
---

# lemlist x Claap — setup and connection check

Proves both servers answer, proves a lemlist lead can be found in Claap, and writes the two files every other skill reads. Tool names and the join are in `CLAUDE.md`.

## Workflow

### Step 1 — Both servers answer
1. Look for lemlist and Claap tools under either prefix (see `CLAUDE.md`, "Finding the tools"). One missing → stop, point to `SETUP.md` step 3. The playbook needs both.
2. Claap `list_workspaces`. One → use it. Several → ask which, by name.
3. lemlist `get_campaigns` (no status filter), paged until you have `totalRecords`. Report the real total and how many draft / running / paused / ended. `get_team_info` for the team name.

### Step 2 — Pick the test campaign
1. `get_campaigns_stats` on every non-draft campaign (in batches). Pick the one with the highest `leadMetrics.answered`.
2. No campaign has `messageMetrics.sent` above 0 → say: "Nothing has been sent from this lemlist team yet, so I can only check the plumbing, not results." Ask which campaign to use for the join check.

### Step 3 — Run the join
1. `search_campaign_leads` on that campaign with `include: ["activities"]`. Take the leads who replied; if there are none, take every lead (join check mode, say so).
2. For up to 20 of them, run the join from `CLAUDE.md`: junk-domain skip, email, domain (by company ID), time window.
3. Report three numbers: `found in Claap (email)`, `found in Claap (domain only)`, `attributable to the campaign`. List prior relationships separately.
4. Diagnose with exactly one of these verdicts:
   - **Ready**: attributable ≥ 50%.
   - **Plumbing works, no results yet**: found in Claap but nothing attributable, campaign never sent. Normal for a draft.
   - **Calls not linked to campaigns**: campaign sent, leads found in Claap, but the calls predate the emails or fall outside 60 days. Check that replies book calls the Claap recorder joins.
   - **Not found**: nobody found in Claap. Cause 1: the Claap recorder is not on these calls. Cause 2: prospects book with another email. Cause 3: calls are on a calendar Claap does not record.

### Step 4 — Write `docs/connection.md`
```markdown
# Connection
- Claap workspace: {name} ({workspaceId})
- lemlist team: {name}
- Campaigns: {n} ({draft} draft, {running} running, {paused} paused, {ended} ended)
- Join key that works: {email | domain | both | none found} ("works" = finds the person in Claap, attributable or not)
- Verdict: {Ready | Plumbing works, no results yet | Calls not linked to campaigns | Not found}
- Test {date} on "{campaign}": {email hits}/{n} by email, {domain hits} by domain, {attributable} attributable
```

### Step 5 — Write `docs/icp.md` (interview, never guess)
Exists → show it, ask if it is still true. Otherwise ask one at a time:
1. What do you sell, in one sentence, and to whom?
2. Which job titles sign? Which only use it?
3. Company size, industry, geography you go after. And the ones you refuse.
4. 3 things that must be true for a great fit (a tool, a team, a moment).
5. Your last 3 won deals: company, buyer title, why they bought.
6. Your positioning line on your homepage today (or the URL, and read it).
7. The problem buyers describe, in their words: what were they saying on the call right before they bought?

Sections, in the user's words: `Offer`, `Problem`, `Buyer titles`, `User titles`, `Firmographics (in)`, `Firmographics (out)`, `Must-be-true`, `Recent wins`, `Positioning today`.

## End state (check before finishing)
- [ ] `docs/connection.md` exists with real counts from this run
- [ ] `docs/icp.md` exists with all 9 sections
- [ ] The user got one of the 4 verdicts, in plain words

No report file for this skill: `docs/connection.md` is its output.
