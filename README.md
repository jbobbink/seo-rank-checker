# Rank Checker by GSC Wizard

Track where your own website ranks in Google for the keywords you choose, from your own Google Search Console data. This repository connects Claude Code (and other MCP clients) to the Rank Checker endpoint of the [GSC Wizard](https://www.gscwizard.com) MCP server and bundles four guided skills: pick keywords worth tracking, report on them, explain drops, and clean up the list.

Keyword sets are per property, and each set can be limited to one country and one device, so a mobile ranking in the Netherlands is never mixed with a desktop ranking in Germany. Every keyword comes back with its position, the change against the previous period, the page that ranks, clicks and impressions, and every skill requires each figure it reports to name the tool and date range it came from.

## Where the rankings come from

The numbers come from your own Search Console account, not from a tool that searches Google on your behalf, and that has consequences worth knowing before you connect:

- **Average position on real searches.** A position is the average position Google Search Console measured for your property on the searches people actually made, for the set's country and device. It is not a scraped results page and there is no live check.
- **Daily, from final data, about three days behind.** Positions are stored once a day from final Search Console data, so the newest day shown is about three days old, and the days shown are complete rather than partial.
- **Your own site only.** There are no competitor rankings, and the tools cannot tell you who ranks above you.
- **No impressions, no position.** A keyword that received no impressions in a period has no position for it and shows as not ranking.
- **History from the start.** Each new keyword is read with up to 16 months of Search Console history, normally within the hour, so a trend is available straight away instead of after weeks of tracking.
- **Removing a keyword deletes its history.** Its stored days go with it and cannot be restored; adding it again reads only what Search Console still keeps.

## Skills

| Skill | What it does |
| :---- | :----------- |
| `/seo-rank-checker:track-keywords` | Check the existing sets, propose striking-distance and money queries from your own data, agree country, device and list, then create or extend a set |
| `/seo-rank-checker:rank-report` | Ranking report for a period (a week by default): improved, declined, new in the top 10 and top 3, not ranking, winners and losers, the page that ranks |
| `/seo-rank-checker:rank-drops` | Why tracked keywords fell: when, in which market or device, whether the ranking page changed or the whole site moved, and the probable causes |
| `/seo-rank-checker:prune-keywords` | Find keywords that never get impressions or drifted out of relevance and remove them after you confirm the exact list |

Claude Code also invokes these skills automatically when your request matches, so "why did my tracked keywords drop?" works without remembering a command name. In other agents each skill goes by its folder name (`track-keywords`, `rank-report`, ...). Every skill is a plain `SKILL.md` you can read before running it; there are no scripts in the skill folders.

## Tools

The endpoint `https://mcp.gscwizard.com/mcp/rank-checker` serves exactly nine tools:

| Tool | What it does |
| :--- | :----------- |
| `list_sites` | Your Search Console properties in GSC Wizard |
| `get_account_info` | Your account, connected Google accounts and plan |
| `list_rank_keyword_sets` | The keyword sets on a property, with each set's last daily check summary |
| `get_rank_keyword_data` | A set's positions for 7 days to 16 months against the period before, its daily trend, or one keyword's daily history |
| `add_rank_keywords` | Create a keyword set for a country and device, or add keywords to one (write) |
| `remove_rank_keywords` | Remove keywords, and their stored history, from a set (write) |
| `get_ranking_changes` | New, lost, improved and declined queries or pages across the whole property between two periods |
| `query_top_queries` | The property's top search queries, the source for keywords worth tracking |
| `get_query_performance` | The daily performance of one query, tracked or not, optionally on one page |

The two write tools are offered only to a connection with write access. Before either of them runs, the skills list the exact keywords and ask you to confirm.

## Requirements

- A [GSC Wizard](https://www.gscwizard.com) account: every account starts with a 7-day free trial and continues on a paid plan
- Your Google account connected to GSC Wizard, with Search Console access to the property you want to track

## Installation

Add this repository as a marketplace and install the plugin:

```
/plugin marketplace add jbobbink/seo-rank-checker
/plugin install seo-rank-checker@seo-rank-checker
```

On first use, Claude Code opens your browser to authorize the connection (OAuth 2.1). Approve access with your GSC Wizard account and you are done; tokens are stored and refreshed by Claude Code. If the connection ever needs re-authorizing, run `/mcp` and select `gsc-wizard-rank-checker`.

Prefer the server without the skills? `claude mcp add --transport http gsc-wizard-rank-checker https://mcp.gscwizard.com/mcp/rank-checker`

## Example prompts

- "Which keywords should I track for sc-domain:example.com in the Netherlands on mobile?"
- "How did my tracked keywords do this week?"
- "Give me a ranking report for the last 28 days"
- "Why did 'running shoes' drop in my keyword set?"
- "Clean up my keyword set: remove the keywords that never rank"

## How the skills are built

Each skill is one `SKILL.md`: a description with the phrasings that trigger it, numbered steps that name the tool and the arguments to call it with, the output format and a fixed grounding rule. The conventions every skill shares:

- Resolve the property with `list_sites` before anything else; never guess a site URL.
- A lower position is better, and a positive `change` is an improvement.
- A position built on a handful of impressions is noise, and a keyword with no impressions has no position.
- Before any tool that writes, list the exact keywords and ask for confirmation.
- Every figure names the tool and the date range it came from.

## Privacy and security

The plugin talks to `https://mcp.gscwizard.com` over OAuth 2.1; no API keys or credentials live in this repository. The server reads your Search Console data through the Google account you connected to GSC Wizard, scoped to your user; nothing is collected from your local agent setup.

- Documentation: [Rank Checker](https://www.gscwizard.com/docs/rankings/rank-checker.html)
- [Privacy policy](https://www.gscwizard.com/privacy.html)
- [Terms of service](https://www.gscwizard.com/terms.html)
- Support: [feedback form](https://www.gscwizard.com/feedback.html) or support@gscwizard.com

## Development

```
git clone https://github.com/jbobbink/seo-rank-checker
claude --plugin-dir ./seo-rank-checker
```

## License

MIT
