---
name: lemlist-claap-pre-call-insight
description: Use when someone replied to a lemlist campaign or booked a call and the user says "send them something before the call", "pre-call asset", "what can I send before we meet", "warm them up before the call", or wants a useful insight drafted from calls with similar companies.
version: 1.2.0
---

# 4. Send something useful before the call

One insight and one piece of proof, from work that got results for a company like theirs, sent between the reply and the call. Draft only. Tool names in `CLAUDE.md`.

## Workflow

### Step 1 — Who replied, and what they said
1. Lead given → lemlist `search_campaign_leads` with `email` → campaign, title, and `variables` (company size, industry). If that campaign never sent, the reply did not come through lemlist: ask where it came from. Reply text: `get_inbox_conversations` with `search` = email and `listId: "teamConversations"` → `get_inbox_conversation` with the `contactId` (never `markAsRead`).
2. No lead given → list leads with a reply activity in the last 7 days (`search_campaign_leads`, `include: ["activities"]`) who have no Claap call yet (join in `CLAUDE.md`).
3. No reply text found → ask the user to paste it. Never write the email without knowing what they said.
4. Size or industry still missing → ask the user. Not lemlist `search_companies` (no firmographics).
5. Check Claap for an existing call with them (join in `CLAUDE.md`). One exists → tell the user; the asset should build on it.

### Step 2 — Find "a company like yours" in Claap
1. `search_recording_transcripts`, `type: semantic`, query = the problem in their reply, `filters.hasExternalSpeaker: true`, `limit: 30` (30 chunks, usually 10–20 calls).
2. Exclude the lead's own company. Similarity is judged from the call itself (Claap has no firmographics). Unclear → keep it, mark `similarity unknown`.
3. `get_recording` on the best 2–3. Keep 1–2 where the same problem was described AND something moved after: a result stated on a later call (`get_recordings` with that `companyId`, `createdAt.gte` = the call date) or a measured before/after on the call. Projections and promises are not results.
4. Nothing qualifies → say so, write the insight from `docs/icp.md` alone and mark it `no proof attached`.

### Step 3 — Format (ask, default to email)
- **Insight email** (default): 4–6 sentences, under 120 words.
- **One-page note**: the same, longer, Markdown they can forward.
- **Claap Slides**: only if asked. Publishing creates a page; ask first, default to password visibility.

### Step 4 — Write it
- Open with their situation, in their words from the reply.
- The proof is "a {size} {industry} team" (drop what is unknown, never guess), never the company or the person's name.
- Every number comes from a call or the user. A customer's exact figures are traceable: round them, or ask whether that customer agreed to be a reference.
- End on one thing for them to look at before the call. No calendar link, no feature list, no "just following up", no question asking for time.

### Step 5 — Hand over
Show the draft and the source calls (title, date, link). Ask: copy-paste, or send from the lemlist inbox? Do not call `send_message` without a clear yes. Save to `reports/pre-call-{lead}-{date}.md`.

## End state
- [ ] Built on the lead's actual reply text
- [ ] Proof names no other company or person
- [ ] Every number traceable to a listed call
- [ ] Nothing sent
