# Checks

Run all four groups before sending. The honesty checks are the ones that
matter in front of a client.

---

## 1. Data

- **Every pull's row count is below its limit.** If any equals the limit,
  the pull truncated. Raise and re-pull; do not report from it.
- **Weeks observed is stated**, and every scaled figure uses that number
  rather than the number of weeks in the period.
- **Deduplication happened before aggregation**, on `(date, asin, query)`.
  Spot-check one ASIN that has both FBA and FBM SKUs.
- **Rank readings carry their date**, and no rank anywhere is an average.
- **The four states plus untracked sum to the demand set.** If they do not,
  a query is double-counted or was dropped in a join.

## 2. Arithmetic

- **Category and ASIN revenue reconcile** to the account total for the
  period. A dropped ASIN removes its revenue from every denominator.
- **Share of voice is between 0 and 100%** per ASIN, and is computed within
  the tracked set — not against the category.
- **Gap revenue per query** equals `purchases × scale × ASP` with the ASIN's
  own ASP, not a blended one.
- **Every aggregate survives the detail beneath it.** The coverage
  percentage on the summary equals the count from the per-ASIN tables. Count
  from the tables; never estimate the headline.

## 3. Honesty

This is the group that gets the report believed or dismissed.

- **No sentence claims a competitor position.** The data shows your rank, not
  the results page. Search the draft for "competitor holds", "outranked by"
  and "position N is held" and remove them.
- **Untracked is never described as unranked.** Search for "not ranking" and
  confirm each one has a rank row proving it.
- **Every derived figure is labelled an estimate** where it appears, not only
  in a caveat panel at the end.
- **Share of voice says "of tracked queries"** every time it appears,
  including in any summary tile.
- **The gap is described as revenue transacting on queries where you are
  absent**, never as "lost revenue" or "revenue left on the table".
- **A section with no data self-hides.** No placeholder rows, no "N/A"
  tables, no apology paragraph.

## 4. Render

- **Open the report and read it top to bottom.** Do not claim visual
  verification without having looked.
- **No text below 12px** except uppercase letterspaced eyebrows.
- **Tables do not overflow their container** at 1180px. A clipped column is
  invisible in the source.
- **Every number in prose matches a cell in a table** somewhere in the
  report.
- **Nothing is stated twice.** Adding a summary tile usually orphans a
  sentence below it.

---

## The one that always bites

The coverage headline and the per-ASIN tables disagree, because the headline
was computed from the demand set and the tables were filtered again while
being written. Count the headline from the rendered rows, last, after the
tables are final.
