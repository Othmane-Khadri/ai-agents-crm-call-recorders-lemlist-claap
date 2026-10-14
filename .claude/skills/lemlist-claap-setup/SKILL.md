---
name: lemlist-claap-setup
description: Use when setting up the lemlist x Claap playbook, when the user says "check my lemlist and Claap connection", "set up the playbook", "is lemlist connected", "is Claap connected", or when docs/icp.md or docs/connection.md is missing.
version: 1.0.0
---

# lemlist x Claap — setup and connection check

Proves both MCP servers answer, proves a lemlist lead can be found in Claap, and writes the two files every other skill reads.

## When This Skill Applies
- "set up the playbook", "check my connection", "is Claap connected", "is lemlist connected"
- Any other skill in this repo finds `docs/icp.md` or `docs/connection.md` missing

## Workflow

### Step 1 — Both servers answer
1. Check the tool list for tools from `lemlist` and from `claap`. If one is missing, stop and point the user to `SETUP.md` step 3 (`/mcp` → authenticate). Do not continue with one tool, the playbook needs both.
2. Claap: call `list_workspaces`. One workspace → use it. Several → ask the user which one, by name. Never pick.
3. lemlist: list campaigns. Report how many exist and how many are running.

### Step 2 — The join works on this account
1. Pick the lemlist campaign with the most replies in the last 90 days.
2. Pull its leads who replied (lemlist activities of type `emailsReplied`, or REST `GET /api/activities?type=emailsReplied&campaignId={id}` through `call_api`).
3. For up to 20 of them, look them up in Claap: `search_contacts` with the email, then `get_recordings` / `search_recording_transcripts` for calls with that contact or company.
4. Report: `X of Y replied leads have a Claap contact, Z have at least one call`.
5. Pick the join key that worked (email, domain, or both) and write it down. If Z is 0, tell the user plainly: the Claap recorder is not on the calls these campaigns book, or attendees use different emails. Fix that before running the other skills.

### Step 3 — Write `docs/connection.md`
```markdown
# Connection
- Claap workspace: {name} ({workspaceId})
- lemlist team: {name}
- Join key that works: {email | domain | both}
- Test run {date}: {X}/{Y} replied leads in Claap, {Z} with a call
```

### Step 4 — Write `docs/icp.md` (interview, do not guess)
If it already exists, show it and ask whether it is still true. If not, ask these one at a time:
1. What do you sell, in one sentence, and to whom?
2. Which job titles sign? Which ones only use it?
3. Company size, industry, geography you go after. And the ones you refuse.
4. The 3 things that must be true for a company to be a great fit (a tool they use, a team they have, a moment they are in).
5. Your last 3 closed deals: company, title of the buyer, why they bought.
6. Your positioning line as it is on your homepage today (or give the URL and read it).

Write the answers to `docs/icp.md` with these sections: `Offer`, `Buyer titles`, `User titles`, `Firmographics (in)`, `Firmographics (out)`, `Must-be-true`, `Recent wins`, `Positioning today`. Use the user's words.

## Key File Paths
| File | Role |
|---|---|
| `docs/connection.md` | written here, read by every skill |
| `docs/icp.md` | written here, read by every skill |
