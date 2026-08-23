# SQL Aggregation & Grouping

## What it is

Everything about `GROUP BY`, the aggregate functions (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`), and the rules that trip people: what you can select alongside a group, `COUNT(*)` vs `COUNT(col)`, and combining aggregation with joins. This is the second-most-common SQL round topic after joins.

---

## Why it matters

Aggregation is how you answer the business questions testers actually run — "how many orders failed," "total revenue per region," "average response per endpoint." It's also full of subtle rules the interviewer uses to check real understanding: the `COUNT(*)` vs `COUNT(col)` NULL behavior, and the "every non-aggregated column must be in GROUP BY" rule are classic separators.

---

## The aggregate functions

| Function | Returns | NULL handling |
|----------|---------|---------------|
| `COUNT(*)` | Number of rows | Counts every row |
| `COUNT(col)` | Non-null values in col | **Skips NULLs** |
| `COUNT(DISTINCT col)` | Distinct non-null values | Skips NULLs, dedups |
| `SUM(col)` | Total | Ignores NULLs |
| `AVG(col)` | Average | Ignores NULLs (divides by non-null count!) |
| `MIN` / `MAX(col)` | Extremes | Ignores NULLs |

**The AVG trap:** `AVG(col)` divides the sum by the count of *non-null* values, not total rows. If you want NULLs treated as zero, `SUM(col)/COUNT(*)`. Subtle and asked.

---

## GROUP BY basics

```sql
SELECT customer_id, COUNT(*) AS order_count, SUM(amount) AS total
FROM orders
GROUP BY customer_id;
```
`GROUP BY` collapses rows sharing a value into one row per group, and aggregates the rest.

### The cardinal rule
**Every column in SELECT that isn't aggregated must be in GROUP BY.**
```sql
-- ERROR (in strict mode): name is neither grouped nor aggregated
SELECT customer_id, name, COUNT(*) FROM orders GROUP BY customer_id;

-- Correct:
SELECT customer_id, COUNT(*) FROM orders GROUP BY customer_id;
-- or join to customers and group by both id and name
```
*MySQL used to allow the loose version (returning an arbitrary name) — a notorious source of wrong results. Postgres/Oracle reject it. Know that strict is correct.*

---

## HAVING — filtering groups

```sql
SELECT customer_id, COUNT(*) AS cnt
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 5;
```
`WHERE` filters rows *before* grouping; `HAVING` filters groups *after*. You can't put an aggregate in `WHERE`. The order of execution:
```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```
Knowing this execution order explains *why* you can't use a SELECT alias in WHERE (SELECT runs after) but can in ORDER BY (runs after SELECT). A genuinely senior thing to state.

---

## Aggregation with joins (the real queries)

### Revenue per city
```sql
SELECT c.city, SUM(o.amount) AS revenue
FROM customers c
JOIN orders o ON o.customer_id = c.id
GROUP BY c.city
ORDER BY revenue DESC;
```

### Order count per customer INCLUDING zeros
```sql
SELECT c.id, c.name, COUNT(o.id) AS order_count
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id
GROUP BY c.id, c.name;
```
**The trap:** `COUNT(*)` here would count the customer row and return **1** for zero-order customers (the LEFT JOIN produces one all-NULL row). `COUNT(o.id)` counts non-null order ids → correctly returns **0**. This LEFT-JOIN + `COUNT(col)` combination is a favorite interview question because it tests two concepts at once.

---

## Common patterns

### Duplicate detection
```sql
SELECT email, COUNT(*) FROM users
GROUP BY email HAVING COUNT(*) > 1;
```
### Top N groups
```sql
SELECT category, SUM(price) AS total FROM products
GROUP BY category ORDER BY total DESC LIMIT 3;
```
### Count by status (test-result bucketing — your world)
```sql
SELECT status, COUNT(*) FROM test_runs GROUP BY status;
-- PASS 420, FAIL 12, SKIP 8
```
*Connect it: "this is exactly how I'd summarize a test run or bucket failures by type."*

### Conditional aggregation (pivot-ish)
```sql
SELECT
  COUNT(CASE WHEN status = 'FAIL' THEN 1 END) AS failures,
  COUNT(CASE WHEN status = 'PASS' THEN 1 END) AS passes
FROM test_runs;
```
*`CASE` inside an aggregate lets you count subsets in one pass — a slick, senior-looking technique.*

---

## Common interview Q&A

**Q: "COUNT(*) vs COUNT(column)?"**
`*` counts all rows; `COUNT(col)` skips NULLs. Critical in LEFT JOINs where you want 0, not 1, for non-matches.

**Q: "Why does this GROUP BY error?"**
A selected column isn't aggregated or grouped. Add it to GROUP BY or wrap it in an aggregate.

**Q: "WHERE or HAVING for `amount > 1000`?"**
WHERE — it's a row filter, not a group filter. HAVING is only for conditions on aggregates.

**Q: "Average is off — why?"**
`AVG` divides by non-null count. NULLs are excluded, not treated as zero. Use `SUM/COUNT(*)` if you want them as zero.

---

## Weak areas to watch

- **`COUNT(col)` skips NULLs; `COUNT(*)` doesn't** — the LEFT JOIN "0 vs 1" trap.
- **Every non-aggregated SELECT column must be in GROUP BY** — strict mode is correct; don't rely on MySQL's old leniency.
- **WHERE before GROUP BY, HAVING after** — aggregates only in HAVING.
- **`AVG` ignores NULLs in the denominator** — know when that's wrong for you.
- **Know the execution order** — it explains alias scope (why WHERE can't see a SELECT alias).
- **`CASE` inside aggregates** for conditional counts — a strong technique to have ready.

---

## Key takeaway

> **Aggregation's traps are `COUNT(*)` vs `COUNT(col)` on NULLs and the "non-aggregated columns must be grouped" rule — nail both and you've shown real understanding, not memorized syntax.** Know the FROM→WHERE→GROUP BY→HAVING→SELECT execution order cold; it explains everything about what you can filter and reference where. Bucket-by-status is your real automation work — say so.
