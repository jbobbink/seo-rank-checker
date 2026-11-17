---
name: rank-drops
description: 'Explain why keywords tracked in the Rank Checker by GSC Wizard dropped: which keywords fell and on what date, the daily history of the worst ones, whether the drop is limited to the set''s country and device, whether the ranking page changed or lost the query, whether the whole site moved, and a short list of probable causes stated with honest uncertainty. Use when the user asks why their rankings dropped, why a tracked keyword fell, what happened to a keyword, or received a Rank Checker drop alert and wants it explained.'
---

# Rank Drops

Find which tracked keywords fell, date each fall, and test the explanations these tools can actually test. The user's request may contain a property, a set, one or more keywords, a date or an alert they received; start from what they give and widen only as far as needed.

Keep the limits in view and state them in the answer: positions are the average position Google Search Console reports for the user's own property on real searches, stored daily from final data about three days behind. There is no live results page and no competitor data, so these tools cannot show who now ranks above the site; they can show when the position moved, where (market, device, page) and whether the rest of the site moved with it.

## Steps

1. **Resolve the property.** If the user's request or the conversation names a property, use it. Site URLs use GSC formats: `sc-domain:example.com` or `https://www.example.com/`. Otherwise call `list_sites` and pick the obvious match; if several fit, ask which one.

2. **Find the set and the drops.** Call `list_rank_keyword_sets` with `siteUrl` and use the set the user named, or every set whose `summary.declined` is above zero. Then call `get_rank_keyword_data` with `siteUrl`, `setId` and `days: 7` (use 28 when the user speaks of the last month or the drop is older). Keep the keywords with `change` of -1 or worse and those with `status` "lost". Order them by what is at stake: `previousImpressions` first, then the size of the fall. Set aside keywords with fewer than about 10 impressions in both periods as noise, since one stray impression moves such an average by tens of places. Note the `period` dates; every later call uses them so the windows line up.

3. **Date the worst ones.** For the three to five keywords with the most at stake, call `get_rank_keyword_data` with `siteUrl`, `setId`, `keyword` and `days: 91` (or 28 for a very recent drop). `history` has one row per day with `date`, `position`, `clicks` and `impressions` for the set's country and device; a missing day had no impressions. Read each series for its shape:
   - a **step** on one date, which points to something that happened on or just before it;
   - a **slide** over weeks, which points to slowly losing ground rather than one event;
   - a **swing** that recovered inside the window, which is volatility, not a drop;
   - **impressions falling with the position**, versus impressions holding while the position fell: the first can be lower demand or a smaller presence, the second is a real loss of place.
   When several keywords step on the same date, say so: a shared date is the strongest clue these tools give.

4. **Market and device, or everywhere.** A set may be limited to one country and device, while `get_query_performance` reads the whole property across all countries and devices. Call it with `siteUrl`, the exact `query` (the tracked keyword), `startDate` set to the `history` start and `endDate` set to `period.endDate`. Its `daily` rows carry `date`, `clicks`, `impressions`, `ctr` (a 0-100 percentage) and `position`. If the property-wide series held while the set's series fell, the drop is specific to that market or device; if both fell, it is not.

5. **Did the ranking page change?** `rankingUrl` from step 2 is the page Search Console showed most often for the keyword at the last check. Call `get_query_performance` again with the same dates and `pageUrl` set to that URL.
   - If the keyword's series on that page fell while the property-wide series from step 4 held, another page of the site took the query over: likely cannibalization or a page that replaced the old one. Ask the user whether the `rankingUrl` is the page they want to rank.
   - If `daily` is empty for that page over the earlier weeks, the page only recently became the ranking page; if it is empty for the recent weeks, the page lost the query.
   - If the page and the property fell together, the change concerns that page or the query as a whole.

6. **One page or the whole site?** Call `get_ranking_changes` twice with `siteUrl`, `startDate` and `endDate` set to the period's `startDate` and `endDate`, `comparisonStartDate` and `comparisonEndDate` set to `previousStartDate` and `previousEndDate`, and `limit: 50`: once with `dimension: "query"` and once with `dimension: "page"`.
   - Compare `counts.declined` with `counts.improved`. Many more decliners than improvers across the property points to a site-wide movement (a Google update, a technical problem, a sitewide change); a balanced picture with the tracked keywords among the few decliners points to something local.
   - Look for the tracked keywords' `rankingUrl` pages in the page buckets by `key`: a ranking page in `declined` or `lost` is the unit to investigate, and a new page in `new` while an old one is in `lost` points to a URL change or a migration.
   - `positionDelta` is prevPosition minus position, so negative is a decline. These rows cover all countries and devices, and query rows only include the queries Google lists.

7. **Probable causes.** Build a short list from what steps 3 to 6 showed, most probable first, each with its evidence and how sure you are. Typical candidates:
   - another page of the site took the query (step 5);
   - a change to the ranking page itself: content, title, internal links, a redirect, noindex or removal. These tools cannot see page content, so ask the user what changed on or just before the step date;
   - a site-wide movement on the same date (step 6): a Google ranking update or a technical problem. These tools do not list Google updates, so suggest checking Google's Search Status Dashboard for that date;
   - a market or device specific change (step 4);
   - lower demand or a changed results page: impressions fell with the position, or the position fell while clicks held;
   - noise: few impressions, or a swing that already recovered.
   Say plainly when nothing fits. Never name a cause as certain unless the data shows it, and never claim to know what a competitor did.

## Output

State the property, the set (with country and device) and the period and comparison dates on the first line. Then:

1. **What dropped**: a table of the falling keywords.

   | Keyword | Position | Before | Change | Impressions before | Status | Ranking page |
   |---|---|---|---|---|---|---|

2. **When**: per worst keyword, two or three sentences on the shape of its daily history and the date it moved, with shared dates called out.
3. **Where**: whether the drop is limited to the set's market or device, tied to one page, or site-wide, with the evidence from steps 4 to 6.
4. **Probable causes**: the list from step 7, each marked likely, possible or ruled out.
5. **What to do**: 3 to 5 concrete actions naming the keyword and page, for example consolidate the two pages trading the query, restore what changed on the page on the step date, or watch a keyword with too few impressions to judge for another week.

Add one line on the lag: the newest stored day is about three days before today, so a fall in the last couple of days cannot be seen yet.

## Follow-up

Offer the rank-report skill for the full picture of the set, and the track-keywords skill when step 6 turned up untracked queries that matter.

## Errors

If any tool call fails with an authentication or subscription error, tell the user to run `/mcp` to (re)authenticate, and that a GSC Wizard account with an active plan or trial plus a connected Google Search Console account is required. If the property is not found, re-run `list_sites` and suggest close matches rather than guessing. If `get_query_performance` returns an empty `daily` with `emptyBecause`, the exact query has no rows in that window; say so instead of reading it as zero. If a single tool fails for another reason, report the gap and complete the analysis with the remaining data.

## Grounding

Every number you report must name the tool it came from and the date range it covers, for example "clicks 12,340 (`get_site_summary`, 2026-09-21 to 2026-09-27)". For a table, state the tool and date range once on the line above it. If you derive a figure yourself (a delta, a sum, a share), say so and name the tool outputs it was computed from. Never report a number no tool returned.
