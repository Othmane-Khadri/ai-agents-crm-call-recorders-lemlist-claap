---
name: lemlist-claap-opener-from-calls
description: Use when the user asks "write my next opener from my calls", "first line from call transcripts", "use buyer language in my campaign", "what words do my buyers use", or wants the next lemlist campaign's opening line built from Claap calls that went somewhere.
version: 1.0.0
---

# 5. Write the next campaign's first line from the calls that went somewhere

The best opener is a sentence a buyer already said out loud. This skill pulls the words people used on calls that moved forward and turns them into first lines.

## When This Skill Applies
"next opener from my calls", "buyer language", "rewrite my first line", "what do my good calls say", "new campaign copy from Claap"

## Workflow

### Step 1 — Which calls count
"Went somewhere" means at least one of: deal advanced a stage, a second meeting happened, the call summary records an agreed next step. Sources, in order:
1. `reports/call-fit-*.md` (latest) — the `good` calls and their quotes
2. Claap won or advancing deals (`search_deals`) → their recordings
3. Ask the user to name 5–10 calls

Need at least 5 calls. Fewer → say so and stop, 2 calls is an anecdote.

### Step 2 — Mine the words
From each transcript (`get_recording_transcript`), pull only what the **prospect** said, never the rep:
- the problem, in their words
- the moment it hurt (when it happened, what it cost)
- what they had tried before
- the phrase they used to describe the outcome they wanted

Group the quotes by theme. Count how many calls each theme appears in.

| Theme | Calls | Strongest quote (verbatim) |
|---|---|---|

### Step 3 — Compare with what you sent
Pull the first step of the campaigns these calls came from (lemlist campaign sequence). Show the current first line next to the top theme. Note where the opener uses your words and the calls use different ones.

### Step 4 — Write 5 openers
One per top theme (or 2 for the strongest). Each:
- is built on a real quote's wording, adapted, not copied with the person's identity
- is about them, under 25 words
- has no "I hope this finds you well", no feature, no question as the first sentence
- has a note: which theme, how many calls, which quote it came from

### Step 5 — Optional: put it in lemlist
Only if the user picks one and says so: add it as a new variant (A/B) on the next campaign's first step through the lemlist tools, as a draft. Never replace a live step, never start a campaign. Confirm the campaign name before writing.

### Step 6 — Save
`reports/openers-{date}.md` with the theme table, the 5 openers and their sources.

## Key File Paths
| File | Role |
|---|---|
| `reports/call-fit-*.md` | preferred input (good calls + quotes) |
| `reports/openers-{date}.md` | output |
