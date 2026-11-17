---
name: rank-report
description: 'Report on the keywords a property tracks in the Rank Checker by GSC Wizard over a period (the last 7 days by default, or 28 days, 3, 6, 12 or 16 months), each set against the period before: headline counts of improved, declined, new in the top 10 and top 3, lost and not ranking, the daily trend, winners and losers tables, the page that ranks for each keyword, and notes on the three-day data lag. Use when the user asks for a ranking report, how their tracked keywords did this week or month, or a rank tracker update.'
---

# Rank Report

Report how the tracked keywords of one property moved over a period, one keyword set at a time. The user's request may contain a property, a set name, a period in any phrasing ("this week", "last month", "this quarter", "year on year") and a focus (one market, the top 10 only, one product line); honor each.

What the numbers are: positions are the average position Google Search Console reports for the user's own property on real searches, for the set's country and device, stored once a day from final data. They are not a live results page check and include no competitor. A keyword with no impressions in a period has no position for it.

## Steps

1. **Resolve the property.** If the user's request or the conversation names a property, use it. Site URLs use GSC formats: `sc-domain:example.com` or `https://www.example.com/`. Otherwise call `list_sites` and pick the obvious match; if several fit, ask which one.

2. **List the sets.** Call `list_rank_keyword_sets` with `siteUrl`. Report on every set unless the user named one. For each set note `name`, `country` and `device` (null means all), `keywordCount`, `lastCheckedAt`, `dataUpTo` (the newest day stored) and `lastCheckStatus`. When `lastCheckStatus` is not "ok", quote `lastCheckError` and say the figures may be older than usual; "no_access" and "token_error" mean the Google connection behind the property needs attention in GSC Wizard. If `sets` is empty, say so, quote `emptyBecause` and offer the track-keywords skill.

3. **Pick the period.** `get_rank_keyword_data` takes `days` as 7 (the default), 28, 91, 182, 365 or 485, and nothing else. Map the user's phrasing to the nearest value (a week is 7, a month 28, a quarter 91, half a year 182, a year 365, everything Search Console keeps 485) and say which you used. Each period ends on the set's newest stored day and is compared with the same number of days directly before it.

4. **Read each set.** Call `get_rank_keyword_data` with `siteUrl`, `setId` and `days`.
   - `period` gives `startDate`, `endDate`, `previousStartDate`, `previousEndDate`, `comparable` and `storedFrom`. When `comparable` is false the period before starts earlier than the stored history: there is no previous position, change, new or lost, so report the current period only and say why.
   - Each row in `keywords` has `keyword`, `status`, `position`, `previousPosition`, `change`, `clicks`, `impressions`, `previousImpressions` and `rankingUrl`. `position` is impression-weighted over the period and lower is better; `change` is previous minus current, so positive is an improvement.
   - `status` is "ranking" (impressions in both periods), "new" (impressions now, none before), "lost" (impressions before, none now), "not_ranking" (none in either) or "pending" (not checked yet).
   - `trend` has one row per day with `date`, `keywords`, `avgPosition`, `top3`, `top10`, `clicks` and `impressions` for the whole set.
   - If the answer carries `emptyBecause`, quote it: a set that was never checked fills within the hour.

5. **Headline counts.** Derive these per set from `keywords` and say they are derived:
   - **improved** and **declined**: `change` of at least 1 place up or down, the rule the Rank Checker's own set summary uses;
   - **new in the top 10** and **new in the top 3**: `position` at or under 10 (or 3) where `previousPosition` was above it or absent;
   - **left the top 10**: the reverse;
   - **lost** and **not ranking**: the `status` counts, plus **pending** if any.
   For a 7-day report, the `summary` of the set in `list_rank_keyword_sets` (`improved`, `declined`, `top3`, `top10`, `ranking`, `avgPosition`, over `windowStart` to `windowEnd`) holds the same figures as stored by the last check; quote it as a cross-check.

6. **Winners and losers.** Rank the improved keywords by `change` and the declined ones by how negative it is, but weigh both by `impressions` (`previousImpressions` for decliners): a keyword sliding from 3 to 6 on 2,000 impressions matters more than one moving from 40 to 60 on 15. Treat a position built on fewer than about 10 impressions as noise and mark it, since one stray impression can move such an average by tens of places. List lost keywords separately, by `previousImpressions`.

7. **Which page ranks.** `rankingUrl` is the page Search Console showed most often for the keyword at the set's last check, not a per-period value. Group the set's keywords by `rankingUrl` to show which pages carry the set, and flag a keyword whose ranking page looks wrong for it (a blog post for a product keyword, the home page for a specific service, two keywords of one intent on different pages); the user knows which page should rank.

8. **Read the trend.** From `trend`, compare the first and last weeks of the period on `avgPosition`, `top3` and `top10`, and say whether the change came as a step on one date or built up gradually. A change in the `keywords` count per day means keywords were added or removed, or started or stopped getting impressions, so do not read it as movement.

9. **Beyond the tracked keywords (optional).** When the user wants to know what moved that nobody tracks, call `get_ranking_changes` with `siteUrl`, `dimension: "query"`, `startDate` and `endDate` set to the period's `startDate` and `endDate`, `comparisonStartDate` and `comparisonEndDate` set to `previousStartDate` and `previousEndDate`, and `limit: 50`. Its `improved`, `declined`, `new` and `lost` rows (with `key`, `position`, `prevPosition`, `positionDelta`, positive is better, and `clicks`) cover the whole property across all countries and devices, so they are not comparable one to one with a set limited to one market or device. Name a few untracked queries worth adding.

## Output

Start with one line per set: name, country and device, the period and the comparison period with their dates, and the newest day stored. Then, per set:

1. **Headline**: one or two sentences plus a compact table of the counts from step 5.
2. **Winners**: up to 10 rows.

   | Keyword | Position | Before | Change | Impressions | Clicks | Ranking page |
   |---|---|---|---|---|---|---|

3. **Losers**: the same table for declined keywords, then the lost keywords.
4. **Pages**: the pages that carry the most tracked keywords, and any keyword whose ranking page looks wrong.
5. **Trend**: two or three sentences on the daily trend.
6. **Lag note**: the newest day is about three days before today because only final Search Console data is stored, so the report runs to `period.endDate`, not to today; the newest days are complete, not partial.
7. End with 3 to 5 concrete next steps that name the keyword or page, for example strengthen a page whose keywords just reached 4 to 10, check a page whose keywords all fell together, or add an untracked query from step 9.

## Follow-up

For keywords that dropped sharply, offer the rank-drops skill. For sets full of keywords that never get impressions, offer the prune-keywords skill.

## Errors

If any tool call fails with an authentication or subscription error, tell the user to run `/mcp` to (re)authenticate, and that a GSC Wizard account with an active plan or trial plus a connected Google Search Console account is required; `get_account_info` shows the plan (`access.level`, "free" without an active plan or trial) and the connected Google accounts (`googleAccounts`). If the property is not found, re-run `list_sites` and suggest close matches rather than guessing. If a set id is not found, re-run `list_rank_keyword_sets`. If a period longer than 7 days is refused as not available yet, fall back to `days: 7` and say so. If a single tool fails for another reason, report the gap and complete the rest of the report.

## Grounding

Every number you report must name the tool it came from and the date range it covers, for example "clicks 12,340 (`get_site_summary`, 2026-09-21 to 2026-09-27)". For a table, state the tool and date range once on the line above it. If you derive a figure yourself (a delta, a sum, a share), say so and name the tool outputs it was computed from. Never report a number no tool returned.
