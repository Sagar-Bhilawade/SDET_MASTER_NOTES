# SQL Window Functions — The Senior Differentiator

## What it is

Window functions — `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `LEAD`/`LAG`, running totals — compute a value across a set of rows *related to the current row* without collapsing them the way `GROUP BY` does. This is the topic that separates mid-level SQL from senior SQL, because most candidates can't write them cold and interviewers know it.

---

## Why it matters

"Second-highest salary," "top order per customer," "rank within category," "running total" — these come up constantly and they're near-impossible to write cleanly *without* window functions. A candidate who reaches for `ROW_NUMBER() OVER (PARTITION BY ...)` on a "top-per-group" question immediately reads as someone who's written real, analytical SQL. Given your data-heavy BASES background, this is exactly where you can shine.

---

## The mental model

```sql
FUNCTION() OVER (PARTITION BY col ORDER BY col2)
```
- **`PARTITION BY`** = "restart the calculation for each group" (like GROUP BY, but rows are *kept*, not collapsed).
- **`ORDER BY`** (inside OVER) = "the order within each partition" that ranking/offset functions use.
- The row stays in the output — you just get an extra computed column.

**GROUP BY collapses; window functions annotate.** That one sentence is the whole concept.

---

## The ranking functions (know the differences cold)

```sql
SELECT name, amount,
  ROW_NUMBER() OVER (ORDER BY amount DESC) AS row_num,
  RANK()       OVER (ORDER BY amount DESC) AS rnk,
  DENSE_RANK() OVER (ORDER BY amount DESC) AS dense
FROM orders;
```

For amounts `100, 100, 90`:
| amount | ROW_NUMBER | RANK | DENSE_RANK |
|--------|:----------:|:----:|:----------:|
| 100 | 1 | 1 | 1 |
| 100 | 2 | 1 | 1 |
| 90 | 3 | **3** | **2** |

- **`ROW_NUMBER`** — always unique, arbitrary tie-break. "Give me exactly one per position."
- **`RANK`** — ties share a rank, then **skips** (1,1,3). "Olympic ranking."
- **`DENSE_RANK`** — ties share a rank, **no gap** (1,1,2). "Distinct positions."

**Which to use:** "second-highest *distinct* value" → `DENSE_RANK` (so tied firsts don't consume the #2 slot). "Exactly one row per customer" → `ROW_NUMBER`. Getting this choice right is the signal.

---

## The canonical problems

### Second-highest value
```sql
SELECT amount FROM (
  SELECT amount, DENSE_RANK() OVER (ORDER BY amount DESC) AS rnk
  FROM orders
) t WHERE rnk = 2;
```
Or without windows: `SELECT DISTINCT amount FROM orders ORDER BY amount DESC LIMIT 1 OFFSET 1;` — know both, mention the window version scales to "Nth per group."

### Top order per customer (top-N-per-group)
```sql
SELECT * FROM (
  SELECT o.*, ROW_NUMBER() OVER (
    PARTITION BY customer_id ORDER BY amount DESC) AS rn
  FROM orders o
) t WHERE rn = 1;
```
**This is THE window-function question.** "Most expensive order per customer," "latest login per user," "top product per category" — all this exact shape. Memorize it. Change `rn = 1` to `rn <= 3` for top-3-per-group.

### Running total
```sql
SELECT order_date, amount,
  SUM(amount) OVER (ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM orders;
```
A cumulative sum — the window is "all rows from the start up to this one."

### Compare to previous/next row (LEAD / LAG)
```sql
SELECT order_date, amount,
  LAG(amount) OVER (ORDER BY order_date) AS prev_amount,
  amount - LAG(amount) OVER (ORDER BY order_date) AS change
FROM orders;
```
`LAG` looks back, `LEAD` looks forward. For "day-over-day change," "gap between events." Impressive when it fits.

---

## Why you can't just use GROUP BY

```sql
-- This gives the MAX amount per customer, but LOSES which order it was:
SELECT customer_id, MAX(amount) FROM orders GROUP BY customer_id;
-- To get the whole ORDER ROW at that max, you need the window version above.
```
The reason window functions exist: GROUP BY can tell you the max value but not *which row* had it, and not while keeping the other columns. The `ROW_NUMBER` + filter pattern gets you the actual row. Explaining *why* GROUP BY isn't enough is the deep answer.

---

## Dialect notes

- Window functions are standard in **Postgres, Oracle, SQL Server, and MySQL 8+**. Older MySQL (5.x) lacks them — you'd fall back to correlated subqueries or self-joins. You've used Oracle and MySQL, so mention you know the version boundary.
- The subquery wrapper (`SELECT ... FROM (window query) WHERE rn = 1`) is needed because you can't filter on a window function in the same-level `WHERE` (it's computed after WHERE). Some dialects offer `QUALIFY` to skip the wrapper.

---

## Common interview Q&A

**Q: "RANK vs DENSE_RANK vs ROW_NUMBER?"**
The tie table above. RANK skips after ties, DENSE_RANK doesn't, ROW_NUMBER is always unique.

**Q: "Top N per group?"**
`ROW_NUMBER() OVER (PARTITION BY group ORDER BY metric DESC)`, filter `<= N` in an outer query. The signature answer.

**Q: "Why not GROUP BY?"**
GROUP BY collapses rows and loses the non-aggregated columns; you can't recover *which* row hit the max. Window functions keep the rows.

**Q: "Why the subquery wrapper?"**
Can't reference a window function in WHERE at the same level — it's computed after WHERE. Wrap and filter outside.

---

## Weak areas to watch

- **`PARTITION BY` = per-group restart** — the core idea. Don't confuse it with `GROUP BY` (which collapses).
- **DENSE_RANK for "Nth distinct," ROW_NUMBER for "exactly one per row"** — the choice is the test.
- **Memorize the top-N-per-group template** — it's the single most-asked window question.
- **You must wrap-and-filter** — window functions can't go in WHERE directly.
- **Know the MySQL 8+ boundary** — older versions need subqueries/self-joins instead.

---

## Key takeaway

> **Window functions are the senior SQL differentiator — most candidates can't write them cold, so reaching for `ROW_NUMBER() OVER (PARTITION BY ...)` on a top-per-group question instantly marks you as analytical.** Memorize the top-N-per-group template and the RANK/DENSE_RANK/ROW_NUMBER tie behavior, and be ready to explain *why* GROUP BY can't do it: it collapses the rows you need.
