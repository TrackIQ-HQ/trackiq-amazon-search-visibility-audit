# Method

Three jobs: place every revenue-bearing query into one of four states, size
the gap, and decide what to do about each state.

---

## 1. The demand set

Start from demand, not from the tracker. The tracker only knows what someone
already thought to add — auditing it against itself finds nothing.

1. **Pull Search Query Performance** for the ASINs in scope, per week, and
   aggregate by query: sum `asin_purchases`, `asin_impressions`,
   `asin_clicks`; take the max of `volume`; recompute impression share as
   `sum(asin_impressions) / sum(market_impressions)`.
2. **Filter to revenue-bearing demand.** At least one purchase across the
   observed weeks, or impression share ≥ 1%. A query nobody buys from is not
   a visibility gap however large its volume.
3. **Deduplicate on `(week, asin, query)`** before summing. One ASIN appears
   under several SKU rows and summing them inflates everything downstream.

That set is the denominator for the whole report. State its size.

## 2. The four states

Join three sources on the **exact lowercase query**, per ASIN:

- **organic rank** from `get_keyword_rank` (latest reading, with its date)
- **paid presence** from `get_search_terms` and `get_targets`
- **demand** from the set above

Every query lands in exactly one state:

| State | Organic | Paid | What it means |
|---|---|---|---|
| **Owned** | page 1 | not bought | Working. Defend it. |
| **Doubled** | page 1 | bought | You are paying for a position you already hold. The overlap. |
| **Bought** | page 2+, or not ranked | bought | Paid is carrying it. Fine, but it is rented. |
| **Absent** | page 2+, or not ranked | not bought | Demand you are not serving. The gap. |

**Untracked is its own row, not a fifth state.** A query with no rank row was
never measured. Report it as *not tracked* and count it separately — calling
it "not ranked" invents a problem and inflates the gap.

Page one is positions 1–16 unless the account's own convention differs; say
which was used.

## 3. Share of voice

Impression share within the tracked set:

```
sov(asin) = sum(asin_impressions) / sum(market_impressions)
            across the ASIN's revenue-bearing queries
```

This is share of the queries measured, **not** share of the category. Label
it that way every single time it appears. A client who reads it as market
share will quote it back in a board deck.

## 4. Sizing the gap

Only **Absent** and **untracked-with-demand** queries carry a gap figure.

```
asp(asin)             = asin revenue / asin orders
observed_purchases(q) = sum of asin_purchases across the weeks that exist
scale                 = days_in_period / (7 x weeks_observed)
gap_revenue(q)        = observed_purchases(q) x scale x asp(asin)
```

Three assumptions, all of which the report states:

1. Purchases on that query continue at the observed rate.
2. Every order on the query is worth the ASIN's average selling price.
3. The weeks observed are representative of the ones that were missing.

**This is not lost revenue.** It is the revenue currently transacting on
queries where the brand is absent from the tracked set. Some of it is already
yours through other queries. Say so in the caveat panel, not in a footnote.

## 5. Reading the states into actions

| Finding | Action |
|---|---|
| **Doubled**, page-one organic, paid click share under ~10% | Stop bidding. The paid spend is buying a click you were getting anyway. |
| **Doubled**, page-one organic, paid click share high | Keep bidding. You are holding a competitor off the slot. |
| **Absent** with purchases | Start bidding while the listing works toward rank. Two levers, not one. |
| **Absent**, untracked | Add to the tracker first. You cannot manage what is not measured. |
| Impressions high, clicks low | The listing is the constraint, not the rank. Main image, title, price, review count — in that order. |
| Clicks high, purchases low | The offer is the constraint. Price, availability, Buy Box. |

The last two are the ones a rank report normally misses, and they are often
the cheapest fixes on the list.
