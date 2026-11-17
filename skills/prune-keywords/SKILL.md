---
name: prune-keywords
description: 'Clean up a Rank Checker by GSC Wizard keyword set: find tracked keywords that never get impressions for the set''s country and device, keywords that were lost and stayed lost, and keywords that drifted out of relevance or rank with the wrong page, show them with the evidence, and remove them only after the user confirms the exact list, warning that a removed keyword''s stored history is deleted. Use when the user asks to clean up, prune or tidy their tracked keywords, remove dead keywords, or make room in a full keyword set.'
---

# Prune Keywords

Find the tracked keywords that no longer earn their place in a keyword set, show the evidence, and remove the ones the user confirms. The user's request may contain a property, a set, a reason (the set is full, the report is cluttered, a product was discontinued) or keywords they already want gone; honor each.

Removing a keyword deletes its stored position history with it, and that cannot be undone. Adding the keyword again reads its history from Search Console afresh, but Search Console keeps only about 16 months, while the Rank Checker keeps stored days for up to two years, so older days are lost for good. Treat every removal as permanent.

## Steps

1. **Resolve the property.** If the user's request or the conversation names a property, use it. Site URLs use GSC formats: `sc-domain:example.com` or `https://www.example.com/`. Otherwise call `list_sites` and pick the obvious match; if several fit, ask which one. A property shared with the user at `accessLevel` READ_ONLY cannot be changed; say so before doing the analysis.

2. **Pick the set.** Call `list_rank_keyword_sets` with `siteUrl` and use the set the user named, or ask when there are several. Note `name`, `country`, `device`, `keywordCount` (a set holds at most 500) and `createdByYou`. Sets are shared by everyone with access to the property, so when `createdByYou` is false, point out that the set was made by someone else, who also loses the history of anything removed.

3. **Read the long view.** Call `get_rank_keyword_data` with `siteUrl`, `setId` and `days: 485`, the longest period, which covers everything Search Console keeps. In `keywords`, a row with `impressions` of 0 had no impressions for the set's country and device in the whole period. Check `period.storedFrom` against `period.startDate`: when the set's history starts later, "never" means "not since `storedFrom`", and say so.

4. **Read the recent view.** Call it again with `days: 91`. It compares the last 91 days with the 91 before (`period.comparable` false means there is no comparison). Note each keyword's `status`, `position`, `impressions`, `previousImpressions` and `rankingUrl`.

5. **Sort the candidates into groups**, each with its evidence:
   - **Never seen**: no impressions in the 485-day period. These show as not ranking in every report and say nothing.
   - **Lost and gone**: `status` "lost" or "not_ranking" in the 91-day view while the 485-day view shows past impressions. These may be worth keeping as a record of a loss the user wants to win back, so ask rather than assume.
   - **Far out of reach**: a `position` beyond about 50 on a handful of impressions over both periods.
   - **Wrong intent or page**: the `rankingUrl` is a page that does not serve the keyword, or the keyword is about a product, service or market the user no longer offers. Only the user can judge relevance, so present these as questions.
   - Never propose a keyword with `status` "pending": it has not been checked yet.

6. **Rule out the wrong market.** For a keyword in the never-seen group of a set limited to one country or device, `get_query_performance` with `siteUrl` and the exact `query` (no dates, so the last 28 settled days) shows whether it gets impressions anywhere on the property. If it does, the keyword is alive in another market or device: suggest tracking it in a set for that market (add it there first with `add_rank_keywords`, after the user agrees) rather than simply deleting it.

7. **Confirm the exact list.** Show the proposed removals as one table per group and ask the user to confirm the exact keywords to remove, stating in the same message that their stored position history is deleted and cannot be restored. Accept edits to the list. Do not call the write tool on a summary like "remove the dead ones"; it needs the confirmed list.

8. **Remove.** Call `remove_rank_keywords` with `siteUrl`, `setId` and `keywords` set to the confirmed list (at most 500 per call). Keywords are matched the way they are stored (lowercased, trimmed, inner spaces collapsed). Report `removedCount`, the `removed` keywords, any `notFound` ones and the remaining `keywordCount`, and quote `message`. Removing every keyword leaves an empty set, which stays until it is deleted in the Rank Checker in GSC Wizard.

## Output

1. One line naming the property, the set (country, device, creator) and the two periods with their dates.
2. One table per group from step 5:

   | Keyword | Impressions (485 days) | Impressions (last 91) | Position | Status | Ranking page | Why |
   |---|---|---|---|---|---|---|

3. The confirmation question with the history warning.
4. After removal: the counts from step 8 and the room now left in the set (500 minus `keywordCount`).

## Follow-up

Offer the track-keywords skill to fill the freed room with striking-distance and money queries from the property's own data, and the rank-report skill for a cleaner report on what remains.

## Errors

If any tool call fails with an authentication or subscription error, tell the user to run `/mcp` to (re)authenticate, and that a GSC Wizard account with an active plan or trial plus a connected Google Search Console account is required. If `remove_rank_keywords` is refused or not available, the connection may have read-only access or the property may be shared with the user as read-only; say so and give the confirmed list so the property owner or a collaborator with full access can remove the keywords in the Rank Checker. If a period longer than 7 days is refused as not available yet, use `days: 7` and say the analysis covers only the last week, which is too short to call a keyword dead. If the property is not found, re-run `list_sites` and suggest close matches rather than guessing.

## Grounding

Every number you report must name the tool it came from and the date range it covers, for example "clicks 12,340 (`get_site_summary`, 2026-09-21 to 2026-09-27)". For a table, state the tool and date range once on the line above it. If you derive a figure yourself (a delta, a sum, a share), say so and name the tool outputs it was computed from. Never report a number no tool returned.
