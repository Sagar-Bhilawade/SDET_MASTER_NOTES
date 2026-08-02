# Playwright - Test Data Management: Isolation, Factories & Cleanup

## What is Test Data Management?

How your tests get the data they need — and how they avoid stepping on each other while doing it.

Every test needs some precondition: a user that exists, a project with three items in it, an account with a specific balance. **Where that data comes from, and who owns it, determines whether your suite scales or collapses.**

This is the #1 cause of flakiness at scale, and it's the topic most candidates have no real answer for.

---

## Why It Matters (Interview Context)

Interviewers ask this because it separates people who've run a suite at scale from people who've written tests.

At 20 tests, you can hardcode a test user and get away with it.
At 300 tests running in parallel, hardcoded shared data **guarantees** collisions.

They're testing whether you understand:
1. **Isolation** — can two tests run simultaneously without interfering?
2. **Determinism** — does the test produce the same result every run?
3. **Cleanup** — does your suite leave the environment usable?
4. **Cost** — how much setup time are you paying per test?

---

## Your 30-Second Interview Answer

> "My rule is that every test owns its data. Tests generate what they need with a unique identifier, usually through a fixture that creates it via API and cleans up afterwards. I avoid shared mutable data entirely — that's the fastest way to get parallel tests colliding. Static shared data is fine only when it's genuinely read-only, like reference lookups."

---

## Your 2-Minute Deep Answer

> "There are four strategies, and the skill is knowing which one to use where.
>
> **1. SHARED STATIC DATA (Use sparingly)**
> A fixed set of records that exist in the environment. Reference data, lookup tables, a catalogue.
>
> ✅ Fine when: genuinely read-only, never mutated by any test.
> ❌ Dangerous when: any test writes to it. One test's edit becomes another's failure.
>
> **2. PER-TEST GENERATED DATA (My default)**
> Each test creates exactly what it needs, with a unique identifier.
>
> ```typescript
> const projectName = `proj-${testInfo.testId}-${Date.now()}`;
> ```
>
> ✅ Complete isolation, safe in parallel, deterministic.
> ❌ Costs setup time per test — mitigate by creating via API, not UI.
>
> **3. FACTORY PATTERN (How I structure #2)**
> A helper that builds valid objects with sensible defaults and lets the test override only what it cares about.
>
> ```typescript
> // utils/factories.ts
> export function buildUser(overrides: Partial<User> = {}): User {
>   return {
>     email: `user-${crypto.randomUUID()}@test.example.com`,
>     name: 'Test User',
>     role: 'standard',
>     ...overrides,
>   };
> }
>
> // In a test — only state what matters to THIS test
> const admin = buildUser({ role: 'admin' });
> ```
>
> This is the pattern that makes tests readable. The test says `role: 'admin'` and nothing else, so the reader immediately knows role is the variable under test. Everything else is noise the factory absorbs.
>
> **4. API-SEEDED SETUP (The performance unlock)**
> Never build preconditions through the UI.
>
> ```typescript
> // ❌ 15 seconds of clicking to reach the state you want to test
> await createProjectThroughUI(page);
> await addItemsThroughUI(page, 5);
> await page.goto('/project/reports');   // ← the actual thing under test
>
> // ✅ Under a second
> const project = await api.createProject(buildProject({ itemCount: 5 }));
> await page.goto(`/project/${project.id}/reports`);
> ```
>
> Two wins here, and the second is the one people miss:
> - **Speed** — obvious.
> - **Failure attribution** — if project creation breaks, the reports test fails for a reason that has nothing to do with reports. API setup means a UI test failure points at the feature under test, not at some unrelated upstream flow.
>
> **Cleanup strategy:**
>
> I put cleanup in the fixture teardown, so it runs whether the test passes or fails:
>
> ```typescript
> export const testProject = base.extend({
>   project: async ({ api }, use) => {
>     const project = await api.createProject(buildProject());
>     await use(project);
>     await api.deleteProject(project.id);   // runs even if the test failed
>   },
> });
> ```
>
> **The nuance on cleanup:** it must never fail the test. If teardown throws because the record was already deleted, you've turned a passing test red for no reason. Cleanup is best-effort — swallow the error, log it, move on.
>
> I also pair this with a **scheduled sweep** — a job that clears test records older than 24 hours. Fixture cleanup handles the normal path; the sweep handles crashed runs, killed CI jobs, and everything else that skips teardown. You need both.
>
> **What I avoid:**
>
> - **Hardcoded credentials in the repo** — environment variables or a secret store, always. This is a security issue, not just a hygiene one.
> - **Tests that depend on data another test created** — order dependence, guaranteed pain when you shard.
> - **A shared 'test user' account** — the classic. Works until parallel execution, then fails mysteriously forever."

---

## Strategy Comparison

| Strategy | Isolation | Speed | Use When |
|----------|-----------|-------|----------|
| **Shared static** | ❌ None | ⚡ Fastest | Read-only reference data only |
| **Per-test generated** | ✅ Full | Medium | Default choice |
| **Factory + API seed** | ✅ Full | ⚡ Fast | Anything with non-trivial preconditions |
| **UI-built setup** | ✅ Full | 🐌 Slowest | Only when testing the creation flow itself |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "With 300+ tests running in parallel, shared test data isn't an option — two tests hitting the same record is a collision waiting for the scheduler to find it.
>
> So every test owns its data. We use factories to build objects with sensible defaults, seed them through the API rather than the UI, and clean up in fixture teardown. The API-seeding piece was the biggest single win — it cut setup time dramatically, and it meant that when a UI test failed, it failed for a reason connected to what it was actually testing.
>
> We back that with a scheduled cleanup job for anything that escapes teardown — killed pipelines, crashed runs. Fixture cleanup covers the happy path, the sweep covers reality."

**The follow-up they'll ask: "What was the hardest part?"**

> "Getting agreement on test-only API endpoints. Creating data through the UI was slow, but it was also *working*, so there wasn't obvious urgency to change it. What moved the conversation was showing the pipeline time — once you put minutes-per-run in front of people, a seeding endpoint stops looking like a QA convenience and starts looking like a velocity issue for the whole team."

That answer works because it shows you can move an organizational blocker, not just write a factory function. At 20 LPA, the technical pattern is assumed — the influence is what they're checking for.

---

## Common Interview Questions & Answers

### Q1: "How do you manage test data at scale?"

**A:** "Every test owns its data. Factories build objects with defaults, tests override only the field they're actually testing, and setup happens through the API rather than the UI. Cleanup lives in fixture teardown so it runs on failure too, backed by a scheduled sweep for runs that die before teardown. The rule I hold to is no mutable shared state — that's what breaks parallel execution."

### Q2: "Isn't creating data per test slow?"

**A:** "It is if you create it through the UI. Through the API it's typically under a second, and you get the more important benefit: failure attribution. If a test builds its preconditions by clicking through three screens, a failure in any of those screens fails a test that was supposed to be about something else. API seeding means the test fails for reasons related to what it's testing."

### Q3: "What if the API for creating data doesn't exist?"

**A:** "Then I'd ask for it — a test-only seeding endpoint is a reasonable thing to request, and dev teams usually agree once you show them the pipeline time it saves. If that's genuinely not possible, I'd seed directly into the database for setup, with the caveat that you're now coupled to the schema and have to maintain that. UI-driven setup is the last resort, reserved for the tests where creation *is* the thing under test."

### Q4: "How do you handle data that takes a long time to set up?"

**A:** "Worker-scoped fixtures. If the setup is expensive and the data is genuinely read-only, create it once per worker instead of per test. The moment any test needs to mutate it, it goes back to per-test. The scope should follow the mutability, not the cost."

### Q5: "What about cleanup when a test crashes?"

**A:** "Fixture teardown handles the normal failure path — it runs whether the test passed or failed. It doesn't help if the process is killed or the CI job times out, which is why I pair it with a scheduled job that clears test records older than a day. Also worth saying: cleanup should never fail a test. If teardown throws, you've turned a green test red for a reason the developer can't act on."

---

## Weak Areas to Watch

❌ **Don't say:** "We have a set of test users we all share"
→ This tells the interviewer you haven't run a parallel suite.

❌ **Don't build preconditions through the UI**
→ Slow, and it misattributes failures.

❌ **Don't let cleanup throw**
→ A failed teardown turning a passing test red is a self-inflicted wound.

❌ **Don't hardcode credentials**
→ Security issue. Environment variables or a secret store.

❌ **Don't rely on test execution order**
→ Breaks the moment you shard across machines.

---

## Key Takeaway for Interview

Test data is where suites quietly die. The discipline is simple and non-negotiable:

**Every test owns its data. Seed through the API. Clean up in teardown. Never share anything mutable.**

The line that signals seniority:

> **"Seeding through the API isn't just about speed — it's about failure attribution. A test should fail for reasons connected to what it's testing."**

---

## Test Data Checklist

✅ Unique identifier per test (UUID or testId + timestamp)
✅ Factory functions with sensible defaults
✅ Test overrides only the field under test
✅ Setup via API, never via UI
✅ Cleanup in fixture teardown (runs on failure too)
✅ Cleanup never throws
✅ Scheduled sweep for orphaned data
✅ Credentials from env vars, never hardcoded
✅ No mutable shared state, ever

---

**Remember:** If two tests can touch the same record, they eventually will. 🎯
