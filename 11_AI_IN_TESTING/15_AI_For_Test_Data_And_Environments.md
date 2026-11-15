# AI For Test Data And Environments

## 1. What it is

Test data and environments are where flaky, slow, or blind tests usually start. AI can help *create*
and *review* them — generating realistic data shapes, edge-case values, API payload variants, and SQL
to inspect state — but it must never be trusted to know what the environment actually contains.

## 2. Why it matters in interviews

Data isolation is a core SDET topic (see 04_TEST_ENGINEERING). The AI angle is a follow-up: "can
AI generate test data?" Yes, with limits — it helps with shape and edge cases; real data
rules, referential integrity and privacy are mine.

## 3. 30-second answer

"I use AI to draft data shapes, boundary values, and API payload variants, and to suggest SQL for
verifying state. I don't use it to invent business rules or to touch real or sensitive data. Anything
it generates gets validated against the schema and constraints, and I keep test data isolated per
test — generated data helps variety, not isolation."

## 4. 2-minute answer

Where AI helps:

- **Data shapes and edge cases.** Given a field list with types and rules, it quickly proposes
  boundary values, null/empty/long strings, special characters, unicode, date edges, and role
  combinations — a good starting set I then trim.
- **Payload variants.** From an API contract, it can draft valid and invalid request bodies for
  RestAssured or Playwright request tests. I check them against the real schema.
- **Synthetic data.** Fake names, addresses, IDs. This avoids copying production data into tests.
- **SQL to inspect state.** Drafting verification queries (joins, duplicate checks, counts). I read
  every query before running it — especially anything that writes or deletes.
- **Environment checks.** Explaining config differences, reading a failing deployment log, or
  drafting a smoke checklist for an environment.

Where it fails: it doesn't know my actual schema constraints, foreign keys, uniqueness rules, or
which records already exist. Generated data that violates a hidden constraint causes confusing failures.
It is also random: tests must never depend on "whatever the AI generated today" — use fixed seeds or
stored fixtures for repeatability.

Rules I keep: (1) isolation first — each test creates and cleans its own data (API-assisted setup);
(2) generated data is committed or seeded so a failure is reproducible; (3) never feed production or
customer data to an external AI tool; (4) read-only SQL unless I've reviewed the statement.

## 5. Real project example

In BASES_PLAYWRIGHT the framework uses API-assisted setup and cleanup. AI helps me draft the
variants of the setup payload (boundary and negative values) and the verification query; the
fixtures, cleanup and isolation stay as framework design. In CIP_LEGACY, SQL checks on Oracle
and MySQL validated high-volume datasets; AI could draft the query shape, but I would verify
against the schema and run read-only first.

## 6. Common interview Q&A

**Q: Can AI generate test data?** Yes for shapes and edge cases and synthetic values; I validate it
against schema and rules, and keep per-test isolation.

**Q: Is AI-generated data reproducible?** Not by default — I store or seed it so failures can be
replayed.

**Q: Would you paste production data into an AI tool?** No — privacy and company policy; use
synthetic or masked data.

**Q: How do you check AI-written SQL?** Read it, run read-only on a safe environment first, compare
against known counts.

## 7. Weak areas to watch

- Generated data that breaks hidden constraints, causing false failures.
- Non-reproducible randomness.
- Letting AI-written SQL write or delete without review.
- Using real customer data. Never.

## 8. Key takeaway

AI is good at the *variety* of test data. Isolation, reproducibility and privacy are still my
design decisions.
