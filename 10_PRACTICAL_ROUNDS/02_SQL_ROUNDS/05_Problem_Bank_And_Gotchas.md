# SQL Problem Bank & Gotchas — Rapid Fire

## What it is

A rapid-fire drill sheet: the common live-SQL prompts with their queries, plus the complete gotcha table. Use this to rehearse writing queries from memory against the reference schema, and to review every trap the morning of an interview. The deep dives are in the sibling files — [joins](01_Joins_And_Filtering.md), [aggregation](02_Aggregation_And_Grouping.md), [windows](03_Window_Functions.md), [validation](04_SDET_Validation_Patterns.md).

---

## Why it matters

Concepts don't help if you freeze on syntax with a schema on screen and someone watching. Drilling the *shapes* until they're reflexive is the whole game — you want to hear the prompt and have your hand already moving. This is the "recall out loud" file for SQL.

---

## The reference schema

```
customers(id, name, city, created_at)
orders(id, customer_id, amount, status, created_at)
products(id, name, price, category)
order_items(order_id, product_id, quantity)
```

---

## Rapid-fire prompts → queries

**"All orders over 1000, newest first"**
```sql
SELECT * FROM orders WHERE amount > 1000 ORDER BY created_at DESC;
```

**"Customers with more than 5 orders"**
```sql
SELECT customer_id, COUNT(*) FROM orders GROUP BY customer_id HAVING COUNT(*) > 5;
```

**"Each order with the customer name"**
```sql
SELECT o.id, c.name, o.amount FROM orders o JOIN customers c ON o.customer_id = c.id;
```

**"Customers with no orders"**
```sql
SELECT * FROM customers c
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);
```

**"Orphaned orders (customer deleted)"**
```sql
SELECT o.* FROM orders o
LEFT JOIN customers c ON o.customer_id = c.id WHERE c.id IS NULL;
```

**"Second-highest order amount"**
```sql
SELECT DISTINCT amount FROM orders ORDER BY amount DESC LIMIT 1 OFFSET 1;
-- or DENSE_RANK() = 2
```

**"Most expensive order per customer"**
```sql
SELECT * FROM (
  SELECT o.*, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY amount DESC) rn
  FROM orders o) t WHERE rn = 1;
```

**"Total revenue per city"**
```sql
SELECT c.city, SUM(o.amount) AS revenue
FROM customers c JOIN orders o ON o.customer_id = c.id
GROUP BY c.city ORDER BY revenue DESC;
```

**"Order count per customer including zeros"**
```sql
SELECT c.id, c.name, COUNT(o.id) AS cnt
FROM customers c LEFT JOIN orders o ON o.customer_id = c.id
GROUP BY c.id, c.name;
```

**"Duplicate customer emails"**
```sql
SELECT email, COUNT(*) FROM customers GROUP BY email HAVING COUNT(*) > 1;
```

**"Top 3 product categories by revenue"**
```sql
SELECT category, SUM(price) AS total FROM products
GROUP BY category ORDER BY total DESC LIMIT 3;
```

**"Failed vs passed count in one row"**
```sql
SELECT COUNT(CASE WHEN status='FAIL' THEN 1 END) AS failed,
       COUNT(CASE WHEN status='PASS' THEN 1 END) AS passed
FROM test_runs;
```

**"Order total vs sum of line items (find mismatches)"**
```sql
SELECT o.id FROM orders o
JOIN order_items oi ON oi.order_id = o.id
JOIN products p ON p.id = oi.product_id
GROUP BY o.id, o.amount HAVING o.amount != SUM(oi.quantity * p.price);
```

---

## The complete gotcha table (review before every interview)

| Gotcha | The catch | Fix |
|--------|-----------|-----|
| `WHERE` vs `HAVING` | Row filter vs group filter | Aggregates go in HAVING |
| `= NULL` | Always "unknown" → no rows | `IS NULL` |
| `!= 'X'` drops NULLs | NULL != 'X' is unknown, not true | Add `OR col IS NULL` |
| `NOT IN` + NULL subquery | Returns **zero** rows | Use `NOT EXISTS` |
| `COUNT(*)` vs `COUNT(col)` | `*` counts rows, `col` skips NULLs | `COUNT(col)` for "0 not 1" in LEFT JOIN |
| `AVG` ignores NULLs | Divides by non-null count | `SUM/COUNT(*)` to treat NULL as 0 |
| Non-grouped column in SELECT | Error in strict mode | Group it or aggregate it |
| RANK vs DENSE_RANK vs ROW_NUMBER | Tie handling differs (skip / no-skip / unique) | DENSE_RANK for "Nth distinct" |
| One-to-many join inflates rows | Order × items multiplies | Aggregate or DISTINCT |
| `INNER` drops non-matches | Loses "X with no Y" | LEFT JOIN + IS NULL |
| Window fn in WHERE | Computed after WHERE — error | Wrap in subquery, filter outside |
| Integer division | `5/2 = 2` in some DBs | Cast to decimal |
| `LIMIT` dialect | MySQL/PG only; Oracle uses `FETCH FIRST`/`ROWNUM` | Know your DB |

---

## Performance quick hits (if they ask "make it faster")

- **Index** the columns you join and filter on. "Check for an index on `customer_id`."
- **Avoid `SELECT *`** — fetch only needed columns.
- **Read the execution plan** (`EXPLAIN`) — where's the full table scan?
- **`EXISTS` often beats `IN`** for large subqueries (stops at first match).
- You're not a DBA — showing you know *where to look* is enough.

---

## Common interview Q&A

**Q: "Write it, then how do you know it's right?"**
Trace a small dataset out loud, name the edge cases (empty table, NULLs, ties). The testing instinct in the SQL round.

**Q: "Which dialect?"**
State it — you've used Oracle and MySQL. It matters for `LIMIT`/`FETCH FIRST`, FULL OUTER JOIN, and window-function availability (MySQL 8+).

**Q: "Subquery or JOIN?"**
Often interchangeable; JOIN is usually clearer and lets the optimizer work. Know how to rewrite one as the other — they sometimes ask.

---

## Weak areas to watch

- **Drill from memory** — reading these isn't enough; write them blind against the schema.
- **The gotcha table is the highest-value review** — most SQL-round losses are a NULL trap, not a missing concept.
- **Alias and qualify** every column in joins.
- **State the dialect** when it affects syntax.
- **Trace and edge-case your query** — you're the testing candidate even here.

---

## Key takeaway

> **Drill the shapes until they're reflexive and review the gotcha table before every interview — most SQL rounds are lost to a NULL trap, not a missing concept.** You have the real DB-validation experience; the only gap between you and a clean round is writing these from memory under a watchful eye. Rehearse out loud against the schema, and always name the NULL behavior unprompted.
