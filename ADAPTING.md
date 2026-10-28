# Using these skills with other tools

The 5 skills are built on lemlist and Claap, but the logic is not. Each skill only needs a short list of things from **a sequencer** (what was sent, who replied) and **a call recorder** (who was on the call, what they said). Any pair of tools that can answer those questions can run the playbook, and Claude Code can do the remapping for you.

## What the skills need

### From your sequencer
| # | Capability | lemlist today | Used by | If your tool can't do it |
|---|---|---|---|---|
| S1 | List campaigns, with a creation date | `get_campaigns` + `get_campaign_details` | all | Ask the user which campaigns, in which order |
| S2 | Leads of a campaign with job title, company size, industry, domain, and the date each lead was added | `search_campaign_leads` + `fields` | 1, 2, 3 | CSV export of the campaign |
| S3 | Per-lead activity with dates: first email sent, reply | `search_campaign_leads` + `include: ["activities"]` | 1, 2, 3, setup | CSV export with sent and replied dates. **Without dates, nothing is attributable**: the join needs them. |
| S4 | Reply text (sentiment is a bonus) | `get_inbox_conversation` | 2, 4 | The user pastes the reply |
| S5 | Funnel per campaign (contacted, replied), ideally per A/B variant | `get_campaigns_stats` | 2, 5 | Count from S3 |
| S6 | Read a sequence, write an A/B variant | `get_campaign_sequences`, `set_ab_variant` | 5 | Skill 5 hands the user the text to paste |
| S7 | Count people matching filters (sizing a list) | `people_database_search_count` | 3 | Skip sizing, or use your data provider |

### From your call recorder
| # | Capability | Claap today | Used by | If your tool can't do it |
|---|---|---|---|---|
| R1 | Calls by attendee email, with dates | `get_recordings` + `contactEmail` | 2, 3, 4, setup | Nothing replaces it: this is the join |
| R2 | Calls by company or domain | `search_companies` → `get_recordings` + `companyId` | the join (fallback) | Email-only join, say so in every report |
| R3 | Full transcript with speaker names, and which speakers are external | `get_recording_transcript` + `speakers[].isExternal` | 2, 3, 4, 5 | Use the team-names list in `docs/connection.md` to tell sides apart |
| R4 | Search across transcripts by meaning | `search_recording_transcripts` (semantic) | 4, 5 | Keyword search, or read the transcripts of the chosen calls one by one (slower, same result) |
| R5 | Won deals | `search_deals` (needs a CRM in Claap) | 3, 5 | Your CRM's own MCP, or ask the user |

**The one non-negotiable is the join**: a lead's email in the sequencer must find that person's calls in the recorder, and both sides must carry dates. If your recorder can only search by meeting title, the playbook will guess, and it is built not to guess.

## How to adapt it with Claude Code

1. **Connect your tools.** Add their MCP servers to `.mcp.json` (or connect them as claude.ai connectors). Many sequencers, recorders and CRMs ship an MCP server now: check the vendor's docs. No MCP server → see "No MCP server" below.
2. **Open Claude Code in this folder and paste:**

   > Adapt this playbook to {your sequencer} and {your recorder}. Read ADAPTING.md and CLAUDE.md. For every capability in the two tables of ADAPTING.md, find the tool that does it on my servers, call it once on real data and write down what comes back (field names, paging, date format, limits). Then rewrite the tool map in CLAUDE.md for my tools, keep the join, the quoting rules and the hard rules exactly as they are, and update the tool names inside each skill. List the capabilities my tools can't do and which fallback you used. Don't send, launch or write anything in my tools while testing.

3. **Run the setup skill** ("set up the playbook"). Its join test is the proof: if it reaches **Ready**, every skill will work on your tools.
4. Optionally rename the skills (`lemlist-claap-*` → `{your-tools}-*`): the folder name and the `name:` line must match.

What Claude Code should check on every new tool, because each one bit us on lemlist or Claap:
- **Paging**: does the next-page cursor return everything? Compare the rows you got with the announced total.
- **Silent fields**: some APIs only return a field if you ask for it by name.
- **Date precision**: day or timestamp. The join compares days.
- **Lookup vs list modes**: looking one person up can return less than listing a campaign.
- **Speaker labels**: transcripts sometimes put a line under the wrong speaker.
- **Naming**: some OAuth servers reject a client whose name contains the vendor's name (lemlist does, which is why our server is called `lem`).

## No MCP server?
- **API**: ask Claude Code to write a small script per capability that calls the tool's REST API and prints JSON. The API key goes in your shell environment or in `~/.claude.json`, never in this repo.
- **CSV exports**: export campaigns, leads and activity from the sequencer into `data/` (git-ignored) and tell the skills to read them instead of calling a tool. Transcripts can be exported the same way. It works, it is just not live.

## Common pairs
Sequencers people run this with: Instantly, Smartlead, Apollo, Outreach, Salesloft, HubSpot sequences, La Growth Machine. Call recorders: Gong, Fathom, Fireflies, Granola, Modjo, Avoma, tl;dv. Deals: HubSpot, Salesforce, Attio, Pipedrive. What each one exposes changes often, so let Claude Code test the capabilities on your account instead of trusting a list.
