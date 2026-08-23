# SDET Database Validation Patterns

## What it is

The SQL an SDET actually writes on the job — not abstract puzzles, but the queries you run to *validate that the application did the right thing*: did the UI action persist correctly, are there orphaned records, do the counts reconcile, did the batch job complete. This is where your two years of legacy DB validation becomes an interview asset.

---

## Why it matters

When an interviewer asks "how do you validate at the database level?", they're separating testers who only click the UI from engineers who verify the *whole path* — UI action → API → database. The second kind catches bugs the UI hides (a value silently truncated, a status not updated, a duplicate created). Framing your SQL as *validation* rather than *querying* is what makes it senior. It ties directly to [Database Testing Strategy](../../03_SELENIUM_JAVA/05_SQL_AND_DB/02_Database_Testing_Strategy.md).

---

## The core validation patterns

### 1. Persistence check — did the write happen correctly?
After a UI/API action that creates or updates a record:
```sql
SELECT status, amount, updated_at
FROM orders
WHERE id = :orderId;
-- assert status = 'CONFIRMED', amount = expected, updated_at is recent
```
*The point: the API returning 200 doesn't prove it persisted the right values. Read the row back and assert the actual data. Truncation, wrong defaults, timezone bugs all hide behind a green response.*

### 2. Orphan / referential-integrity check
```sql
SELECT o.id FROM orders o
LEFT JOIN customers c ON o.customer_id = c.id
WHERE c.id IS NULL;
-- expect 0 rows; any result is a data-integrity bug
```
*A delete that didn't cascade, a broken foreign key — this finds it. Run it as a data-quality assertion.*

### 3. Count reconciliation
```sql
-- UI says "42 active users" — verify against source of truth
SELECT COUNT(*) FROM users WHERE status = 'ACTIVE';
```
*The classic UI-vs-DB mismatch: pagination bugs, caching, a filter applied in one place but not another. The DB count is the truth.*

### 4. Duplicate detection
```sql
SELECT email, COUNT(*) FROM users
GROUP BY email HAVING COUNT(*) > 1;
-- a unique-constraint gap or a double-submit bug
```

### 5. State-transition validation
```sql
-- after cancelling an order, confirm the full state is consistent
SELECT status, cancelled_at, refund_status
FROM orders WHERE id = :id;
-- status='CANCELLED' AND cancelled_at IS NOT NULL AND refund_status='INITIATED'
```
*Don't just check the one field the UI shows — check that all the related fields moved together. Partial state updates are a rich bug source.*

### 6. Data freshness / batch-job validation
```sql
SELECT MAX(processed_at) FROM daily_summary;
-- assert it's within the expected window — did the batch job run?
```

### 7. Cross-table consistency
```sql
-- order total should equal the sum of its line items
SELECT o.id, o.amount, SUM(oi.quantity * p.price) AS computed
FROM orders o
JOIN order_items oi ON oi.order_id = o.id
JOIN products p ON p.id = oi.product_id
GROUP BY o.id, o.amount
HAVING o.amount != SUM(oi.quantity * p.price);
-- any row here is a calculation bug
```
*This is a genuinely senior validation — asserting a derived value matches its components. Great to mention as "the kind of check I add for money-handling features."*

---

## How this runs in a framework (JDBC / test integration)

In the legacy stack you did this through JDBC inside the test:
```java
try (Connection conn = DriverManager.getConnection(url, user, pass);
     PreparedStatement ps = conn.prepareStatement(
         "SELECT status FROM orders WHERE id = ?")) {
    ps.setLong(1, orderId);
    ResultSet rs = ps.executeQuery();
    assertTrue(rs.next());
    assertEquals("CONFIRMED", rs.getString("status"));
}
```
*Key points to mention: `PreparedStatement` (not string concatenation — SQL injection + correctness), close connections (try-with-resources), and keep DB assertions targeted — validate what the UI can't show you, don't re-test everything.*

---

## What to validate where (the strategy answer)

| Layer | Validate | Don't over-validate |
|-------|----------|---------------------|
| UI | Rendering, wiring, user-visible state | Business logic (push down) |
| API | Response contract, status, business rules | — |
| **DB** | Persistence correctness, integrity, derived values, batch results | Things already asserted at API level |

*"I validate at the DB level what the upper layers can't reveal — silent truncation, integrity, derived calculations. I don't duplicate assertions the API already covers." That's the disciplined, senior framing.*

---

## Common interview Q&A

**Q: "How do you validate a feature at the DB level?"**
Read the affected row back after the action, assert the actual persisted values, and check integrity (orphans, related fields moved together). The response code isn't proof of correctness.

**Q: "The UI count doesn't match the DB — how do you debug?"**
DB count is the source of truth. Check for pagination limits, caching, a filter applied inconsistently, or a timezone/status boundary difference.

**Q: "Why not just trust the API response?"**
It can return success while persisting wrong data — truncation, wrong defaults, a missed field update. DB validation catches the silent bugs.

**Q: "How do you avoid SQL injection in test code?"**
`PreparedStatement` with bound parameters, never string concatenation. Same rule as production code.

---

## Weak areas to watch

- **Frame SQL as validation, not querying** — that framing is the seniority signal.
- **Read the row back after a write** — a 200 response is not proof of correct persistence.
- **Check related fields move together** — partial state updates hide behind the one field the UI shows.
- **Don't duplicate API-level assertions in the DB** — validate what the upper layers can't reveal.
- **`PreparedStatement`, always** — injection-safe and correct; string-concatenated SQL is a red flag.
- **Cross-table derived-value checks** (order total = sum of items) are a strong senior example — have one ready.

---

## Key takeaway

> **The SDET SQL that matters is validation SQL — read the row back to prove persistence, hunt orphans for integrity, reconcile counts against the UI, and assert derived values match their components.** Framing your queries as "verifying the whole path, catching what a 200 response hides" turns two years of legacy DB work into a clear seniority signal. Validate what the upper layers can't; don't duplicate what they already cover.
