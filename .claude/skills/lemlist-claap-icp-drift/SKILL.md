---
name: lemlist-claap-icp-drift
description: Use when the user asks "is my targeting still aligned", "check the ICP of my last campaigns", "ICP drift", "who did I actually target", or wants to compare the leads in recent lemlist campaigns against their current positioning.
version: 1.0.0
---

# 1. Check the ICP I went after

Targeting drifts one campaign at a time and nobody notices while it happens. This skill reads who was actually loaded into the last campaigns and scores them against `docs/icp.md`.

## When This Skill Applies
"check my ICP", "ICP drift", "who am I really targeting", "are my campaigns aligned with my positioning"

## Workflow

### Step 1 — Inputs
1. Read `docs/icp.md` and `docs/connection.md`. Missing → run `lemlist-claap-setup` first.
2. Ask: how many recent campaigns (default: the last 5 launched), and should paused ones count?

### Step 2 — Pull the leads that were loaded
For each campaign, list its leads (lemlist leads tool, or `GET /api/campaigns/{id}/export/leads` via `call_api`). Keep: title, company, company domain, industry, company size, location, plus any custom variable that looks like a segment tag.

### Step 3 — Score each lead against the ICP
Per lead, one label:
- `in` — buyer or user title AND firmographics in
- `adjacent` — one of the two matches
- `out` — neither, or hits a firmographic listed as out

Judge only on fields that exist. Missing title or size → `unknown`, never a guess.

### Step 4 — Read the trend, not the snapshot
Table per campaign, oldest to newest:

| Campaign | Launched | Leads | in % | adjacent % | out % | unknown % | Top 3 titles |
|---|---|---|---|---|---|---|---|

Then answer in 3 lines:
1. Is `in %` going up, flat, or down across the campaigns?
2. Which title or segment crept in that is not in `docs/icp.md`?
3. Does that creep match a positioning change the user made on purpose? Ask them. If yes, offer to update `docs/icp.md`. If no, that is the drift.

### Step 5 — Optional Claap cross-check
For the `out` and `adjacent` leads who replied, check Claap for calls (join rules in `CLAUDE.md`). If out-of-ICP replies are booking calls, show it: either the ICP is too narrow or the offer pulls the wrong crowd. Let the user decide which.

### Step 6 — Save
Write `reports/icp-drift-{date}.md` with the table, the 3-line read, and the list of creeping titles.

## Key File Paths
| File | Role |
|---|---|
| `docs/icp.md` | the reference |
| `reports/icp-drift-{date}.md` | output |
