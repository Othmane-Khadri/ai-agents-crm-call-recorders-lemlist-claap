# Connect lemlist and Claap to Claude Code

About 10 minutes. You need: Claude Code installed, a lemlist account, a Claap account on a paid plan, and the Claap recorder joining the calls your campaigns book.

## Step 1 — Get the repo
```bash
git clone <this-repo-url> lemlist-claap-playbook
cd lemlist-claap-playbook
```

## Step 2 — Open Claude Code in the repo
```bash
claude
```
The repo ships a `.mcp.json` that declares both servers:

| Server | URL |
|---|---|
| lemlist | `https://app.lemlist.com/mcp` |
| Claap | `https://api.claap.io/mcp` |

Claude Code asks whether to trust the project's MCP servers. Say yes to both.

**Already connected lemlist or Claap on claude.ai?** If you are logged in to Claude Code with the same account, those connectors show up in Claude Code too (as `claude_ai_Lemlist` / `claude_ai_Claap` in `/mcp`). Use them and decline the repo's copy of that server, so you don't have two of each. The skills work with either.

## Step 3 — Log in to both (OAuth, no API key)
1. Make sure you are logged in to [app.lemlist.com](https://app.lemlist.com) and [app.claap.io](https://app.claap.io) in your browser.
2. In Claude Code, type `/mcp`.
3. Select **lemlist** → Authenticate. Your browser opens, pick the lemlist team, approve.
4. Select **claap** → Authenticate. Approve in the browser.
5. Run `/mcp` again. Both should show `connected`.

A server that shows `needs authentication` is invisible to Claude until you finish this step, even if you connected it elsewhere. If you just authorised it, start a new Claude Code session so the tools load.

## Step 4 — Run the setup skill
In Claude Code, say:
> set up the playbook

It will:
1. confirm both servers answer and pick your Claap workspace (it asks if you have several)
2. take your most-replied campaign and check how many replied leads it can find in Claap. **This number is the whole game.** You get one of 4 verdicts: *Ready*, *Plumbing works, no results yet* (nothing sent yet), *Calls not linked to campaigns*, or *Not found*.
3. interview you for your ICP and write `docs/icp.md`

**Two things that make the results better:**
- Load job title, company size and industry on your lemlist leads (`jobTitle`, `companySize`, `companyIndustry`). Skills 1 and 3 score on them. Without them, leads are scored `unknown`, never guessed.
- Connect HubSpot to Claap if you can. Skill 3 then finds your won deals itself; without it, it asks you.

## Step 5 — Run the 5 things
| Say | What runs |
|---|---|
| "check the ICP of my last campaigns" | 1. ICP drift |
| "who turned up on the calls from my campaigns" | 2. Call fit |
| "was deal X an outlier" | 3. Outlier check |
| "send {lead} something before the call" | 4. Pre-call insight |
| "write my next opener from my calls" | 5. Opener from calls |

Reports land in `reports/` (git-ignored, they contain names and call content).

## If something breaks

**A server shows `failed` or `needs auth` in `/mcp`.** Re-run Authenticate. For Claap, the URL must have no trailing slash.

**You prefer API keys (headless runs, CI).** Remove the server from `.mcp.json` and add it to your user config instead, so the key never sits in the repo:
```bash
claude mcp add --transport http --scope user lemlist https://app.lemlist.com/mcp \
  --header "X-API-Key: <your lemlist key>"
claude mcp add --transport http --scope user claap https://api.claap.io/mcp \
  --header "Authorization: Bearer <your claap key>"
```
lemlist key: Settings → Team → Integrations → Generate. Claap key: see Claap's "Using Claap's API" help article. Never commit either.

**The join finds almost nobody.** Three usual causes:
1. The Claap recorder is not joining the calls (check Claap's calendar settings).
2. Prospects book with a different email than the one you emailed. The skills fall back to company domain, but that is weaker.
3. Calls are booked on someone else's calendar that Claap does not record.

**Deals are empty in Claap.** Deal tools need a CRM connected to Claap (HubSpot today). Skill 3 then asks you for the company name instead.

**Too many lemlist tools for your context window.** lemlist lets you narrow them with a bucket filter, for example `https://app.lemlist.com/mcp?bucket=insights`. The skills need campaigns, leads, activities and stats.
