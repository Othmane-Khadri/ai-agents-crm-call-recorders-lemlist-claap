---
name: lemlist-claap-pre-call-insight
description: Use when someone replied to a lemlist campaign or booked a call and the user says "send them something before the call", "pre-call asset", "what can I send before we meet", "warm them up before the call", or wants a useful insight drafted from calls with similar companies.
version: 1.3.0
---

# 4. Send something useful before the call

One insight and one piece of proof, from work that got results for a company like theirs, sent between the reply and the call. Plus a short prep note for you. Draft only. Tool names in `CLAUDE.md`.

## Workflow

### Step 1: Who replied, and what they said
1. Lead given → lemlist `search_campaign_leads` with `email` → campaign, `variables` (title, company size, industry). If that campaign never sent, the reply did not come through lemlist: ask where it came from.
2. Reply text: `get_inbox_conversations` with `search` = email and `listId: "teamConversations"` → `get_inbox_conversation` with the `contactId` (never `markAsRead`). Note `aiLeadInterest` on the reply.
3. No lead given → `get_inbox_conversations` (`listId: "teamConversations"`, `dateFilter` = last 7 days) → keep contacts with a reply and no Claap call yet (join in `CLAUDE.md`). Ask which one.
4. No reply text found → ask the user to paste it. Never write the email without knowing what they said.
5. **Read the reply for three things:** the problem in their words, any question they asked, and their tone. Tone `negative`, or the reply is a "not now" → stop: a pre-call asset to someone who said no is spam. Offer a two-line, no-pressure close instead.
6. Size or industry still missing → ask the user. Not lemlist `search_companies` (no firmographics).
7. Check Claap for an existing call with them (join in `CLAUDE.md`). One exists → tell the user; the asset should build on it.

### Step 2: Find "a company like yours" in Claap
1. `search_recording_transcripts`, `type: semantic`, query = the problem in their reply, `filters.hasExternalSpeaker: true`, `limit: 30` (30 chunks, usually 10-20 calls).
2. Exclude the lead's own company. Similarity is judged from the call itself (Claap has no firmographics). Unclear → keep it, mark `similarity unknown`.
3. `get_recording` on the best 2-3. Keep 1-2 where the same problem was described AND something moved after: a result stated on a later call (`get_recordings` with that `companyId`, `limit: 100`, later dates) or a measured before/after on the call. Projections and promises are not results.
4. Nothing qualifies → say so, write the insight from `docs/icp.md` alone and mark it `no proof attached`.

### Step 3: Format (ask, default to email)
- **Insight email** (default): 4-6 sentences, under 120 words.
- **One-page note**: the same, longer, Markdown they can forward.
- **Claap Slides**: only if asked. Publishing creates a page; ask first, default to password visibility.

### Step 4: Write it
- Open with their situation, in their words from the reply.
- **If they asked a question, answer it** in one or two sentences. That is what they are waiting for.
- The proof is "a {size} {industry} team" (drop what is unknown, never guess), never the company or the person's name.
- Every number comes from a call or the user. A customer's exact figures are traceable: round them, or ask whether that customer agreed to be a reference.
- End on one thing for them to look at or measure before the call. No calendar link, no feature list, no "just following up", no question asking for time.
- Count the words. Over the limit → cut, do not shrink the proof.

### Step 5: Your prep note (for you, not for them)
3 lines:
1. **What they said**: the problem, in their words.
2. **What came up next with similar prospects**: from the source calls, the objection or hesitation they raised after this point (semantic search on the same calls for doubts about price, timing, effort or an existing tool). None found → say so.
3. **One question to ask** on the call, built from the gap between their reply and the proof call.

### Step 6: Hand over
Show the draft, the prep note and the source calls (title, date, link). Ask: copy-paste, or send from the lemlist inbox? Do not call `send_message` without a clear yes. Save to `reports/pre-call-{lead}-{date}.md`.

## End state
- [ ] Built on the lead's actual reply text, and answers their question if they asked one
- [ ] Not drafted for a negative reply
- [ ] Proof names no other company or person
- [ ] Every number traceable to a listed call
- [ ] A prep note for the user
- [ ] Nothing sent
