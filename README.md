# lemlist x Claap: 5 things I run on top of them

lemlist knows what you sent. Claap knows how the call went. They have never spoken to each other, so everybody optimises reply rate and nobody optimises the conversation that follows.

These 5 Claude Code skills join the two.

| # | Skill | What it answers |
|---|---|---|
| 1 | `lemlist-claap-icp-drift` | Is the ICP I loaded into my last campaigns still the one I'm positioned for? |
| 2 | `lemlist-claap-call-fit` | Who actually turned up on the calls each campaign produced? Ranked by good calls, not replies. |
| 3 | `lemlist-claap-outlier-check` | Was the deal I closed an outlier, or does the rest of the list share its context? Keep going or rebuild. |
| 4 | `lemlist-claap-pre-call-insight` | Someone replied. What useful thing do I send them before the call? |
| 5 | `lemlist-claap-opener-from-calls` | What did buyers say on calls that went somewhere, and what's my next first line? |

Plus `lemlist-claap-setup`, which connects both tools and proves the join works on your account.

## Start
Follow [SETUP.md](SETUP.md). Short version: clone, open Claude Code in the folder, `/mcp` → log in to lemlist and Claap, then say **"set up the playbook"**.

## What it will never do
- Send an email, launch a campaign or change a live sequence without your yes
- Create Claap fields, views or automations without asking (they apply to the whole workspace)
- Make up a number. Everything in a report comes from your data in that run.

## Layout
```
.mcp.json                 lemlist + Claap servers (OAuth, no keys)
CLAUDE.md                 join rules + safety rules every skill follows
SETUP.md                  step-by-step connection guide
.claude/skills/<name>/    one folder per skill
docs/                     your ICP + connection notes (written by setup)
reports/                  outputs (git-ignored)
```
