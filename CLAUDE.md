# lemlist x Claap playbook — rules for Claude

This repo runs 5 skills on top of two MCP servers: **lemlist** (what was sent, who replied) and **Claap** (how the call went). The whole point is joining the two.

## Shared files
- `docs/icp.md` — the ICP and positioning. Written by `lemlist-claap-setup`. Every skill reads it. If it is missing, run the setup skill first.
- `docs/connection.md` — the Claap workspace ID, the lemlist team, and which join key works for this account. Written by `lemlist-claap-setup`.
- `reports/` — every skill writes its output here as `reports/{skill}-{YYYY-MM-DD}.md`. Git-ignored, because it holds lead names and call content.

## The join (how a lemlist lead becomes a Claap call)
1. **Email first.** lemlist lead email == an attendee email on a Claap recording, or the email of a Claap contact (`search_contacts`).
2. **Company domain second.** The lead's email domain or `companyDomain` == a Claap company's domain (`search_companies`). Mark these matches as `domain-match`, they are weaker.
3. **Time window.** Only count a call that happened AFTER the lead's first email in that campaign, and within 60 days of their reply. Otherwise the call did not come from the campaign.
4. Never match on first name or company name alone.

Always report the match rate (`X of Y replied leads found in Claap`). Below 50%, say so before drawing any conclusion. The finding is only as good as the join.

## Hard rules
- **Nothing is sent, launched or written to lemlist without an explicit yes in the conversation.** Skills draft. The human sends.
- **Claap writes are workspace-wide** (fields, views, automations). Never create one without asking first.
- **No invented numbers.** Every count in a report comes from a tool call made in this run. If a number cannot be fetched, write `unknown`, never an estimate.
- **No secrets in chat or in files.** Both servers use OAuth. If someone uses API keys instead, they live in `~/.claude.json` via `claude mcp add --header`, never in this repo.
- Claap needs a `workspaceId` on every call. Read it from `docs/connection.md`. If there are several workspaces and none is saved, ask which one, never pick.
- lemlist tool names can change. At the start of a run, look at which `lemlist` tools are available and pick the ones that list campaigns, read campaign stats, list leads, and list activities. If only a generic `call_api` tool exists, use the lemlist REST paths named in each skill.
