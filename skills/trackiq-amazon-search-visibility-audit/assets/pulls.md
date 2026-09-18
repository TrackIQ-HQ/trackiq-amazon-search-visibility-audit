# The pulls

Order matters. Each step narrows the next, and pulling rank before you know
the demand set wastes most of the rows.

---

## 0. Confirm the account

`list_marketplaces` first. Several TrackIQ MCPs can be connected at once with
identical tool names and different brands behind them — match on the returned
name before pulling anything. A correct report about the wrong account is the
worst outcome this skill can produce.

## 1. Catalogue and revenue

`show_products` and `get_product_performance` for the period.

Drop ASINs with zero revenue. Keep the ASIN → revenue, orders and ASP map;
every gap figure later depends on that ASP.

If `account.md` names hero ASINs, scope to those. Otherwise take every
revenue-bearing ASIN and say in the report how many were covered.

## 2. Demand

`get_search_query_performance`, **one call per week**, for the ASINs in scope.

**Discover which weeks exist before using any of them.** The feed is
intermittent and a missing week returns `{"rows": []}`, not an error. Probe
each week in the period, record which were present, and scale by the weeks
actually observed. Assuming four weeks when two arrived halves every figure
in the report.

## 3. Rank

`get_keyword_rank` for the ASINs in scope.

Returns one row per keyword × ASIN × day. Take the **latest** reading per
`(asin, query)` and carry its date into the report. Never average.

**It truncates at `limit` silently.** If rows == limit, raise it and pull
again. An incomplete rank pull makes untracked terms look ranked-and-absent,
which points every recommendation in the wrong direction.

## 4. Paid

`get_search_terms` for the period, and `get_targets` for what is actively
bid on.

Search terms are what shoppers typed; targets are what you bid. A term can
appear in search terms through broad match without ever being a target —
that distinction drives the "stop bidding" recommendation, so keep both.

---

## The five ways these rows mislead

1. **SKU duplication.** One ASIN appears under several SKU rows — FBA and
   FBM, or a relisted SKU. Deduplicate on `(date, asin, query)` before any
   sum, or purchases and impressions are inflated by whatever the duplicate
   factor happens to be.

2. **Silent truncation.** Both the rank pull and the query pull cap at
   `limit` with no error and no flag. Always compare the row count to the
   limit you asked for.

3. **Missing weeks read as zero.** An absent SQP week returns an empty row
   set. Scaling by the weeks you assumed rather than the weeks you observed
   understates every figure, and the error is invisible because the totals
   still look plausible.

4. **Untracked looks identical to unranked.** Both produce no rank row. They
   are different findings with opposite actions — one says "measure this",
   the other says "fix this". Keep them apart from the first join.

5. **Broad match inflates paid presence.** A term showing in the search-term
   report is not a term you chose. Cross-reference `get_targets` before
   recommending that a bid be stopped, or the recommendation is to stop
   something that was never started.

---

## Before writing anything

Print, and check:

- ASINs in scope, and their revenue and ASP
- weeks requested vs weeks observed
- rows returned vs limit, for every pull
- the size of the revenue-bearing demand set
- how many queries landed in each of the four states, plus untracked

If any pull hit its limit, the report is not ready. Raise and re-pull.
