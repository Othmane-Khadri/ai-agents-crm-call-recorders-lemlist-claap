---
name: lemlist-claap-pre-call-insight
description: Use when someone replied to a lemlist campaign or booked a call and the user says "send them something before the call", "pre-call asset", "what can I send before we meet", "warm them up before the call", or wants a useful insight drafted from calls with similar companies.
version: 1.0.0
---

# 4. Send something useful before the call

Not a pitch. One insight and one piece of proof, from work that got results for a company like theirs, sent between the reply and the call. Draft only, the user sends it.

## When This Skill Applies
"send them something before the call", "pre-call asset for {lead}", "they replied, what do I send", "prep a pre-call email"

## Workflow

### Step 1 — Who replied
Ask for the lead (email or name), or list leads who replied in the last 7 days and have no Claap call yet (lemlist reply activities, joined to Claap per `CLAUDE.md`). For the chosen lead, read: their reply text, title, company, the campaign and the step they replied to.

### Step 2 — Find "a company like yours" in Claap
Search past calls for look-alikes, in this order:
1. `search_recording_transcripts` (semantic) with the lead's industry + the problem their reply mentions
2. `search_companies` for companies of similar size and industry, then their recordings
3. Won deals (`search_deals`) at similar companies

Pick 1 or 2 calls where a similar company described the same problem AND something moved after (deal advanced, result mentioned). Extract the concrete result and the exact words.

**Never name the other company or quote a person in the asset** unless the user confirms that customer agreed to be a reference. Default: "a {size} {industry} team".

### Step 3 — Choose the format (ask the user, give a default)
- **Insight email** (default): 4–6 sentences. One observation about their situation, one result from the look-alike, one thing to look at before the call. No calendar link, no feature list, no "just following up".
- **One-page note**: the same, longer, as a Markdown doc they can forward internally.
- **Claap Slides**: only if the user asks. Publishing creates a page on Claap (`publish_artifact_code_as_is`); ask before publishing and default to password visibility.

### Step 4 — Write it
Rules:
- Open with their situation, not with you.
- Every number comes from a Claap call or the user. No invented stats.
- End on something for them to check, not a question asking for more time.
- Under 120 words for the email.

### Step 5 — Hand over
Show the draft and the source calls (titles + dates) it was built from. Ask: send from lemlist as a manual step, or copy-paste? Do not send anything without a clear yes. Save to `reports/pre-call-{lead}-{date}.md`.

## Key File Paths
| File | Role |
|---|---|
| `docs/icp.md` | tone and offer |
| `reports/pre-call-{lead}-{date}.md` | output |
