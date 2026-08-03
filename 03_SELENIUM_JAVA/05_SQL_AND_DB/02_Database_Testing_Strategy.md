# Database Testing - Strategy & JDBC Integration

## What This Covers

Connecting to a database from a test framework, what's worth validating there, and where database testing helps versus where it creates coupling.

---

## Why It Matters (Interview Context)

Database validation is a genuine differentiator — many UI automation engineers have never done it. You did it for two years, so it's worth leading with.

The judgement question underneath: **database assertions couple your tests to the schema.** Knowing when that's worth it, and when you should validate through the API instead, is the senior layer.

---

## Your 30-Second Interview Answer

> "I use JDBC from the test framework to verify what actually persisted, because the UI showing success doesn't prove the write happened correctly. The highest-value checks are data integrity — orphaned records, nulls where they shouldn't be — and verifying backend processing that has no UI at all. The trade-off is that database assertions couple tests to the schema, so I'd prefer validating through an API where one exists and reserve direct queries for things the API doesn't expose."

---

## Your 2-Minute Deep Answer

> "**JDBC from a test framework:**
>
> ```java
> public class DatabaseUtil {
>     private static Connection getConnection() throws SQLException {
>         return DriverManager.getConnection(
>             ConfigReader.get("db.url"),
>             ConfigReader.get("db.user"),
>             ConfigReader.get("db.password"));
>     }
>
>     public static <T> T queryForObject(String sql, RowMapper<T> mapper, Object... params) {
>         try (Connection conn = getConnection();
>              PreparedStatement ps = conn.prepareStatement(sql)) {
>
>             for (int i = 0; i < params.length; i++) {
>                 ps.setObject(i + 1, params[i]);
>             }
>
>             try (ResultSet rs = ps.executeQuery()) {
>                 return rs.next() ? mapper.map(rs) : null;
>             }
>         } catch (SQLException e) {
>             throw new FrameworkException("Query failed: " + sql, e);
>         }
>     }
> }
> ```
>
> Three things worth noting:
>
> **try-with-resources** on connection, statement, and result set. Leaked connections exhaust the pool over a long suite, and the failure — 'cannot get connection' partway through — looks like an infrastructure problem rather than a code bug.
>
> **`PreparedStatement`, not string concatenation.** Partly SQL injection, but mainly correctness — a test data value containing an apostrophe breaks a concatenated query. That happens more often than injection does in a test framework.
>
> **Credentials from config**, never hardcoded.
>
> For a real framework I'd use a connection pool like HikariCP rather than opening a connection per query, and Spring's `JdbcTemplate` if Spring is already in the stack — it removes most of this boilerplate.
>
> ---
>
> **What's actually worth validating:**
>
> **1. The write actually happened, and correctly**
> ```java
> checkoutPage.completeOrder();
> Order order = DatabaseUtil.queryForObject(
>     "SELECT * FROM orders WHERE id = ?", Order::from, orderId);
>
> assertThat(order.getStatus()).isEqualTo("CONFIRMED");
> assertThat(order.getTotal()).isEqualByComparingTo("99.99");
> ```
>
> The UI showing "Order confirmed" doesn't prove the row is correct. A write can partially succeed, round a decimal wrong, or land with a stale status. That's a real class of bug the UI can't reveal.
>
> **2. Backend processing with no UI**
>
> Scheduled jobs, batch imports, data transformations, archival. If it doesn't surface in a screen, database validation is the only way to test it.
>
> **3. Data integrity**
> ```sql
> SELECT COUNT(*) FROM orders o
> LEFT JOIN customers c ON o.customer_id = c.id
> WHERE c.id IS NULL;
> ```
> Orphaned records, unexpected nulls, duplicate rows on unique fields, values out of range.
>
> **4. UI-to-database consistency**
>
> The dashboard shows 47 active users; does the database agree? Aggregation bugs, caching bugs, and timezone bugs all show up here and nowhere else.
>
> ---
>
> **The trade-off worth naming:**
>
> Database assertions couple your tests to the **schema**. A column rename breaks test code that has nothing to do with the feature that changed. That's real maintenance cost.
>
> So my preference order:
>
> 1. **API validation** if an endpoint exposes what you need — that's a contract designed to be consumed, and it's more stable than the schema
> 2. **Direct database query** for integrity checks and processing the API doesn't expose
> 3. **UI validation** for what the user actually sees
>
> The rule I'd apply: query the database for things the API genuinely doesn't expose, not as a shortcut around the API.
>
> ---
>
> **Database as test data setup:**
>
> Seeding directly into the database is faster than the API and much faster than the UI. But it bypasses application logic — validation, derived fields, triggers, side effects — so you can create states the application itself would never produce, and then test against something impossible.
>
> I'd use API seeding by default and direct database inserts only where no endpoint exists and the shortcut is genuinely worth the schema coupling.
>
> ---
>
> **Cleanup:**
>
> ```java
> @AfterMethod(alwaysRun = true)
> public void cleanup() {
>     try {
>         DatabaseUtil.execute("DELETE FROM orders WHERE test_run_id = ?", testRunId);
>     } catch (Exception e) {
>         logger.warn("Cleanup failed: {}", e.getMessage());
>     }
> }
> ```
>
> Tagging test-created rows with a run identifier makes cleanup precise and lets a scheduled sweep catch anything a crashed run left behind. And cleanup never fails the test — that's a teardown concern the developer can't act on."

---

## What to Validate Where

| Check | Best level | Why |
|-------|-----------|-----|
| User-visible behaviour | UI | That's what users experience |
| Business logic, validation | API | Fast, stable contract |
| Write correctness | DB | UI success ≠ correct persistence |
| Integrity, orphans, nulls | DB | Invisible to UI and API |
| Batch jobs, no UI | DB | Only option |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Database validation was a substantial part of the legacy role, because a lot of the application's real work happened in the backend rather than the screens. We connected via JDBC from the framework and verified what actually persisted after UI and API operations.
>
> The category of bug that justified it was writes that looked successful and weren't — the UI shows a confirmation while the row landed with a stale status or a rounded total. No amount of UI testing finds that.
>
> Integrity checks were the other high-value area. Orphaned records, nulls in columns that shouldn't allow them — the sort of thing that renders as a blank field in the UI and looks fine.
>
> The trade-off I'd name is schema coupling. A column rename breaks test code unrelated to whatever changed, and that's real maintenance cost. So my order of preference is API validation where an endpoint exposes what I need — it's a contract designed to be consumed and it's more stable — then direct queries for integrity and backend processing the API doesn't cover.
>
> Same reasoning applied to test data. Seeding straight into the database is fastest, but it bypasses application logic, so you can create states the application would never produce and then test against something impossible. API seeding by default; direct inserts only where nothing else exists."

---

## Common Interview Questions & Answers

### Q1: "Why do database validation if the UI shows success?"

**A:** "Because the UI showing success doesn't prove the write is correct. A row can land with a stale status, a rounded total, or a partial commit, and the confirmation message still appears. That's a real class of bug the UI structurally can't reveal — and it's usually the expensive kind, because it's silently wrong data."

### Q2: "How do you connect to a database from a test framework?"

**A:** "JDBC with try-with-resources so connections and result sets close even on failure — leaked connections exhaust the pool partway through a long suite and it looks like an infrastructure problem. `PreparedStatement` rather than concatenation, mainly because test data containing an apostrophe breaks a concatenated query. Credentials from config. For a real framework, a pool like HikariCP rather than a connection per query."

### Q3: "What's the downside of database assertions?"

**A:** "Schema coupling. A column rename breaks tests that have nothing to do with the feature that changed. So I'd validate through an API where one exposes what I need, since that's a contract designed for consumption and more stable, and reserve direct queries for integrity checks and backend processing the API doesn't cover."

### Q4: "Would you seed test data directly into the database?"

**A:** "Only where no API exists. It's fastest, but it bypasses application logic — validation, derived fields, triggers — so you can create states the application would never produce and then test against something impossible. API seeding is the default; direct inserts are a considered exception."

### Q5: "How do you clean up test data?"

**A:** "Tag rows with a test run identifier so deletion is precise, in teardown with `alwaysRun` so it happens on failure too. Cleanup never fails the test — a teardown error turning a passing test red is something the developer can't act on. And a scheduled sweep for anything crashed runs leave behind."

---

## Weak Areas to Watch

❌ **Don't concatenate SQL**
→ Injection risk, and apostrophes in test data break it.

❌ **Don't leak connections**
→ Pool exhaustion looks like infrastructure failure.

❌ **Don't query the DB as a shortcut around the API**
→ Unnecessary schema coupling.

❌ **Don't seed via DB when an API exists**
→ Bypasses logic; creates impossible states.

❌ **Don't let cleanup fail the test**
→ Undiagnosable red on a passing test.

---

## Key Takeaway for Interview

Database testing is a differentiator — plenty of UI engineers have never done it. Lead with it, and show the judgement.

The justification:

> **"The UI showing 'order confirmed' doesn't prove the row is correct. A write can land with a stale status or a rounded total and the confirmation still appears — that's silently wrong data, which is the expensive kind."**

And the trade-off that shows seniority:

> **"Database assertions couple you to the schema, so a column rename breaks unrelated tests. I'd validate through an API where one exists — that's a contract designed to be consumed — and reserve direct queries for integrity and backend processing nothing else exposes."**

---

## Checklist

✅ JDBC with try-with-resources
✅ `PreparedStatement`, never concatenation
✅ Credentials from config
✅ Connection pooling for real suites
✅ API validation preferred over direct queries
✅ Integrity checks: orphans, nulls, duplicates
✅ Test rows tagged with a run identifier
✅ Cleanup best-effort, never fails the test

---

**Remember:** A green UI test over silently wrong data is the most expensive kind of pass. 🎯
