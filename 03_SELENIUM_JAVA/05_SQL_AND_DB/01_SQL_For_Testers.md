# SQL for Testers - Queries You Actually Need

## What This Covers

The SQL that matters for backend validation — joins, aggregates, subqueries, and the query patterns interviewers ask you to write on the spot.

---

## Why It Matters (Interview Context)

SQL questions come up in almost every SDET interview that involves backend testing, and they're usually **written live**. Common prompts:

> "Find the second-highest salary."
> "Find duplicate records in a table."
> "Write a query joining orders and customers where the order failed."

These are practical checks. You did database validation for two years on Track 2, so this is a strength — but only if the queries come out cleanly under pressure.

---

## Your 30-Second Interview Answer

> "For test validation I'm mostly doing joins to pull related data, aggregates to verify counts and totals against what the UI shows, and subqueries for things like finding records without a matching child row. The join type distinction matters most — INNER only returns matches, LEFT keeps everything from the left side, and a LEFT JOIN with a NULL check is how you find orphaned or missing relationships, which is a common data integrity bug."

---

## Your 2-Minute Deep Answer

> "**Joins — the core of validation queries:**
>
> ```sql
> -- INNER: only rows matching in both
> SELECT o.order_id, c.name, o.total
> FROM orders o
> INNER JOIN customers c ON o.customer_id = c.id;
>
> -- LEFT: all orders, even those without a customer
> SELECT o.order_id, c.name
> FROM orders o
> LEFT JOIN customers c ON o.customer_id = c.id;
>
> -- LEFT + NULL check: find orphans — the data-integrity pattern
> SELECT o.order_id
> FROM orders o
> LEFT JOIN customers c ON o.customer_id = c.id
> WHERE c.id IS NULL;
> ```
>
> That last one is the pattern worth having ready. Orders referencing a customer that doesn't exist is exactly the kind of integrity bug a UI test would never surface — the page just renders a blank name.
>
> **RIGHT JOIN** is the mirror of LEFT and rarely used in practice; you'd reorder the tables instead. **FULL OUTER JOIN** returns everything from both sides — not supported in MySQL, where you'd union a LEFT and a RIGHT.
>
> ---
>
> **Aggregates and GROUP BY:**
>
> ```sql
> SELECT customer_id, COUNT(*) AS order_count, SUM(total) AS lifetime_value
> FROM orders
> WHERE status = 'COMPLETED'
> GROUP BY customer_id
> HAVING COUNT(*) > 5;
> ```
>
> **WHERE versus HAVING** is a standard question: WHERE filters rows *before* grouping, HAVING filters groups *after*. You can't use an aggregate in WHERE, which is why HAVING exists.
>
> Order of execution is worth knowing: `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY`. That explains why you can't reference a SELECT alias in WHERE — SELECT hasn't run yet.
>
> **`COUNT(*)` versus `COUNT(column)`:** `COUNT(*)` counts rows; `COUNT(column)` skips NULLs. That difference silently changes results and is a classic gotcha.
>
> ---
>
> **Finding duplicates — asked constantly:**
>
> ```sql
> SELECT email, COUNT(*)
> FROM users
> GROUP BY email
> HAVING COUNT(*) > 1;
> ```
>
> Group by the column that should be unique, keep groups with more than one row. If a unique constraint is supposed to exist, this query finding rows *is* the bug.
>
> ---
>
> **Second-highest value — the classic:**
>
> ```sql
> -- Portable
> SELECT MAX(salary) FROM employees
> WHERE salary < (SELECT MAX(salary) FROM employees);
>
> -- Nth highest, modern SQL
> SELECT DISTINCT salary
> FROM employees
> ORDER BY salary DESC
> LIMIT 1 OFFSET 1;
>
> -- Window function version — handles ties properly
> SELECT salary FROM (
>     SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
>     FROM employees
> ) t WHERE rnk = 2;
> ```
>
> Worth knowing more than one. The `DENSE_RANK` version is the strongest answer because it handles duplicate salaries correctly — with `LIMIT/OFFSET`, two people on the top salary means "second highest" returns the top value again.
>
> ---
>
> **Subqueries:**
>
> ```sql
> -- IN
> SELECT * FROM users
> WHERE id IN (SELECT user_id FROM orders WHERE total > 1000);
>
> -- EXISTS — usually faster on large sets
> SELECT * FROM users u
> WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
>
> -- NOT EXISTS: users who never ordered
> SELECT * FROM users u
> WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
> ```
>
> **`NOT IN` has a NULL trap worth knowing.** If the subquery returns any NULL, `NOT IN` returns no rows at all — because comparing to NULL is unknown. `NOT EXISTS` doesn't have that problem. That's a genuinely good thing to mention; it produces a "query returns nothing and I don't know why" situation that's hard to debug.
>
> ---
>
> **Window functions:**
>
> ```sql
> SELECT
>     order_id,
>     customer_id,
>     total,
>     ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY created_at DESC) AS rn
> FROM orders;
> ```
>
> `ROW_NUMBER` with `PARTITION BY` gets you "the most recent order per customer" — filter `WHERE rn = 1`. That's a common validation need and awkward without window functions.
>
> `ROW_NUMBER` gives unique numbers, `RANK` leaves gaps after ties, `DENSE_RANK` doesn't.
>
> ---
>
> **Things I check when validating data:**
>
> ```sql
> -- Nulls where they shouldn't be
> SELECT COUNT(*) FROM orders WHERE customer_id IS NULL;
>
> -- Referential integrity
> SELECT COUNT(*) FROM orders o
> LEFT JOIN customers c ON o.customer_id = c.id
> WHERE c.id IS NULL;
>
> -- Value ranges
> SELECT * FROM orders WHERE total < 0;
>
> -- Duplicates on a supposedly unique field
> SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;
> ```
>
> These are the queries I'd run when a bug report says data looks wrong — they find the shape of the problem fast."

---

## Query Patterns to Have Ready

| Need | Pattern |
|------|---------|
| Orphaned records | `LEFT JOIN ... WHERE right.id IS NULL` |
| Duplicates | `GROUP BY col HAVING COUNT(*) > 1` |
| Second highest | `DENSE_RANK() OVER (ORDER BY x DESC)` = 2 |
| Never-related rows | `NOT EXISTS (SELECT 1 FROM ...)` |
| Latest per group | `ROW_NUMBER() OVER (PARTITION BY ...)` = 1 |
| Filter groups | `HAVING` (not `WHERE`) |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Database validation was a real part of that role, not an occasional check. The application had substantial backend processing, so verifying the UI showed one thing while the database contained something else was where a lot of genuine bugs surfaced.
>
> The queries I ran most were joins pulling related data to confirm what the UI displayed matched what was stored, and aggregates verifying counts and totals. The pattern I'd single out is `LEFT JOIN` with a `IS NULL` check for orphaned records — orders referencing a customer that no longer exists. That's exactly the kind of integrity bug a UI test can't find, because the page just renders a blank name and looks fine.
>
> The trap I'd mention is `NOT IN` with NULLs. If the subquery returns any NULL, `NOT IN` returns nothing at all — the comparison is unknown rather than false. You get an empty result set with no error and no obvious reason. `NOT EXISTS` doesn't have that behaviour, so I default to it.
>
> The other one that catches people is `COUNT(*)` versus `COUNT(column)` — the second skips NULLs, so your totals silently differ."

---

## Common Interview Questions & Answers

### Q1: "Difference between INNER and LEFT JOIN?"

**A:** "INNER returns only rows with a match in both tables. LEFT returns everything from the left table, with NULLs where there's no match. For testing, LEFT JOIN with an `IS NULL` check on the right side is how you find orphaned records — that's a data integrity bug UI tests can't catch."

### Q2: "WHERE or HAVING?"

**A:** "WHERE filters rows before grouping; HAVING filters groups after. You can't use an aggregate in WHERE because grouping hasn't happened yet. The execution order is FROM, WHERE, GROUP BY, HAVING, SELECT, ORDER BY — which also explains why you can't reference a SELECT alias in WHERE."

### Q3: "Find the second-highest salary."

**A:**
```sql
SELECT salary FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) t WHERE rnk = 2;
```
"I'd use `DENSE_RANK` rather than `LIMIT 1 OFFSET 1` because it handles ties correctly — if two people share the top salary, the offset version returns the top value again."

### Q4: "Find duplicate emails."

**A:**
```sql
SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;
```
"And if there's supposed to be a unique constraint on that column, this query returning rows is itself the bug."

### Q5: "What's the problem with `NOT IN`?"

**A:** "NULLs. If the subquery returns any NULL, `NOT IN` returns no rows at all, because comparing anything to NULL is unknown rather than false. You get an empty result with no error and no obvious cause. `NOT EXISTS` handles it correctly, so I default to that."

---

## Weak Areas to Watch

❌ **Don't use `NOT IN` with a nullable subquery**
→ Silently returns nothing.

❌ **Don't confuse `COUNT(*)` and `COUNT(col)`**
→ The second skips NULLs.

❌ **Don't use `LIMIT/OFFSET` for Nth-highest**
→ Breaks on ties. `DENSE_RANK` is correct.

❌ **Don't put aggregates in WHERE**
→ That's what HAVING is for.

---

## Key Takeaway for Interview

These get written live, so the patterns need to be fluent.

The validation pattern worth leading with:

> **"`LEFT JOIN` with an `IS NULL` check finds orphaned records — orders pointing at customers that don't exist. UI tests can't catch that; the page just renders a blank field and looks fine."**

And the trap that shows real SQL experience:

> **"`NOT IN` returns nothing if the subquery contains a NULL, because the comparison is unknown rather than false. Empty result, no error, no obvious cause. I default to `NOT EXISTS`."**

---

## Checklist

✅ Join types and when each applies
✅ Orphan-detection pattern fluent
✅ Duplicate-detection pattern fluent
✅ Nth-highest with `DENSE_RANK`
✅ WHERE vs HAVING and execution order
✅ `NOT EXISTS` over `NOT IN`
✅ Window functions for per-group latest
✅ `COUNT(*)` vs `COUNT(col)` distinction

---

**Remember:** The bugs worth finding are the ones the UI renders as a blank field. 🎯
