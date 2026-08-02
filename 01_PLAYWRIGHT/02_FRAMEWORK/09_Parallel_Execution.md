# Playwright - Parallel Execution: Sharding & Workers for Speed

## What is Parallel Execution?

Running multiple tests simultaneously instead of one at a time. 

Basic idea:
```
Sequential: Test 1 (10s) + Test 2 (10s) + Test 3 (10s) = 30s
Parallel: Test 1, 2, 3 run together = 10s
```

Playwright runs 300 tests in 8-10 minutes (not 30+ minutes).

---

## Why It Matters (Interview Context)

Interviewers ask about parallel execution because:
1. **Shows CI/CD optimization thinking** (tests must be fast)
2. **Demonstrates scalability** (how you handle growth)
3. **Proves infrastructure understanding** (workers, sharding, resources)

Fast tests = fast feedback = faster shipping.

---

## Your 30-Second Interview Answer

> "Playwright parallelizes by default - multiple tests run simultaneously. At NielsenIQ, we run 300+ tests on Azure DevOps VMs using workers. Tests run in parallel within one worker for isolation. We also shard across multiple machines in CI. Result: 300 tests in 8-10 minutes instead of 30+."

---

## Your 2-Minute Deep Answer

> "Parallel execution is how you keep test suites fast as they grow.
>
> **How Playwright Parallelization Works:**
>
> ```
> Test Suite (300 tests)
>     ↓
> Worker 1: Tests 1-50 (parallel)
> Worker 2: Tests 51-100 (parallel)
> Worker 3: Tests 101-150 (parallel)
> Worker 4: Tests 151-300 (parallel)
>     ↓
> All workers complete (each in ~10 min, total ~10 min)
> ```
>
> **vs Sequential (Old Way):**
> ```
> Test 1 (10s) → Test 2 (10s) → ... → Test 300 (10s)
> Total: 300 × 10s = 50 minutes!
> ```
>
> **Configuration:**
>
> ```typescript
> // playwright.config.ts
> export default defineConfig({
>   // Local: Use 4 workers (fast feedback)
>   workers: process.env.CI ? 1 : 4,
>
>   // CI: Can use more workers if resources available
>   // workers: process.env.CI ? 4 : 8,
> });
> ```
>
> **Test Isolation (Critical for Parallel Tests):**
>
> Parallel tests must not interfere with each other:
>
> ```typescript
> // ❌ WRONG: Shared database (Tests interfere!)
> let database;
>
> beforeAll(async () => {
>   database = await connectDatabase();
> });
>
> test('Create user', async () => {
>   await database.createUser('user@example.com');
> });
>
> test('Delete user', async () => {
>   // If parallel, might delete user from Test 1!
>   await database.deleteUser('user@example.com');
> });
> ```
>
> ```typescript
> // ✅ RIGHT: Isolated per test
> test('Create user', async ({ database }) => {
>   const user = await database.createUser('user1@example.com');
>   // Test uses unique data
> });
>
> test('Delete user', async ({ database }) => {
>   const user = await database.createUser('user2@example.com');
>   // Different data, no interference
> });
> ```
>
> **Playwright's Test Isolation:**
>
> Playwright handles isolation automatically:
> - Each test gets its own browser context
> - Contexts are completely isolated (no shared cookies, localStorage)
> - Tests in parallel don't see each other's state
>
> You just need to make sure your test data is unique per test.
>
> **Sharding (Across Multiple Machines):**
>
> For very large test suites, shard across multiple CI jobs:
>
> ```yaml
> # .github/workflows/tests.yml
> jobs:
>   test:
>     strategy:
>       matrix:
>         shard: [1, 2, 3, 4]
>     steps:
>       - name: Run tests
>         run: npx playwright test --shard=${{ matrix.shard }}/4
> ```
>
> This runs:
> - Shard 1: Tests 1-75
> - Shard 2: Tests 76-150
> - Shard 3: Tests 151-225
> - Shard 4: Tests 226-300
>
> All 4 shards run in parallel on different VMs.
> Total time: Same as 75 tests (much faster!)
>
> **At NielsenIQ:**
>
> **Local Execution:**
> ```typescript
> export default defineConfig({
>   workers: 4,  // 4 parallel workers
> });
> ```
> 300 tests in ~10 minutes locally
>
> **CI Execution:**
> ```yaml
> # Azure DevOps Pipeline
> jobs:
>   - job: TestChromium
>     timeoutInMinutes: 30
>     steps:
>       - script: npx playwright test --project=chromium
>
>   - job: TestFirefox
>     timeoutInMinutes: 30
>     steps:
>       - script: npx playwright test --project=firefox
> ```
> Chromium and Firefox test suites run in parallel (different jobs)
> Each suite parallelizes internally (4 workers)
> Total: ~10 minutes (not 20+)
>
> **Parallel Execution Best Practices:**
>
> **1. USE FIXTURES FOR TEST DATA**
> ```typescript
> // Each test gets unique data
> export const uniqueUser = base.extend({
>   user: async ({}, use) => {
>     const email = `user-${Date.now()}@example.com`;
>     const user = await createUser(email);
>     await use(user);
>   },
> });
> ```
>
> **2. CLEAR STATE BETWEEN TESTS**
> ```typescript
> afterEach(async ({ page }) => {
>   // Clear cookies, localStorage
>   await page.context().clearCookies();
>   await page.evaluate(() => localStorage.clear());
> });
> ```
>
> **3. DON'T SHARE RESOURCES**
> ```typescript
> // ❌ Wrong
> const sharedDatabase = await connectDatabase();
>
> // ✅ Right
> database: async ({}, use) => {
>   const db = await connectDatabase();  // Fresh per test
>   await use(db);
> }
> ```
>
> **4. USE UNIQUE IDENTIFIERS**
> ```typescript
> // Unique per test
> const projectId = `proj-${Date.now()}-${Math.random()}`;
> ```
>
> **Result:**
> - ✅ 300+ tests in 8-10 minutes (not 30+)
> - ✅ Tests run in parallel without interference
> - ✅ CI jobs finish fast, feedback is quick
> - ✅ Developers can iterate faster"

---

## Parallel Execution Speed Comparison

| Setup | Tests | Time | Speed vs Sequential |
|-------|-------|------|------------------|
| Sequential | 300 | 50 min | 1x |
| 4 Workers Local | 300 | 12 min | 4x |
| 4 Workers + 4 Shards CI | 300 | 3 min | 16x |

---

## Your Real Project Example (NielsenIQ)

**Local Configuration:**
```typescript
export default defineConfig({
  workers: 4,  // 4 parallel workers
  use: {
    navigationTimeout: 30000,
    actionTimeout: 10000,
  },
});
```

**CI Configuration (Azure DevOps):**
```yaml
jobs:
  - job: Test_Chromium
    timeoutInMinutes: 30
    pool:
      vmImage: 'ubuntu-latest'
    steps:
      - task: NodeTool@0
        inputs:
          versionSpec: '18.x'
      - script: npm ci
      - script: npx playwright test --project=chromium
        env:
          CI: true

  - job: Test_Firefox
    timeoutInMinutes: 30
    dependsOn: []  # Run in parallel
    pool:
      vmImage: 'ubuntu-latest'
    steps:
      - task: NodeTool@0
      - script: npm ci
      - script: npx playwright test --project=firefox
```

**Test Implementation (Isolated Data):**
```typescript
test('Create project', async ({ page, database }) => {
  const projectName = `project-${Date.now()}`;  // Unique per test
  
  await page.goto('/dashboard');
  await page.getByRole('button', { name: 'Create' }).click();
  await page.getByTestId('project-name').fill(projectName);
  await page.getByRole('button', { name: 'Save' }).click();
  
  const project = await database.getProjectByName(projectName);
  expect(project).toBeDefined();
});
```

**Result:**
- Chromium suite: 300 tests in 10 min
- Firefox suite: 300 tests in 10 min
- Both run in parallel on different VMs
- Total CI time: ~10 minutes (not 20+)

---

## Common Interview Questions & Answers

### Q1: "How many workers should you use?"

**A:** "Local: CPU count (usually 4-8). CI: Depends on resources. We use 1-4 workers on CI to avoid resource contention. Sharding across multiple CI jobs is more scalable than many workers on one VM."

### Q2: "What if parallel tests interfere with each other?"

**A:** "They're not truly isolated. Use unique test data (email: user-123@example.com), clear state between tests (cookies, localStorage), use fixtures for per-test setup. Playwright's context isolation helps but you must ensure data isolation."

### Q3: "Can you run different browsers in parallel?"

**A:** "Yes! Define separate projects in config (chromium, firefox). CI runs each project in parallel. Each project internally parallelizes with workers."

### Q4: "What if a CI machine is slow?"

**A:** "Tests take longer but still complete. No tests are skipped. Total time increases but parallelism still helps - slow machine with 4 workers is better than 1."

### Q5: "How do you report results from parallel execution?"

**A:** "Playwright merges results automatically. You run `npx playwright test` and get one HTML report with all tests (from all workers, all projects). No manual merging needed."

---

## What You Should Be Ready to Explain

✅ How parallel execution reduces test time  
✅ Worker configuration (local vs CI)  
✅ Test isolation strategies (unique data, clear state)  
✅ Sharding across multiple CI jobs  
✅ Browser parallelization (multiple projects)  
✅ Your 300+ test execution at NielsenIQ  

---

## Weak Areas to Watch

❌ **Don't share:** Database connections across tests  
→ **Reality:** Each test needs fresh/isolated data

❌ **Don't hardcode:** Test identifiers  
→ **Reality:** Use timestamps or UUIDs for uniqueness

❌ **Don't ignore:** Browser context isolation  
→ **Reality:** Rely on it but verify data isolation

❌ **Don't over-parallelize:** 100 workers on single machine  
→ **Reality:** Resource contention actually slows things down

---

## Key Takeaway for Interview

Parallel execution is how test suites scale without becoming slow. Your understanding of isolation, sharding, and worker management shows if you can handle growth.

At 300+ tests, parallelization is non-negotiable for fast feedback.

**At NielsenIQ:** Your 300 tests run in 8-10 minutes, not 30+. That's engineering.

**Your competitive edge:** You think about test infrastructure for scale, not just test correctness. ✨

---

## Parallel Execution Checklist

✅ Workers configured for environment (4 local, 1-4 CI)  
✅ Test data isolation (unique identifiers per test)  
✅ State cleared between tests (cookies, storage)  
✅ No shared resources across tests  
✅ Fixtures for per-test setup  
✅ Multiple projects run in parallel (if needed)  
✅ CI sharding configured (if 1000+ tests)  
✅ Results merged and reported  

---

**Remember:** Parallel tests = fast feedback = happy developers. 🚀
