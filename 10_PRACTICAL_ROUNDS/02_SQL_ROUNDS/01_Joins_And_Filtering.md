# SQL Joins & Filtering — Deep Dive

## What it is

The foundation of the live-SQL round: getting data out of multiple tables correctly and filtering it. Joins are where most SQL mistakes happen, so interviewers probe them hard. This file covers every join type, when each matters, and the filtering traps (NULL, `WHERE` vs `HAVING`). The reference schema and the broader bank live in the [problem bank](05_Problem_Bank_And_Gotchas.md).

---

## Why it matters

You did two years of real database validation on the legacy stack — this round should be a strength. But knowing concepts and writing correct SQL cold with someone watching are different skills. Joins are also where the *senior* signal lives: anyone can `INNER JOIN`, but knowing that `LEFT JOIN ... IS NULL` finds orphans, and that `NOT IN` with a NULL returns nothing, separates you from candidates who memorized syntax.

---

## The reference schema

```
customers(id, name, city, created_at)
orders(id, customer_id, amount, status, created_at)
products(id, name, price, category)
order_items(order_id, product_id, quantity)
```

---

## The join types (know all four + when each is right)

### INNER JOIN — only matching rows
```sql
SELECT o.id, c.name, o.amount
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id;
```
Returns orders that have a matching customer. Orders with no valid customer, and customers with no orders, are **dropped**. This is the default and the most common.

### LEFT JOIN — all left rows, matched or not
```sql
SELECT c.name, o.id
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id;
```
Every customer appears; those with no orders get NULL in the order columns. Use when you need "all X, plus their Y if any."

### RIGHT JOIN — all right rows (rarely used)
Mirror of LEFT; most people just flip the tables and use LEFT. Mention you know it exists.

### FULL OUTER JOIN — everything from both
All rows from both sides, NULLs where no match. Useful for reconciliation ("what's in A but not B, and B but not A"). *Note: MySQL doesn't support it directly — you union a LEFT and a RIGHT. Oracle/Postgres do. Know the dialect difference.*

**The visual model to say out loud:** "INNER is the intersection; LEFT keeps everything on the left; FULL keeps both sides."

---

## The orphan-detection pattern (the SDET classic)

*"Find orders whose customer no longer exists"* — real data-integrity validation you did on the job:
```sql
SELECT o.*
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.id
WHERE c.id IS NULL;
```
**The pattern: LEFT JOIN + `IS NULL` on the right table = "exists in left, missing in right."** Know this cold — it's the query you actually write to validate referential integrity, and interviewers love it because it proves you understand what LEFT JOIN produces (NULLs for non-matches), not just the syntax.

---

## Filtering traps

### WHERE vs HAVING
```sql
-- WHERE filters ROWS before grouping:
SELECT * FROM orders WHERE amount > 1000;

-- HAVING filters GROUPS after aggregation:
SELECT customer_id, COUNT(*) FROM orders
GROUP BY customer_id HAVING COUNT(*) > 5;
```
`WHERE` can't use aggregate functions (they don't exist yet at row-filter time). Aggregate conditions go in `HAVING`. Mixing this up is the most common grouping mistake.

### NULL comparisons — the silent killer
```sql
WHERE status = NULL      -- WRONG: always "unknown", returns nothing
WHERE status IS NULL     -- correct
WHERE status != 'SHIPPED' -- also excludes NULL rows! (NULL != 'SHIPPED' is unknown)
```
You cannot compare to NULL with `=` or `!=` — the result is "unknown," not true. Use `IS NULL` / `IS NOT NULL`. And `!= 'X'` silently drops NULL rows, which surprises people. **Naming this unprompted is a strong signal.**

### The NOT IN + NULL trap
```sql
-- DANGEROUS:
SELECT * FROM customers WHERE id NOT IN (SELECT customer_id FROM orders);
```
If the subquery returns even one NULL, the whole `NOT IN` returns **zero rows** — because `id != NULL` is unknown for every row. Use `NOT EXISTS` instead:
```sql
SELECT * FROM customers c
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);
```
This is the single most-loved SQL gotcha. Volunteer it.

---

## Filtering operators worth knowing

| Operator | Use |
|----------|-----|
| `BETWEEN a AND b` | Inclusive range |
| `IN (...)` | Membership (watch NULLs with NOT IN) |
| `LIKE 'A%'` / `'%x%'` | Pattern (% = any chars, _ = one char) |
| `IS NULL` / `IS NOT NULL` | Null checks — never `= NULL` |
| `EXISTS (subquery)` | "Does at least one row exist" — often faster than IN |

---

## Common interview Q&A

**Q: "INNER vs LEFT JOIN?"**
INNER keeps only matching rows; LEFT keeps all left rows with NULLs for non-matches. "If I need customers *with no* orders, I need LEFT + IS NULL — INNER would drop them."

**Q: "Find records in A with no match in B."**
LEFT JOIN + `WHERE b.key IS NULL`, or `NOT EXISTS`. Not `NOT IN` (the NULL trap).

**Q: "Why did your `!= 'X'` miss rows?"**
NULL rows fail the `!=` comparison (unknown, not true). Add `OR col IS NULL` if you want them.

**Q: "This join returned more rows than expected."**
A one-to-many join multiplies rows (one order → many order_items). Expected — aggregate or DISTINCT if you want one row per order.

---

## Weak areas to watch

- **LEFT JOIN + IS NULL for "missing in B"** — the orphan pattern, know it reflexively.
- **`= NULL` never works** — `IS NULL`. And `!= 'X'` silently drops NULLs.
- **`NOT IN` + NULL = zero rows** — use `NOT EXISTS`. The top gotcha.
- **WHERE before grouping, HAVING after** — aggregates only in HAVING.
- **One-to-many joins multiply rows** — know why the count inflated.
- **Alias and qualify** every column in a join (`o.id`, `c.name`) — unqualified reads sloppy and breaks on ambiguous names.

---

## Key takeaway

> **Joins are where the round is won or lost — and the senior move is the orphan pattern (LEFT JOIN + IS NULL) plus naming the NULL traps unprompted.** `= NULL` never works, `!= 'X'` drops NULLs, and `NOT IN` with a NULL returns nothing. Your two years of real DB validation should make this the round you look forward to.
