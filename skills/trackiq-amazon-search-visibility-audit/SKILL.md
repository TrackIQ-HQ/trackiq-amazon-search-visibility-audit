---
name: trackiq-amazon-search-visibility-audit
description: Audits where an Amazon brand's ASINs actually appear in search — organic rank per revenue-bearing query, which terms are bought but not ranked, which are ranked and still paid for, share of voice within the tracked set, and the revenue sitting behind the terms where the brand is absent or stranded on page two. Use when the user asks for a search visibility audit, an ASIN visibility check, organic rank coverage, where do we rank, paid versus organic overlap, share of voice, a cannibalisation check, or which keywords we are missing.
---

# TrackIQ: Amazon Search Visibility Audit

Answers one question per ASIN: **where do we actually show up, and what is
the gap worth?** Output is a branded HTML report — a coverage summary, then
per-ASIN query tables, then a priced gap list.

The finding is usually the overlap. Brands buy terms they already own and
leave the terms they don't rank on to competitors, and neither shows up in a
report that looks at paid and organic separately.

## Requires

- The TrackIQ MCP, for `list_marketplaces`, `show_products`,
  `get_product_performance`, `get_keyword_rank`,
  `get_search_query_performance`, `get_search_terms` and `get_targets`.
  Ask which brand and marketplace before pulling anything.
- **No third-party API.** Everything here comes from the TrackIQ MCP. There
  is no scraping step, no SERP vendor, no price-history provider.
- Nothing else. No filesystem, no shell, no internet.
- **Without the MCP:** ask for the rank-tracker export, the Search Query
  Performance export and the search-term report, plus revenue by ASIN.
  Every section except share-of-voice movement works from those.

## First run

Fill in a copy of `assets/account.example.md` saved as account.md beside the
skill. Every TrackIQ skill reads the same file, so an account already set up
for another TrackIQ report needs nothing added here.

1. **Brand and marketplace** — which account and catalogue to pull
2. **Hero ASINs** — the products the audit covers, if not the whole catalogue
3. **Delivery** — in-chat, file, Slack, n8n or email

If the runtime has no filesystem, print the same block and ask the user to
paste it into their project instructions once.

## Read first

- `assets/pulls.md` — the call sequence, and the five ways these rows
  mislead. Read before the first tool call.
- `assets/method.md` — the four states, share of voice, and the gap valuation
- `assets/checks.md` — the data, judgement and honesty checks
- `assets/account.example.md` — the first-run answers, filled in once
- `assets/trackiq-logo-white.png` — the masthead lockup
- `assets/trackiq-bug-white.png` — the footer bug

Copy `assets/report-template.html` and replace every `{{TOKEN}}`, repeating
the ASIN section per ASIN. Do not restyle it.

## Delivery

The report is produced in the chat first. Delivery is the last step and the
method comes from the Delivery block in account.md — never ask per run.

| Method | What to do | Needs |
|---|---|---|
| `in-chat` | Return the HTML. The default, and the fallback for every other method. | nothing |
| `file` | Write it beside the skill as `search-visibility-<YYYY-MM-DD>.html`. | a filesystem |
| `slack` | Post the coverage percentage and the priced gap as text, then upload the HTML. | a connected Slack tool |
| `n8n` | POST the HTML to the configured webhook, `Content-Type: text/html`. | network access |
| `email` | Hand it to the connected mail tool. | a connected mail tool |

Confirm before the first outward send of a session, fall back to in-chat
loudly when a method is unavailable, and never substitute a different
outward channel. Naming Slack or n8n here is configuration, not report
content — the report names no platform but TrackIQ and Amazon.

## Non-negotiables

1. **Rank is a point-in-time reading, not a period average.** `get_keyword_rank`
   returns one row per keyword × ASIN × day. State the date the rank was read
   and never average a rank across a month — a term that sat at 4 for
   twenty-nine days and 60 for one is not "rank 6".
2. **Absence in the tracker is not absence from the results.** A query with
   no rank row means it was never tracked, which is different from tracking
   it and finding nothing. Report the two states separately; conflating them
   invents a visibility problem that may not exist.
3. **Every gap figure is an estimate and is labelled one.** Revenue behind an
   untracked or unranked query is `query purchases × scale × ASIN ASP`, and
   the report states all three assumptions. Never present it as measured
   lost revenue.
4. **Never claim a competitor holds a position you did not observe.** The MCP
   reports your rank, not the full results page. Write "we do not rank in the
   tracked set", never "a competitor holds position 3".
5. **Deduplicate on `(date, asin, query)` before counting anything.** One ASIN
   appears under several SKU rows, and the rank pull truncates silently — if
   rows == limit, raise it and pull again.
6. **Paid and organic are joined on the exact lowercase query.** Near-misses
   are not matches: "grill cover xl" and "waterproof grill cover xl" are two
   terms, and treating them as one hides the overlap the report exists to
   find.
7. **Share of voice is impression share within the tracked set**, not of the
   whole category. Say so wherever the number appears, or a client will read
   it as market share.
8. **A query with no purchases anywhere is not a gap.** Filter to
   revenue-bearing demand before valuing anything, or the gap list fills with
   high-volume terms nobody buys from.
9. **Fewer qualifying queries means a shorter table.** Never pad an ASIN's
   section to a target row count.
10. **Never print `account_id`.** Refer to the account by name or as "your
    US Seller account".

## The actionable output

The report is the artifact; the four lists are the action. End by offering,
as plain text ready to paste:

- terms to **add to the rank tracker** (revenue-bearing, untracked)
- terms to **stop bidding on** (ranked page one, still paid, no incremental
  click share)
- terms to **start bidding on** (revenue-bearing, not ranked, not bought)
- ASINs whose **listing is the constraint** (impressions without clicks)

## Relationship to the other TrackIQ skills

`category-priority-keywords` decides **which** terms are worth tracking and
prices them. This one reports **how you are doing** on the terms you track,
and what the untracked ones are costing. Run that one on onboarding and
quarterly; run this one monthly, or before and after a listing change.

The daily and weekly reports carry a rank section covering movement on the
hero ASINs. This is the full read.

## Version

`trackiq-amazon-search-visibility-audit` v1.0.0 (2026-09-18).

If the user asks whether this skill is current, fetch
`https://trackiq.com/skills/registry.json`, compare the `version` field for
`trackiq-amazon-search-visibility-audit`, and if it is newer, give them the
download link and the one-line changelog. Do not fetch at any other time.
