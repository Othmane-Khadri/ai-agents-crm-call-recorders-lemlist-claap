---
name: lemlist-claap-outlier-check
description: Use when the user asks "was this deal an outlier", "should I keep going after this list", "do I need a new list", "what did my closed deals have in common", or wants to compare a won deal against the rest of the lemlist campaign that sourced it.
version: 1.0.0
---

# 3. Check whether the deal I closed was an outlier

You closed a deal from a campaign. Either that buyer shares a context with the rest of the list (keep going), or they were a lucky outlier (build a new list). This skill decides which.

## When This Skill Applies
"was this deal an outlier", "keep the list or rebuild it", "what do my wins have in common", "deal X came from campaign Y, now what"

## Workflow

### Step 1 — Find the won deal and its campaign
1. Ask which deal, or list recent won deals: Claap `search_deals` filtered on a closed-won stage (`list_deal_stages` gives the stage IDs; deals need a connected CRM, HubSpot today). If no CRM is connected, ask the user for the company name instead.
2. Find the buyer's email from the deal's contacts, then find which lemlist campaign they were in (search the lead by email in lemlist). No campaign → this deal did not come from outbound, stop and say so.

### Step 2 — Build the winner's context (from the calls, not the CRM)
Read the deal's Claap calls (`get_deal` → recordings, `get_recording`, `get_recording_transcript`). Extract, with a quote for each:
- **Trigger** — why now (new hire, funding, tool change, a failed attempt, a deadline)
- **Stack** — tools they named
- **Team** — who owns the problem, team size
- **Pain** — the problem in their words
- **Buyer** — title and seniority of who signed

### Step 3 — Check the rest of the campaign for the same context
Pull the campaign's other leads. For each, check only what can be checked from data you have: title, company size, industry, plus a website or LinkedIn look-up for the trigger and stack if the user agrees to the cost (ask first, give the count of look-ups).

| Context element | Winner | Share of the campaign that matches |
|---|---|---|

### Step 4 — Verdict
- **Shared** (most context elements match a meaningful share of the list): the win is repeatable. Recommend continuing on the not-yet-contacted part of the list, and name the 1–2 elements to lead with.
- **Outlier** (the winner's trigger or stack is rare in the list): the list is wrong for this win. Write the new list definition: the filters that would find more companies like the winner, in plain words, ready to paste into a lead search.

Never call it shared or outlier on fewer than 2 checkable elements. Say `not enough data` instead.

### Step 5 — Save
`reports/outlier-{deal}-{date}.md`.

## Key File Paths
| File | Role |
|---|---|
| `docs/icp.md` | compare the winner against the stated ICP too: wins outside it are worth flagging |
| `reports/outlier-{deal}-{date}.md` | output |
