# Playwright - Fixtures: Reusable Setup & Teardown

## What Are Fixtures?

Fixtures are functions that run before tests to set up conditions, and after tests to tear down. They're like beforeEach/afterEach hooks, but better:
- Reusable across multiple tests
- Scoped (test-level, worker-level, project-level)
- Automatic cleanup
- Parameterizable
- Lazy (only run when used)

---

## Why It Matters (Interview Context)

Interviewers ask about fixtures because:
1. **Shows test design thinking** (not just hooks scattered everywhere)
2. **Demonstrates reusability** (write setup once, use 50 times)
3. **Proves DRY principle** (don't repeat setup logic)
4. **Explains scalability** (how you handle 300+ tests with clean setup)

Fixtures are the Playwright way. Hooks are the old way.

---

## Your 30-Second Interview Answer

> "Fixtures are reusable setup functions. Instead of beforeEach hooks in every test, I define a fixture once (like authenticatedUser) and inject it where needed. Playwright handles setup/teardown automatically. At NielsenIQ, 50+ tests use the same login fixture - updated once, all tests adapt. More maintainable than scattered hooks."

---

## Your 2-Minute Deep Answer

> "Fixtures replace beforeEach/afterEach hooks with a cleaner, reusable model.
>
> **The Problem with Hooks:**
>
> ```typescript
> // ❌ Hooks scattered, duplicated
> describe('Dashboard', () => {
>   let page: Page;
>
>   beforeEach(async () => {
>     // Login setup (duplicated in EVERY test file)
>     page = await browser.newPage();
>     await page.goto('/login');
>     await page.getByTestId('email').fill('user@example.com');
>     await page.getByTestId('password').fill('password123');
>     await page.getByRole('button', { name: 'Login' }).click();
>     await page.waitForURL('/dashboard');
>   });
>
>   afterEach(async () => {
>     // Cleanup (duplicated)
>     await page.close();
>   });
>
>   test('User sees widgets', async () => {
>     // Test logic
>   });
> });
>
> // Problem: Same login setup in 50+ test files!
> // Change login flow → Update 50+ files
> ```
>
> **The Solution: Fixtures**
>
> ```typescript
> // fixtures/auth.fixture.ts
> export const authenticatedUser = base.extend({
>   page: async ({ page }, use) => {
>     // Setup
>     await page.goto('/login');
>     await page.getByTestId('email').fill('user@example.com');
>     await page.getByTestId('password').fill('password123');
>     await page.getByRole('button', { name: 'Login' }).click();
>     await page.waitForURL('/dashboard');
>
>     // Provide to test
>     await use(page);
>
>     // Teardown (automatic)
>     // page.close() automatically called
>   },
> });
>
> // tests/dashboard/dashboard.spec.ts
> import { authenticatedUser } from '../fixtures/auth.fixture';
> const test = base.extend(authenticatedUser);
>
> test('User sees widgets', async ({ page }) => {
>   // Already logged in!
>   // Page automatically cleaned up after test
> });
> ```
>
> **Fixture Benefits:**
>
> **1. SETUP IN ONE PLACE**
> ```typescript
> // Write once
> export const authenticatedUser = base.extend({
>   page: async ({ page }, use) => {
>     await login(page);
>     await use(page);
>   },
> });
>
> // Use in 50+ tests
> test('Create project', async ({ page }) => { ... });
> test('Delete project', async ({ page }) => { ... });
> test('Edit project', async ({ page }) => { ... });
> ```
>
> **2. LAZY EVALUATION**
> ```typescript
> // Fixtures only run if test uses them
> export const apiClient = base.extend({
>   client: async ({}, use) => {
>     const client = new APIClient();
>     await use(client);
>     await client.cleanup();
>   },
> });
>
> // test1 uses apiClient → Fixture runs
> test('Create user via API', async ({ apiClient }) => { ... });
>
> // test2 doesn't use apiClient → Fixture doesn't run (saves time)
> test('Navigate home', async ({ page }) => { ... });
> ```
>
> **3. FIXTURE SCOPES**
>
> ```typescript
> // Test scope (runs before each test)
> export const freshUser = base.extend({
>   user: async ({ request }, use) => {
>     const response = await request.post('/api/users', {
>       data: { email: 'user@example.com' }
>     });
>     const user = await response.json();
>     await use(user);
>     // Cleanup after test
>   },
>   scope: 'test'  // Default
> });
>
> // Worker scope (runs once per test worker)
> export const database = base.extend({
>   db: async ({}, use) => {
>     const db = await connectDatabase();
>     await use(db);
>     // Cleanup after all tests in worker
>   },
>   scope: 'worker'
> });
>
> // Project scope (runs once for entire run)
> export const server = base.extend({
>   server: async ({}, use) => {
>     const server = await startTestServer();
>     await use(server);
>     // Cleanup after all tests
>   },
>   scope: 'project'
> });
> ```
>
> **4. PARAMETERIZED FIXTURES**
>
> ```typescript
> // Run same test with different users
> export const userFixture = base.extend({
>   user: async ({ }, use, testInfo) => {
>     const users = {
>       admin: { email: 'admin@example.com', role: 'admin' },
>       user: { email: 'user@example.com', role: 'user' },
>     };
>
>     await use(users.admin);  // First run
>     await use(users.user);   // Second run
>   },
> });
>
> test('User can access dashboard', async ({ user, page }) => {
>   await login(page, user.email);
>   await expect(page.getByText(`Welcome, ${user.role}`)).toBeVisible();
> });
>
> // Test runs twice: once with admin, once with user
> ```
>
> **At NielsenIQ:**
> ```typescript
> // fixtures/auth.fixture.ts
> export const authenticatedUser = base.extend({
>   loginPage: async ({ page }, use) => {
>     const login = new LoginPage(page);
>     await login.login('sdet@example.com', 'password123');
>     await use(login);
>   },
> });
>
> // fixtures/database.fixture.ts
> export const withDatabaseReset = base.extend({
>   db: async ({}, use) => {
>     const db = new DatabaseClient();
>     await db.reset();  // Clear test data
>     await use(db);
>     await db.cleanup();
>   },
>   scope: 'test'
> });
>
> // tests/projects/create-project.spec.ts
> import { authenticatedUser } from '../fixtures/auth.fixture';
> const test = base.extend(authenticatedUser);
>
> test('User creates project', async ({ loginPage, page }) => {
>   await loginPage.clickNewProject();
>   await page.getByTestId('project-name').fill('My Project');
>   await page.getByRole('button', { name: 'Create' }).click();
>   await expect(page).toHaveURL(/\\/projects\\/.*/);
> });
> ```
>
> **Result:**
> - ✅ 50+ tests use authenticatedUser fixture
> - ✅ Change login flow → Update fixture once → All tests adapt
> - ✅ New test: just add fixture parameter
> - ✅ Automatic cleanup (no manual page.close())
> - ✅ Clear test purpose (fixture handles setup, test handles feature)"

---

## Fixture Types & Scopes

| Type | Scope | Runs | When to Use |
|------|-------|------|------------|
| **Test Fixture** | test | Before each test | Setup unique to one test (fresh user) |
| **Worker Fixture** | worker | Once per worker | Expensive setup shared by tests (database) |
| **Project Fixture** | project | Once for all | Very expensive setup (test server) |

---

## Your Real Project Example (NielsenIQ)

**fixtures/auth.fixture.ts:**
```typescript
import { test as base } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';

export const authenticatedUser = base.extend({
  loginPage: async ({ page }, use) => {
    const login = new LoginPage(page);
    await login.login('sdet@example.com', 'password123');
    await use(login);
    // Cleanup automatically
  },
});

export const adminUser = base.extend({
  loginPage: async ({ page }, use) => {
    const login = new LoginPage(page);
    await login.login('admin@example.com', 'password123');
    await use(login);
  },
});
```

**tests/auth/auth-tests.spec.ts:**
```typescript
import { authenticatedUser, adminUser } from '../fixtures/auth.fixture';
const test = authenticatedUser;  // Tests use this fixture

test('User sees own profile', async ({ loginPage, page }) => {
  // Already logged in as SDET user
  await page.goto('/profile');
  await expect(page.getByText('SDET User')).toBeVisible();
});

test('User edits preferences', async ({ loginPage, page }) => {
  // Already logged in
  await page.goto('/settings');
  await page.getByTestId('theme-switch').click();
});
```

**Result:**
- No beforeEach hooks
- Setup in one place
- 50+ tests use same fixture
- Clean, readable tests

---

## Common Interview Questions & Answers

### Q1: "Why fixtures instead of beforeEach hooks?"

**A:** "Reusability. With beforeEach, login logic is in every test file. With fixtures, it's in one place. Change login → Update fixture → All tests adapt. Also, fixtures are lazy - if a test doesn't use the fixture, it doesn't run."

### Q2: "What's the difference between test and worker scope?"

**A:** "Test scope runs before EACH test (fresh setup every time). Worker scope runs ONCE per worker. Use test scope for setup that must be fresh (new user), worker scope for expensive setup shared by tests (test database)."

### Q3: "How do you handle cleanup with fixtures?"

**A:** "Automatic. After `await use()`, cleanup code runs. For example: open database → `await use()` → close database. You don't manually call cleanup."

### Q4: "Can fixtures depend on other fixtures?"

**A:** "Yes. A fixture can require another fixture. For example: `adminUser` fixture might depend on `loginPage` fixture. Playwright handles the dependency chain automatically."

### Q5: "How do you test multiple users with one test?"

**A:** "Parameterized fixtures. Define the fixture to run multiple times with different data. Test runs once per parameter set."

---

## What You Should Be Ready to Explain

✅ Why fixtures are better than beforeEach hooks  
✅ How fixtures are reusable across tests  
✅ Fixture scopes (test, worker, project)  
✅ Automatic cleanup  
✅ Parameterized fixtures  
✅ Your fixture strategy at NielsenIQ  

---

## Weak Areas to Watch

❌ **Don't put:** Heavy logic in beforeEach  
→ **Reality:** Use fixtures for reusability

❌ **Don't ignore:** Fixture scope  
→ **Reality:** Correct scope improves performance

❌ **Don't forget:** Cleanup in fixtures  
→ **Reality:** Automatic cleanup prevents resource leaks

❌ **Don't create:** Monolithic fixtures  
→ **Reality:** Compose small, focused fixtures

---

## Key Takeaway for Interview

Fixtures are how Playwright makes setup reusable, scalable, and clean. They're more powerful than hooks because they're designed for test infrastructure.

Your fixture design determines if 300+ tests can share setup or if setup is scattered everywhere.

**At NielsenIQ:** Your fixtures reduce setup duplication by 90%. That's why tests are maintainable.

**Your competitive edge:** You think in fixtures, not hooks. That's modern test architecture. ✨

---

## Fixture Checklist

✅ Setup in fixtures, not beforeEach  
✅ One fixture per concern (auth, db, API)  
✅ Correct scope (test vs worker vs project)  
✅ Lazy evaluation (only run when used)  
✅ Automatic cleanup (code after await use())  
✅ Composed fixtures (dependent fixtures work)  
✅ Clear fixture naming  
✅ Reused by 10+ tests (validates single responsibility)  

---

**Remember:** Fixtures scale, hooks don't. Build with fixtures. 🎯
