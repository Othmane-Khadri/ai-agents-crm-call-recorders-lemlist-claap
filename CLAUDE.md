# lemlist x Claap playbook: rules for Claude

This repo runs a setup skill and 5 skills on top of two MCP servers: **lemlist** (what was sent, who replied) and **Claap** (how the call went). The whole point is joining the two.

Using another sequencer or call recorder? Read `ADAPTING.md` before anything else: it says what each skill needs and how to remap it.

## Shared files
- `docs/icp.md`: ICP and positioning. Written by `lemlist-claap-setup`, read by every skill. Missing → run the setup skill first. It can hold several segments (`## Segment: {name}`); score a lead against the segment it fits best.
- `docs/connection.md`: Claap workspace ID, lemlist team, which join key works, your team's names. Written by `lemlist-claap-setup`. Use its workspace ID instead of calling `list_workspaces` again.
- `reports/`: each skill writes the file its own SKILL.md names. Create the folder if missing. In file names, `{lead}` is the lead's email and `{company}` is a lowercase slug.
- `docs/icp.md`, `docs/connection.md` and `reports/` are git-ignored: they hold your positioning, lead names and call content.

## Finding the tools
The same server can be mounted under different prefixes, depending on how it was connected:
- lemlist: `mcp__lem__*` (this repo's `.mcp.json`, see the note below) or `mcp__claude_ai_Lemlist__*` (claude.ai connector)
- Claap: `mcp__claap__*` or `mcp__claude_ai_Claap__*`

Either works. If tools are deferred, load them by name first. Both servers have a `search_companies` and a `search_contacts`: always say which server you mean, they return different things.

The lemlist server is named `lem` on purpose: lemlist's login rejects any client whose name contains "lemlist" (`invalid_client_metadata`). Never rename it back.

## Tool map (verified live against both servers, 2026-10-28)

**lemlist**
| Need | Tool | Verified behaviour |
|---|---|---|
| List campaigns | `get_campaigns` | 20 per page by default (`limit` up to 100): page with `offset` until you have `totalRecords`. `total` is the page size, not the count. Rows have no creation date. `search` matches the name. |
| Creation date, senders | `get_campaign_details` (`campaignIds`, 1-20) | has `createdAt` (full timestamp) |
| Funnel | `get_campaigns_stats` (`campaignIds[]`, optional `startDate`/`endDate`, `ABSelected: "A"|"B"`) | `leadMetrics.reached` = contacted, `leadMetrics.answered` = replied, `leadMetrics.interested`; `messageMetrics.sent`. `sent: 0` = never sent. `ABSelected` splits the funnel by variant. |
| Leads of a campaign | `search_campaign_leads` (`campaignId` or `campaignIds`, `limit` ≤100, `offset`, `fields: [...]`) | Page until `hasMore` is false. `jobTitle`, `companyName`, `createdAt` always come back. **Company fields only come back if you name them in `fields`**: `companySize`, `companyIndustry`, `companyDomain`, and when your leads carry them `companyLocation`, `location`, `companyTechnologies`, `companyEmployeeCount`, `companySubIndustry`. A field the lead does not have is simply absent. |
| Who replied, and when | `search_campaign_leads` (campaign mode) with `include: ["activities"]` (`limit` ≤50) | **`activities` is a text block, one event per line: `[YYYY-MM-DD] {type}: {detail}`** (for example `[2026-10-18] emailsSent: Orbitdesk's SDR ramp`). Types include `emailsSent`, `emailsOpened`, `emailsReplied`, `linkedinReplied`, `emailsInterested`. Dates are day-precision. No activity → `""`. |
| One lead by email | `search_campaign_leads` (`email`, optional `campaignId`) | returns the campaign(s), `status`, `createdAt` and a `variables` object (title, size, industry, domain). **This mode ignores `include` and `fields`: no activities.** To get that lead's activities, use the lookup below. |
| One lead's activities | `search_campaign_leads` (`campaignId`, `include: ["activities"]`, `addedAfter` = the lead's `createdAt` minus 1 second, `addedBefore` = plus 1 second) | returns the leads added in that same second (a CSV import adds many at once): pick the one with the right email. |
| Replies of a campaign | `get_inbox_conversations` (`listId: "teamConversations"`, `campaignFilter: {in: [campaignId]}`, `limit` ≤50) | one row per contact who has a conversation; `search` = an email narrows to one person |
| Reply text + sentiment | `get_inbox_conversation` (`contactId`) | each reply carries `aiLeadInterest` (`positive` / `neutral` / `negative`) when lemlist scored it, null otherwise. Never set `markAsRead`. |
| Sequence copy, IDs | `get_campaign_sequences` (`campaignId`) | gives `sequenceId` and each step's `id` (`stp_...`). An A/B step returns variant B in `variantB` and the test state in `abTest`. |
| Add an A/B variant | `set_ab_variant` (`sequenceId`, `stepId`, `message`, optional `subject`) | write, needs a yes. With no test yet, B starts as a copy of A and only the fields you pass change. **A running campaign can reject edits to a step that already sent: the user must pause it first.** |
| Team name, your user | `get_team_info` | |

Do **not** use lemlist `search_companies` for firmographics: it returns only id, name and domain.

**Claap** (every call needs `workspaceId`)
| Need | Tool | Verified behaviour |
|---|---|---|
| Calls by attendee | `get_recordings` (`filters.contactEmail`, `limit: 100`) | **Always ask for `limit: 100` and check that you got `total` rows.** Following `nextCursor` drops results, even below 100 (verified: total 14, two pages of 5 returned 6). Above 100, split by `filters.createdAt` windows. Each row: `createdAt` (YYYY-MM-DD), `people[]` with emails, `companies[]`, `speakers[]` with `isExternal`, `source`, `recordingId`, `url`. |
| Calls by company | `get_recordings` (`filters.companyId: [id]`, `limit: 100`) | use the ID, never `companyName` (substring match, over-matches) |
| Company by domain | Claap `search_companies` (`name` = the domain) | returns `companyId` and `domains[]` |
| One call + AI summary | `get_recording` | cheap. The summary is a paraphrase, often in the call's language: **never quote it as the prospect's words.** |
| Full transcript | `get_recording_transcript` | the only source of verbatim quotes. Plain text, one turn per line as `{Speaker name}: {text}`, **no timestamps**. Speaker labels are sometimes wrong (see "Quoting a prospect"). |
| Search what was said | `search_recording_transcripts` (`search: {query, type: semantic|keyword}`, `filters`: `hasExternalSpeaker`, `recordingId`, `companyId`, `contactEmail`, `createdAt`, `tag`) | returns chunks grouped by call; chunk text is `MM:SS {Speaker}: {text}`, so search results do carry timestamps. `limit` counts chunks, not calls. Speakers in results have no `isExternal`: take it from `get_recordings`. `tag` (`PainPoints`, `Objections`, ...) is often empty: do not rely on it. |
| Deals | `search_deals`, `get_deal`, `list_deal_stages` | **only with a CRM connected to Claap (HubSpot).** `list_deal_stages` errors "Crm not connected" and `search_deals` returns 0: that means no CRM, not no wins. `filters.stage` is a substring match. |

## The join (how a lemlist lead becomes a Claap call)
1. **Skip junk domains.** Free mailboxes (gmail, googlemail, outlook, hotmail, live, yahoo, icloud, proton) and reserved test domains (example.com/.org/.net, test, invalid) never get a domain match.
2. **Email first.** `get_recordings` with `filters.contactEmail` = the lead's email, `limit: 100`.
3. **Domain second.** No email hit → Claap `search_companies(name: domain)` → `get_recordings(filters.companyId)`. Label it `domain-match` and name the person who was actually on the call: it is someone else at the company.
4. **Time window.** Find the lead's first `emailsSent` line in its activities. A call counts for the campaign only if it is on or after that date and within 60 days of the lead's reply. Both sides are day-precision: compare days.
   - Call before the first email → **prior relationship**: list separately, never count.
   - **No `emailsSent` at all** (draft, or lead not reached yet) → every call is a prior relationship. Nothing is attributable.
5. Never match on a first name or a company name typed by hand.
6. Calls made from lemlist's own dialer show in Claap with `source: "LemlistVoip"`: they join like any other call.

Always report the match rate: `X of Y leads found in Claap (A by email, B by domain), Z attributable to the campaign`. Below 50% attributable, say so before drawing any conclusion.

## Quoting a prospect
Every quote shown to the user, or reused in copy, must be something the **prospect** said, word for word.
1. Take the words from `get_recording_transcript` or from a search chunk, never from the summary.
2. Keep only lines whose speaker is external (`isExternal: true` in `get_recordings`) and is not one of your team's names (stored in `docs/connection.md`; ask once if missing). A teammate on a personal email shows as external.
3. **Speaker labels can be wrong**: a line tagged with the prospect's name sometimes belongs to your side. Drop a line if it pitches your offer, says "we" about your company, or asks the discovery questions a seller asks. When unsure, read the lines around it.
4. Transcripts are machine-made: keep the words, but you may cut filler ("uh", "like", repeated words) and mark the cut with `…`. Never add or swap a word.

## Small samples
Say how many things a conclusion rests on. Under 5 calls in a campaign, or a verdict built on a single deal, write **directional** next to it and do not rank on it alone.

## Data gaps you will hit, and what to do
- **Campaign never sent** (`messageMetrics.sent: 0`): there are no replies and nothing is attributable. Say it in one line. Offer a **join check**: run the join on every lead to prove the plumbing and report prior relationships, using the labels above. Never present that as campaign results.
- **Job title empty** on more than half the leads of a campaign (strictly more than 50%, per campaign): say so and score those leads `unknown`. Offer lemlist enrichment (`bulk_enrich_leads`) only with the lead count, it **costs credits**: wait for a yes.
- **Firmographics empty**: score on the title alone and mark the lead with `*` (title-only). Never infer size, industry, region or stack from a company name.
- **No deals in Claap**: ask the user which deals they won.

## Hard rules
- **Nothing is sent, launched, started or paused without an explicit yes.** Never call `launch_campaign`, `set_campaign_state`, `send_message`, `send_task` on your own. Skills draft, the human sends.
- `create_campaign_with_sequence` reports `"status": "running"` on a brand-new campaign that is actually a draft. Confirm with `get_campaigns` before acting on a status.
- Adding leads: pass `findEmail`, `verifyEmail`, `linkedinEnrichment`, `findPhone` as `false` unless the user asked for paid enrichment.
- **Claap writes are workspace-wide** (fields, views, automations). Never create one without asking.
- **No invented numbers.** Every count comes from a tool call made in this run. Unknown → `unknown`.
- **No secrets in chat or in files.** API keys, if used, live in `~/.claude.json` via `claude mcp add --header`, never in this repo.
- **Anything meant for a prospect** never names another customer or person, and never reuses a customer's exact figures (they can be traced back). Round them ("about 3,000", "roughly half") or ask the user whether that customer agreed to be a reference.
