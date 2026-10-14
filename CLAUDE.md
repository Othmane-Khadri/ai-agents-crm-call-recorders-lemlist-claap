# lemlist x Claap playbook — rules for Claude

This repo runs a setup skill and 5 skills on top of two MCP servers: **lemlist** (what was sent, who replied) and **Claap** (how the call went). The whole point is joining the two.

## Shared files
- `docs/icp.md` — ICP and positioning. Written by `lemlist-claap-setup`. Every skill reads it. Missing → run the setup skill first.
- `docs/connection.md` — Claap workspace ID, lemlist team, which join key works. Written by `lemlist-claap-setup`. Use its workspace ID instead of calling `list_workspaces` again.
- `reports/` — each skill writes the file its own SKILL.md names. Create the folder if missing. Git-ignored: it holds lead names and call content. In file names, `{lead}` is the lead's email and `{company}` is a lowercase slug.

## Finding the tools
The same server can be mounted under different prefixes, depending on how it was connected:
- lemlist: `mcp__lemlist__*` (this repo's `.mcp.json`) or `mcp__claude_ai_Lemlist__*` (claude.ai connector)
- Claap: `mcp__claap__*` or `mcp__claude_ai_Claap__*`

Either works. If tools are deferred, load them by name first. Both servers have a `search_companies` and a `search_contacts`: always say which server you mean, they return different things.

## Tool map (verified live against both servers, 2026-10-14)

**lemlist**
| Need | Tool | Verified behaviour |
|---|---|---|
| List campaigns | `get_campaigns` | 20 per page: page with `offset` until you have `totalRecords` (`total` is the page size, not the count). No creation date in the rows. |
| Creation date, senders | `get_campaign_details` (`campaignIds`, max 20) | has `createdAt` |
| Funnel | `get_campaigns_stats` (`campaignIds[]`) | `leadMetrics.reached` = contacted, `leadMetrics.answered` = replied, `leadMetrics.interested`; `messageMetrics.sent`. `sent: 0` = never sent. |
| Leads + title + firmographics | `search_campaign_leads` (`campaignId`, `limit` ≤100, `offset`, `fields: ["companySize","companyIndustry","companyDomain"]`) | returns `jobTitle` when one is set. **Firmographics only come back if you name them in `fields`.** Page until `hasMore` is false. |
| One lead by email | `search_campaign_leads` (`email`) | finds the campaign(s) a person is in, with `status` and `campaign.status` |
| Who replied, and when | `search_campaign_leads` with `include: ["activities"]` (`limit` ≤50) | activities are a list with a `type` (`emailsSent`, `emailsOpened`, `emailsReplied`, `linkedinReplied`, `emailsInterested`...) and a date. **A lead with no activity returns `activities: ""`** (empty string): treat it as no activity. |
| Reply text | `get_inbox_conversations` (`search` = email, `listId: "teamConversations"`) → `get_inbox_conversation` (`contactId`) | never set `markAsRead` |
| Sequence copy, IDs | `get_campaign_sequences` (`campaignId`) | gives `sequenceId` and each step's `stepId` |
| Add an A/B variant | `set_ab_variant` (`sequenceId`, `stepId`, `message`, optional `subject`) | write, needs a yes |
| Team name | `get_team_info` | |

Do **not** use lemlist `search_companies` for firmographics: it returns only id, name and domain, and ignores id filters.

**Claap** (every call needs `workspaceId`)
| Need | Tool | Verified behaviour |
|---|---|---|
| Calls by attendee | `get_recordings` (`filters.contactEmail`) | 10 per page by default: raise `limit` (max 100). Past 100 results `nextCursor` can return an empty page: split by `createdAt` windows instead. `hasExternalSpeaker` counts anyone outside the workspace, including team members on personal emails. Each recording has `createdAt` (a date, YYYY-MM-DD), `people[]` with emails, `companies[]`, `recordingId`, `url`. |
| Calls by company | `get_recordings` (`filters.companyId: [id]`) | use the ID, never `companyName` (substring match, over-matches) |
| Company by domain | Claap `search_companies` (`name` = the domain) | returns the company ID |
| One call + AI summary | `get_recording` | cheap. The summary is a paraphrase (may be in another language): **never quote it as the prospect's words.** |
| Full transcript | `get_recording_transcript` | the only source of verbatim quotes |
| Search what was said | `search_recording_transcripts` (`search {query, type: semantic|keyword}`, `filters`: `hasExternalSpeaker`, `recordingId`, `companyId`, `createdAt`, `tag`) | `limit` counts chunks, not calls. `tag`: `PainPoints`, `Objections`, `CompetitorMentions`, `FeatureRequests` |
| Deals | `search_deals` (`filters` required), `get_deal`, `list_deal_stages` | **only with a CRM connected to Claap (HubSpot).** `list_deal_stages` errors "Crm not connected" and `search_deals` returns 0: that means no CRM, not no wins. |

## The join (how a lemlist lead becomes a Claap call)
1. **Skip junk domains.** Free mailboxes (gmail, googlemail, outlook, hotmail, live, yahoo, icloud, proton) and reserved test domains (example.com/.org/.net, test, invalid) never get a domain match.
2. **Email first.** `get_recordings` with `filters.contactEmail` = the lead's email. Page through all results.
3. **Domain second.** No email hit → Claap `search_companies(name: domain)` → `get_recordings(filters.companyId)`. Label it `domain-match` and name the person who was actually on the call: it is someone else at the company.
4. **Time window.** Find the lead's first `emailsSent` activity date. A call counts for the campaign only if it is on or after that date and within 60 days of the lead's reply. Claap dates are day-precision, so compare days.
   - Call before the first email → **prior relationship**: list separately, never count.
   - **No `emailsSent` at all** (draft, or lead not reached yet) → every call is a prior relationship. Nothing is attributable.
5. Never match on a first name or a company name typed by hand.

Always report the match rate: `X of Y leads found in Claap (A by email, B by domain), Z attributable to the campaign`. Below 50% attributable, say so before drawing any conclusion.

## Data gaps you will hit, and what to do
- **Campaign never sent** (`messageMetrics.sent: 0`): there are no replies and nothing is attributable. Say it in one line. Offer a **join check**: run the join on every lead to prove the plumbing and report prior relationships, using the labels above. Never present that as campaign results.
- **Job title empty** on more than half the leads of a campaign (strictly more than 50%, per campaign): say so and score those leads `unknown`. Offer lemlist enrichment (`bulk_enrich_leads`) only with the lead count, it **costs credits**: wait for a yes.
- **Firmographics empty**: score on the title alone and mark the lead with `*` (title-only). Never infer size or industry from a company name.
- **No deals in Claap**: ask the user which deals they won.

## Hard rules
- **Nothing is sent, launched or started without an explicit yes.** Never call `launch_campaign`, `set_campaign_state` with `start`, `send_message`, `send_task` on your own. Skills draft, the human sends.
- `create_campaign_with_sequence` reports `"status": "running"` on a brand-new campaign that is actually a draft. Confirm with `get_campaigns` before acting on a status.
- Adding leads: pass `findEmail`, `verifyEmail`, `linkedinEnrichment`, `findPhone` as `false` unless the user asked for paid enrichment.
- **Claap writes are workspace-wide** (fields, views, automations). Never create one without asking.
- **No invented numbers.** Every count comes from a tool call made in this run. Unknown → `unknown`.
- **No secrets in chat or in files.** API keys, if used, live in `~/.claude.json` via `claude mcp add --header`, never in this repo.
- **Anything meant for a prospect** never names another customer or person, and never reuses a customer's exact figures (they can be traced back). Round them ("about 3,000", "roughly half") or ask the user whether that customer agreed to be a reference.
