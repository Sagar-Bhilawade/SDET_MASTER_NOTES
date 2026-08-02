# Playwright - Framework Design: Building Scalable Test Architecture

## What is Framework Design?

Framework design is how you structure your codebase to make tests scalable, maintainable, and reusable. It's not just "running tests" - it's building infrastructure.

A good framework:
- Eliminates code duplication
- Separates concerns (tests, pages, utilities)
- Makes debugging easy
- Scales from 100 to 10,000 tests without chaos
- Allows new SDETs to write tests in 30 minutes, not 3 days

---

## Why It Matters (Interview Context)

Interviewers ask about framework design because:
1. **Shows architectural thinking** (not just "I write test scripts")
2. **Demonstrates scalability** (can your framework handle growth?)
3. **Explains maintainability** (can others maintain your tests?)
4. **Shows leadership** (would they want you designing their framework?)

Bad framework = 300+ tests that are impossible to maintain.
Good framework = 300+ tests that anyone can maintain.

---

## Your 30-Second Interview Answer

> "At NielsenIQ, I designed a test framework separating concerns: tests describe WHAT to verify, page objects encapsulate HOW to interact, fixtures handle setup/teardown. This keeps our 300+ tests maintainable. Each test is 5-10 lines - describing behavior, not implementation. Folders organized by feature, not by test type. New SDETs can write tests in 30 minutes."

---

## Your 2-Minute Deep Answer

> "Building a Playwright framework is like building production code - structure matters.
>
> **Our Framework Architecture at NielsenIQ:**
>
> ```
> tests/
>   ├── auth/
>   │   ├── login-valid-credentials.spec.ts      (Test files)
>   │   ├── logout-clears-session.spec.ts
>   ├── dashboard/
>   │   ├── display-user-widgets.spec.ts
>   │   ├── edit-widget-preferences.spec.ts
>
> pages/
>   ├── LoginPage.ts                             (Page Objects)
>   ├── DashboardPage.ts
>   ├── BasePage.ts
>
> fixtures/
>   ├── auth.fixture.ts                          (Setup/Teardown)
>   ├── user.fixture.ts
>   ├── database.fixture.ts
>
> utils/
>   ├── dataUtils.ts                             (Helpers)
>   ├── apiHelpers.ts
>   ├── dbHelpers.ts
>
> config/
>   ├── playwright.config.ts
>   ├── constants.ts
>
> data/
>   ├── testData.json                            (Test Data)
>   ├── users.json
> ```
>
> **Why this structure?**
>
> **1. SEPARATION OF CONCERNS**
>
> Test files describe WHAT:
> ```typescript
> // tests/auth/login-valid-credentials.spec.ts
> test('User logs in with valid credentials', async ({ loginPage }) => {
>   await loginPage.navigateToLoginPage();
>   await loginPage.fillEmail('user@example.com');
>   await loginPage.fillPassword('password123');
>   await loginPage.clickLoginButton();
>   await expect(page).toHaveURL('/dashboard');
> });
> ```
>
> Page objects encapsulate HOW:
> ```typescript
> // pages/LoginPage.ts
> export class LoginPage {
>   constructor(private page: Page) {}
>
>   async navigateToLoginPage() {
>     await this.page.goto('/login');
>   }
>
>   async fillEmail(email: string) {
>     await this.page.getByTestId('email-input').fill(email);
>   }
>
>   async clickLoginButton() {
>     await this.page.getByRole('button', { name: 'Login' }).click();
>   }
> }
> ```
>
> Why? If the email field changes from data-testid='email-input' to data-testid='user-email', you update ONE place (LoginPage), not 50+ tests.
>
> **2. FIXTURE-BASED SETUP**
>
> Instead of beforeEach hooks in tests:
> ```typescript
> // ❌ Scattered logic in every test
> test('User edits profile', async ({ page }) => {
>   await page.goto('/login');
>   await page.getByTestId('email').fill('user@example.com');
>   await page.getByTestId('password').fill('pass');
>   await page.getByRole('button', { name: 'Login' }).click();
>   await page.waitForURL('/dashboard');
>   // ... NOW test the actual feature
> });
> ```
>
> Use fixtures:
> ```typescript
> // fixtures/auth.fixture.ts
> export const authenticatedUser = base.extend({
>   loginPage: async ({ page }, use) => {
>     const loginPage = new LoginPage(page);
>     await loginPage.login('user@example.com', 'password123');
>     await use(loginPage);
>   },
> });
>
> // tests/profile/edit-profile.spec.ts
> test('User edits profile', async ({ loginPage, page }) => {
>   // Already logged in, ready to test
>   await page.getByTestId('profile-btn').click();
>   // ... test actual feature
> });
> ```
>
> **3. ORGANIZED BY FEATURE**
>
> NOT: tests/positive/, tests/negative/, tests/smoke/
> (Mixes related features)
>
> YES: tests/auth/, tests/dashboard/, tests/payments/
> (All related tests together)
>
> Why? New dev joins. They want to modify the dashboard feature. They open tests/dashboard/ and find everything. Not scattered across 5 folders.
>
> **4. REUSABLE UTILITIES**
>
> ```typescript
> // utils/apiHelpers.ts
> export async function createUserViaAPI(email: string, password: string) {
>   const response = await request.post('/api/users', {
>     data: { email, password }
>   });
>   return response.json();
> }
>
> // Now used in fixtures and tests
> // 300+ tests can reuse this without duplicating
> ```
>
> **5. CENTRALIZED CONFIGURATION**
>
> One source of truth:
> ```typescript
> // config/constants.ts
> export const TIMEOUTS = {
>   NAVIGATION: 30000,
>   ELEMENT_VISIBLE: 5000,
> };
>
> export const API_BASE_URL = process.env.API_URL || 'http://localhost:3000';
>
> // If timeouts need to change, update once
> ```
>
> **At NielsenIQ:**
>
> Our framework decisions:
> - ✅ Feature-based folder structure
> - ✅ Page Object Model for all pages
> - ✅ Fixtures for setup/teardown
> - ✅ Utilities for API, DB, helpers
> - ✅ Tests are 5-10 lines (pure behavior description)
> - ✅ One setup in fixture, used by 50+ tests
>
> Result:
> - ✅ 300+ tests, easy to maintain
> - ✅ New SDET writes tests in 30 minutes
> - ✅ Change login flow → update LoginPage once → 50+ tests adapt
> - ✅ Debug time: 5-10 minutes (not 2 hours)"

---

## Framework Design Principles

| Principle | What It Means | Why It Matters |
|-----------|--------------|----------------|
| **DRY** | Don't Repeat Yourself | Write once, use 50 times (in fixtures/utils) |
| **Separation of Concerns** | Tests describe WHAT, pages describe HOW | Change page → tests still pass |
| **Reusability** | Fixtures, utilities, page objects | Scale from 100 to 10,000 tests |
| **Maintainability** | Clear structure, organized folders | New dev understands code in 30 min |
| **Scalability** | Add 100 tests without complexity | Framework grows with codebase |

---

## Your Real Project Example (NielsenIQ)

**Before (Poor Framework):**
```typescript
// Every test duplicates login logic
test('User creates project', async ({ page }) => {
  await page.goto('/login');
  await page.getByTestId('email').fill('user@example.com');
  await page.getByTestId('password').fill('password123');
  await page.click('button:has-text("Login")');
  await page.waitForNavigation();
  
  // NOW test the actual feature
  await page.click('button:has-text("Create Project")');
  // ...
});

test('User deletes project', async ({ page }) => {
  await page.goto('/login');  // Duplicated!
  await page.getByTestId('email').fill('user@example.com');  // Duplicated!
  // ...
});

// 300+ tests, EACH has login logic duplicated
// Total: 500+ lines of duplicated setup code
// Change login flow → Update 300+ tests
```

**After (Good Framework):**
```typescript
// fixtures/auth.fixture.ts
export const authenticatedUser = base.extend({
  page: async ({ page }, use) => {
    const login = new LoginPage(page);
    await login.login('user@example.com', 'password123');
    await use(page);
  },
});

// tests/projects/create-project.spec.ts
test('User creates project', async ({ page }) => {
  // Already logged in!
  await page.click('button:has-text("Create Project")');
  // ...
});

test('User deletes project', async ({ page }) => {
  // Already logged in!
  await page.click('project-row');
  // ...
});

// Change login flow → Update LoginPage once → All tests adapt
```

**The Win:**
- Reduced test setup code by 500+ lines
- Login change → Update ONE place, not 300+
- New SDET productivity: 30 min vs 3 days

---

## Common Interview Questions & Answers

### Q1: "How do you keep tests maintainable as they grow?"

**A:** "Framework design. Separate concerns: tests describe WHAT to verify, page objects describe HOW to interact. Organize by feature, not test type. Use fixtures for setup/teardown, utilities for reusable logic. At 300+ tests, every line of code is reused by multiple tests - no duplication."

### Q2: "A developer changes the login flow. What happens to your tests?"

**A:** "One update in LoginPage, all 50+ login tests adapt. That's the power of Page Object Model. If tests were scattered with duplicate login logic, I'd update 50+ places - error-prone and slow."

### Q3: "What's your folder structure?"

**A:** "Feature-based. tests/auth/, tests/dashboard/, tests/payments/. Each feature folder has all related tests. pages/ has page objects, fixtures/ has setup/teardown, utils/ has helpers. New dev opens tests/dashboard/ and finds everything related to dashboard."

### Q4: "How do you scale from 100 to 1000 tests?"

**A:** "Good framework design makes scaling automatic. If I use fixtures for setup and utilities for helpers, adding 900 more tests doesn't require structural changes. The infrastructure already exists."

### Q5: "How do you get new team members up to speed?"

**A:** "Framework is self-documenting. Open tests/auth/login.spec.ts → See what tests are doing. Open pages/LoginPage.ts → See how page object works. Open fixtures/auth.fixture.ts → See setup pattern. In 30 minutes, they can write their first test."

---

## What You Should Be Ready to Explain

✅ Why framework design matters (not just "running tests")  
✅ Your folder structure at NielsenIQ (feature-based)  
✅ How Page Object Model reduces duplication  
✅ Why fixtures are better than beforeEach hooks  
✅ How changes propagate (update once, tests adapt)  
✅ Your 300+ test scaling strategy  

---

## Weak Areas to Watch

❌ **Don't say:** "I just put all tests in one folder"  
→ **Reality:** Structure matters as tests grow

❌ **Don't repeat:** Login logic in every test  
→ **Reality:** Use fixtures to write once, use 50 times

❌ **Don't mix:** Test logic with page interaction  
→ **Reality:** Tests describe WHAT, pages describe HOW

❌ **Don't hardcode:** URLs, timeouts, credentials  
→ **Reality:** Use configuration files for everything

---

## Key Takeaway for Interview

Good framework design is invisible - tests are simple, setup is clean, changes propagate automatically. Bad framework design is visible - tests are verbose, setup is scattered, changes break everything.

Your framework design shows if you think about long-term maintainability, not just "making tests work today."

**At NielsenIQ:** Your architecture is why 300+ tests remain maintainable. That's a competitive advantage. ✨

---

## Framework Architecture Checklist

✅ Feature-based folder organization  
✅ Page Object Model for all pages  
✅ Fixtures for setup/teardown (not beforeEach)  
✅ Utilities for reusable logic (API, DB, helpers)  
✅ Centralized configuration  
✅ Tests are 5-10 lines (pure behavior)  
✅ One change → Tests adapt automatically  
✅ New dev can write tests in 30 minutes  

If you have all 8, your framework is solid.

---

**Remember:** Framework design is about making the next person's life easier (often your future self). Build for scale, not for today. 🎯
