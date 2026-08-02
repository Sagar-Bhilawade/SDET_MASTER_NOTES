# Playwright - Hooks: beforeEach, afterEach, beforeAll, afterAll

## What Are Hooks?

Hooks are functions that run at specific points in test execution:
- **beforeAll** - Runs once before ALL tests in a file
- **afterAll** - Runs once after ALL tests in a file  
- **beforeEach** - Runs before EACH test
- **afterEach** - Runs after EACH test

Hooks are useful for setup/teardown that applies to all tests in a file.

---

## Why It Matters (Interview Context)

Interviewers ask about hooks because:
1. **Shows you understand test lifecycle**
2. **Demonstrates cleanup discipline** (tests don't leave garbage)
3. **Proves you avoid test interdependence** (each test is independent)

Important: **Fixtures are preferred over hooks for most cases.** But hooks have their place.

---

## Your 30-Second Interview Answer

> "Hooks (beforeEach, afterEach) run before/after tests. I use fixtures for most setup, but hooks handle global setup (like starting a test server in beforeAll). At NielsenIQ, most setup is in fixtures for reusability, hooks handle only suite-level concerns."

---

## Your 2-Minute Deep Answer

> "Hooks and fixtures both manage setup/teardown, but they're different:
>
> **FIXTURES (Preferred):**
> - Reusable across test files
> - Can be parameterized
> - Lazy evaluation
> - Clear dependencies
> - Scoped (test, worker, project)
>
> **HOOKS (Use when fixtures don't fit):**
> - Run in a specific file
> - Not reusable
> - Always run (no lazy evaluation)
> - Good for suite-level setup
>
> **When to Use Hooks:**
>
> **1. beforeAll/afterAll (Project/Worker Setup)**
> ```typescript
> beforeAll(async () => {
>   // Start test server ONCE before all tests
>   await startTestServer();
> });
>
> afterAll(async () => {
>   // Stop test server ONCE after all tests
>   await stopTestServer();
> });
>
> // All tests in this file use the same server
> test('Test 1', async ({ page }) => { ... });
> test('Test 2', async ({ page }) => { ... });
> ```
>
> Why not fixture? This is global setup, not per-test setup.
>
> **2. beforeEach/afterEach (Test-Level Cleanup)**
>
> Generally, use fixtures instead. But hooks are useful if:
> - You need the raw Playwright API (not a custom test type)
> - You're adding to existing hook-based code
>
> ```typescript
> beforeEach(async ({ page }) => {
>   // Clear browser cache before each test
>   await page.context().clearCookies();
> });
>
> afterEach(async ({ page }) => {
>   // Take screenshot on failure
>   if (testInfo.status !== 'passed') {
>     await page.screenshot({ path: `failure-${testInfo.title}.png` });
>   }
> });
> ```
>
> **At NielsenIQ:**
>
> **Our Strategy:**
> - ✅ Use fixtures for reusable setup (auth, database, API)
> - ✅ Use hooks for suite-level setup (starting server, global config)
> - ✅ Avoid beforeEach if fixture can do it
> - ✅ Always cleanup in afterEach (or use fixtures for auto-cleanup)
>
> **Example:**
> ```typescript
> // ❌ Don't do this (use fixtures instead)
> beforeEach(async ({ page }) => {
>   await page.goto('/login');
>   await page.getByTestId('email').fill('user@example.com');
>   await page.getByTestId('password').fill('password123');
>   await page.getByRole('button', { name: 'Login' }).click();
> });
>
> // ✅ Do this (fixture)
> export const authenticatedUser = base.extend({
>   page: async ({ page }, use) => {
>     // Login setup
>     await use(page);
>   },
> });
>
> // ✅ Hooks are okay for this
> beforeAll(async () => {
>   // Start test server
>   await startServer();
> });
>
> afterAll(async () => {
>   // Stop test server
>   await stopServer();
> });
> ```
>
> **Hook Best Practices:**
>
> **1. ALWAYS CLEANUP**
> ```typescript
> afterEach(async ({ page }) => {
>   // Cleanup cookies, localStorage
>   await page.context().clearCookies();
>   await page.evaluate(() => localStorage.clear());
> });
> ```
>
> **2. HANDLE ERRORS IN HOOKS**
> ```typescript
> beforeEach(async ({ page }) => {
>   try {
>     await page.goto('http://localhost:3000');
>   } catch (error) {
>     console.error('Setup failed:', error);
>     throw error;
>   }
> });
> ```
>
> **3. SKIP CLEANUP IF NOT NEEDED**
> ```typescript
> // Bad: Too much cleanup
> afterEach(async ({ page }) => {
>   await page.close();  // Page is auto-closed anyway
>   await page.context().close();  // Context is auto-closed anyway
> });
>
> // Good: Only cleanup what you added
> afterEach(async ({ page }) => {
>   // Only cleanup resources you explicitly created
>   // Page/context cleanup is automatic
> });
> ```"

---

## Hook Lifecycle

```
Test File Starts
    ↓
beforeAll()  ← Runs once before all tests
    ↓
Test 1
    ├── beforeEach()  ← Runs before this test
    ├── Test logic
    └── afterEach()   ← Runs after this test
    ↓
Test 2
    ├── beforeEach()  ← Runs before this test
    ├── Test logic
    └── afterEach()   ← Runs after this test
    ↓
afterAll()  ← Runs once after all tests
    ↓
Test File Ends
```

---

## Hooks vs Fixtures Comparison

| Aspect | Hooks | Fixtures |
|--------|-------|----------|
| **Reusability** | File-level only | Cross-file reusable |
| **Scoping** | beforeEach/afterEach | test/worker/project |
| **Lazy** | No (always run) | Yes (only if used) |
| **Parameters** | No | Yes (parameterizable) |
| **Dependencies** | Manual | Automatic |
| **Best for** | Suite setup | Per-test setup |

---

## Your Real Project Example (NielsenIQ)

**Good: Using hooks for suite-level setup**
```typescript
// tests/integration/api.spec.ts
let apiServer: Server;

beforeAll(async () => {
  // Start test server once
  apiServer = await startTestServer();
});

afterAll(async () => {
  // Stop test server once
  await apiServer.close();
});

test('API returns 200', async ({ request }) => {
  const response = await request.get('/api/health');
  expect(response.status()).toBe(200);
});

test('API handles errors', async ({ request }) => {
  const response = await request.post('/api/invalid', { data: {} });
  expect(response.status()).toBe(400);
});
```

**Not Good: Using hooks for per-test setup**
```typescript
// ❌ Don't do this (use fixtures instead)
beforeEach(async ({ page }) => {
  const login = new LoginPage(page);
  await login.login('user@example.com', 'password123');
});

test('User creates project', async ({ page }) => {
  // Already logged in
});

test('User edits project', async ({ page }) => {
  // Already logged in
});

// Problem: Setup is duplicated in 50+ test files
// Use a fixture instead!
```

---

## Common Interview Questions & Answers

### Q1: "When should you use hooks vs fixtures?"

**A:** "Fixtures for reusable per-test setup (auth, database). Hooks for suite-level setup that runs once (starting server). Most setup should be fixtures because they're reusable and lazy."

### Q2: "What if a hook fails?"

**A:** "The entire test file fails. If beforeAll fails, all tests in that file are marked as failed. That's why it's important to handle errors in hooks properly."

### Q3: "Should you put test-specific setup in beforeEach?"

**A:** "No. Use fixtures. beforeEach runs for EVERY test even if some don't need it. Fixtures are lazy - they only run when a test uses them."

### Q4: "How do you handle cleanup if a test fails?"

**A:** "afterEach runs even if the test fails. That's its job - cleanup resources regardless of outcome. Use testInfo.status to check if test passed/failed if you need different cleanup logic."

### Q5: "Can hooks call fixtures?"

**A:** "Hooks and fixtures are different systems. Don't mix them. Use fixtures for setup that tests need, hooks for setup that doesn't depend on test context."

---

## What You Should Be Ready to Explain

✅ Difference between beforeEach and beforeAll  
✅ When to use hooks vs fixtures  
✅ How to handle cleanup in afterEach  
✅ Hook execution order  
✅ Error handling in hooks  
✅ Why fixtures are preferred  

---

## Weak Areas to Watch

❌ **Don't put:** Per-test setup in hooks  
→ **Reality:** Use fixtures for reusability

❌ **Don't forget:** Cleanup in afterEach  
→ **Reality:** Every test cleanup must happen

❌ **Don't ignore:** Hook failures  
→ **Reality:** Failing beforeAll fails all tests

❌ **Don't mix:** Hooks and fixtures  
→ **Reality:** Keep them separate

---

## Key Takeaway for Interview

Hooks are for suite-level setup. Fixtures are for per-test setup. Use the right tool for the job.

If you're putting the same setup in beforeEach in 50+ files, you should be using fixtures.

**At NielsenIQ:** Most setup is in fixtures. Hooks handle only the global concerns.

**Your competitive edge:** You know when to use hooks vs fixtures, not just that they exist. ✨

---

## Hook Checklist

✅ beforeAll/afterAll for suite-level setup (not per-test)  
✅ beforeEach/afterEach only when fixtures can't  
✅ Always cleanup in afterEach  
✅ Handle errors in hooks  
✅ No test-specific logic in hooks  
✅ Prefer fixtures for reusability  

---

**Remember:** Hooks run for their scope. Use them wisely. 🎯
