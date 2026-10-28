# Connect lemlist and Claap to Claude Code

About 10 minutes. You need: Claude Code installed, a lemlist account, a Claap account on a paid plan, and the Claap recorder joining the calls your campaigns book.

Using another sequencer or call recorder? Read [ADAPTING.md](ADAPTING.md) first.

## Step 1: Get the repo
```bash
git clone https://github.com/Othmane-Khadri/ai-agents-crm-call-recorders-lemlist-claap.git
cd ai-agents-crm-call-recorders-lemlist-claap
```

## Step 2: Connect both tools (pick one way)

### Option A: claude.ai connectors (easiest)
1. On [claude.ai](https://claude.ai), open Settings → Connectors.
2. Find **lemlist**, click Connect, log in, pick your team, approve.
3. Find **Claap**, click Connect, approve.
4. Log in to Claude Code with the same Claude account. The two connectors show up there too (`/mcp` lists them in its claude.ai section).

When Claude Code asks whether to trust this repo's own MCP servers, say **no**: you already have both, and you don't want two of each.

### Option B: this repo's servers
The repo ships a `.mcp.json` with both servers:

| Server name | Tool | URL |
|---|---|---|
| `lem` | lemlist | `https://app.lemlist.com/mcp` |
| `claap` | Claap | `https://api.claap.io/mcp` |

1. Run `claude` in the repo folder. Claude Code asks whether to trust the project's MCP servers: yes to both.
2. Make sure you are logged in to [app.lemlist.com](https://app.lemlist.com) and [app.claap.io](https://app.claap.io) in your browser.
3. Type `/mcp` → **lem** → Authenticate. Your browser opens: pick the lemlist team, approve.
4. Type `/mcp` → **claap** → Authenticate. Approve.
5. Run `/mcp` again. Both should show `connected`.

Why `lem` and not `lemlist`: lemlist's login refuses any app whose name contains "lemlist" (error `invalid_client_metadata`). Keep the short name.

A server that shows `needs authentication` is invisible to Claude until you finish this step. If you just authorised it, start a new Claude Code session so the tools load.

## Step 3: Run the setup skill
In Claude Code, say:
> set up the playbook

It will:
1. confirm both tools answer and pick your Claap workspace (it asks if you have several)
2. measure how complete your lead data is (job title, size, industry, domain, location, technologies) and tell you which skills a gap will weaken
3. take your most-replied campaign and check how many replied leads it can find in Claap. **This number is the whole game.** You get one of 4 verdicts: *Ready*, *Plumbing works, no results yet* (nothing sent yet), *Calls not linked to campaigns*, or *Not found*.
4. interview you for your ICP and write `docs/icp.md`

**Two things that make the results better:**
- Load job title, company size and industry on your lemlist leads (`jobTitle`, `companySize`, `companyIndustry`). Skills 1 and 3 score on them. Without them, leads are scored `unknown`, never guessed. Location and technologies unlock the region and stack checks.
- Connect HubSpot to Claap if you can. Skill 3 then finds your won deals itself; without it, it asks you.

## Step 4: Run the 5 things
| Say | What runs |
|---|---|
| "check the ICP of my last campaigns" | 1. ICP drift |
| "who turned up on the calls from my campaigns" | 2. Call fit |
| "was the {company} deal an outlier" | 3. Outlier check |
| "{name} from {company} replied, send them something before the call" | 4. Pre-call insight |
| "write my next opener from my calls" | 5. Opener from calls |

If a sentence doesn't trigger its skill, type its name: `/lemlist-claap-call-fit`.

Reports land in `reports/`. Your ICP and connection notes stay in `docs/`. All three are git-ignored: they contain names, positioning and call content.

## If something breaks

**`invalid_client_metadata` when authenticating lemlist.** The server is named something containing "lemlist". Keep it as `lem` in `.mcp.json`.

**A server shows `failed` or `needs auth` in `/mcp`.** Re-run Authenticate. For Claap, the URL must have no trailing slash.

**You prefer API keys (headless runs, CI).** Remove the server from `.mcp.json` and add it to your user config instead, so the key never sits in the repo:
```bash
claude mcp add --transport http --scope user lem https://app.lemlist.com/mcp \
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

**Too many lemlist tools for your context window.** lemlist lets you narrow them with a bucket filter, for example `https://app.lemlist.com/mcp?bucket=insights`. The skills need campaigns, leads, activities, inbox and stats.
