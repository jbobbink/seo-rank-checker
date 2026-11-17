---
name: track-keywords
description: 'Set up rank tracking for a Search Console property with the Rank Checker by GSC Wizard tools: review the keyword sets that already exist, propose keywords worth tracking from the property''s own search queries (striking-distance and money queries, each with the reason it was picked), confirm the country, device and keyword list with the user, then create a keyword set or add to one and explain when the first positions appear. Use when the user asks to track keywords, start rank tracking, add keywords to the Rank Checker, or which keywords to monitor.'
---

# Track Keywords

Set up rank tracking for one property: find out what is already tracked, suggest the keywords that are worth following and why, agree the country, device and list with the user, and create the keyword set (or add to an existing one). The user's request may contain a property, keywords they already have in mind, a market or device, or a theme (a product line, a service, a site section); use each of them and only suggest keywords to fill the gaps.

Before you start, keep in mind what the Rank Checker measures, and say it to the user if they seem to expect something else. Positions are the average position Google Search Console reports for the user's own property on real searches, stored once a day from final Search Console data, so the newest day is about three days old. It is not a live check of a results page and it shows no competitor rankings. A keyword that gets no impressions in a period has no position there and shows as not ranking.

## Steps

1. **Resolve the property.** If the user's request or the conversation names a property, use it. Site URLs use GSC formats: `sc-domain:example.com` or `https://www.example.com/`. Otherwise call `list_sites` and pick the obvious match; if several fit, ask which one. Skip a row with `readable: false` (the property is not verified in Search Console, so tell the user to verify it there) or `activated: false` (it has to be activated on the GSC Wizard dashboard first). A property shared with the user at `accessLevel` READ_ONLY can be read but not changed, so the set has to be created by its owner or a collaborator with FULL access.

2. **Check the existing sets.** Call `list_rank_keyword_sets` with `siteUrl`. For each set note `name`, `country` and `device` (null means all), `keywordCount`, `createdByYou` and `dataUpTo`. Sets belong to the property and are shared by everyone with access to it, so a colleague may already track what the user wants. A set holds at most 500 keywords and a property at most 50 sets, counting everyone's. If a set with the same country and device already exists, suggest adding to it rather than starting a parallel one, and read its keywords with `get_rank_keyword_data` (`siteUrl`, `setId`) so you do not propose them again.

3. **Find candidates in the property's own queries.** Call `query_top_queries` with `siteUrl`, `limit: 1000` and a `startDate` about 90 days before the latest settled day, leaving `endDate` out so the server uses the latest settled date. Each row in `queries` carries `query`, `clicks`, `impressions`, `ctr` (already a 0-100 percentage) and `position` (lower is better). Drop the queries already tracked in the set you will add to. Then pick, and keep the reason for each pick:
   - **Striking distance**: average position roughly 4 to 20 with real impressions. A move of a few places here changes clicks the most, so these are the keywords a rank tracker earns its keep on.
   - **Money queries**: queries with commercial or transactional intent (product or service names, "price", "buy", "best", "near me", comparisons) that already bring clicks, so the user sees early when they slip.
   - **Brand**: one to three branded queries as a baseline, if the user wants them; they rarely move, which makes a fall all the more telling.
   - Leave out one-off long phrases with a handful of impressions, misspellings and queries for things the site does not offer: they mostly show as not ranking and clutter the report.
   These rows cover all countries and devices together, so a query that is strong overall can still be weak in the market the set will track. For a borderline candidate, `get_query_performance` with `siteUrl` and the exact `query` shows its daily trend over the same dates, so you can tell a steady ranking from a single spike before proposing it. Query rows only include the queries Google lists; `coverage.hiddenClicksShare` says what share of clicks has no query row, so mention it when it is large.

4. **Propose the list and agree the settings.** Show the candidates in a compact table (query, clicks, impressions, position, why it was picked), grouped by reason, and suggest a set size the user can act on (30 to 150 keywords is a good start). Then ask the user to confirm, in one message:
   - the **country**: an ISO 3166-1 alpha-3 code such as NLD, GBR, USA or DEU, or "all";
   - the **device**: DESKTOP, MOBILE, TABLET or "all";
   - the **keyword list**, with any they want to add or drop;
   - for a new set, its **name** and the alerts: `emailAlerts` (default true, mails the person who created the set), `alertThreshold` (places a keyword must move before it alerts, 1 to 50, default 3) and `alertDirection` ("drops", the default, or "both").
   Country and device are fixed when a set is created and cannot be changed through these tools, so a set for another market is a new set. Never call the write tool before the user has confirmed.

5. **Create the set or add to it.** Call `add_rank_keywords` with `siteUrl` and `keywords` (at most 500 per call), and either:
   - `name` plus the optional `country`, `device`, `alertThreshold`, `alertDirection` and `emailAlerts` to create a new set, or
   - `setId` alone to add to an existing set. Passing any of the new-set settings together with `setId` is refused.
   Keywords are lowercased, trimmed and deduplicated, and one longer than 200 characters is skipped. Report `created`, `setName`, `added`, `alreadyInSet`, `tooLong`, `skippedOverCap` (keywords left out because the set reached 500) and `keywordCount`.

6. **Explain when the data appears.** Quote the result's `message`, then explain it:
   - A new set, and normally an existing set that just received keywords, is checked within the hour, and each new keyword is read with up to 16 months of Search Console history, so the trend is there at once rather than after weeks. When the message says the keywords wait for the set's next scheduled check, that check is at most a day away.
   - The newest day stored is about three days before today, because the check reads only final Search Console data.
   - Until the first check runs, `get_rank_keyword_data` answers with `emptyBecause` and the keywords have `status` "pending"; that is expected, not an error.

## Output

1. One line naming the property and what is already tracked (sets, their country and device, keyword counts).
2. The candidate table from step 4, with the date range and tool above it.
3. After the write: the set name, its country and device, and the counts from step 5.
4. A short note on when the first positions appear and what they mean (average position on real searches, final data about three days behind, no competitor rankings).

## Follow-up

Offer to come back once the first check has run: the rank-report skill gives the first report with history, and in a few weeks the prune-keywords skill finds keywords that never earn impressions.

## Errors

If any tool call fails with an authentication or subscription error, tell the user to run `/mcp` to (re)authenticate, and that a GSC Wizard account with an active plan or trial plus a connected Google Search Console account is required. `get_account_info` tells which part is missing: `access.level` is "paid" during a trial or plan and "free" without one, `googleAccounts` lists the connected Google accounts, and `apiKey.scope` is "read" when the connection may not change anything. If `add_rank_keywords` is refused or not available, the connection may have read-only access or the property may be shared with the user as read-only; say so and give the agreed list so the user or the property owner can add it in the Rank Checker. If a country code is refused, ask for an ISO 3166-1 alpha-3 code. If the property is not found, re-run `list_sites` and suggest close matches rather than guessing. If a tool says the Rank Checker is not available yet, report that message and stop.

## Grounding

Every number you report must name the tool it came from and the date range it covers, for example "clicks 12,340 (`get_site_summary`, 2026-09-21 to 2026-09-27)". For a table, state the tool and date range once on the line above it. If you derive a figure yourself (a delta, a sum, a share), say so and name the tool outputs it was computed from. Never report a number no tool returned.
