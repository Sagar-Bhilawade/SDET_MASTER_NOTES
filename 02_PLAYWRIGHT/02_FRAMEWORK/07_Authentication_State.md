# Playwright - Authentication & Storage State: Reusing Login Sessions

## What is Storage State?

Storage state is the authenticated session (cookies, localStorage, sessionStorage) saved to disk. Instead of logging in before every test, load the saved session - 10x faster.

Basic flow:
```
1. Setup project: Log in once, save storage state to file
2. Main projects: Load storage state, tests start already logged in
```

---

## Why It Matters (Interview Context)

Interviewers ask about storage state because:
1. **Shows CI/CD thinking** (tests need to be fast)
2. **Demonstrates scalability** (login 300 times vs login once)
3. **Explains architectural patterns** (separation of concerns)

Storage state is THE pattern for fast, maintainable test suites.

---

## Your 30-Second Interview Answer

> "At NielsenIQ, we log in once during a setup project, save the authenticated session to storage state, and every test loads it. Instead of paying a few seconds of login per test across the whole suite, we pay it once and each test loads the session in milliseconds. The storage state file holds cookies and localStorage, wired in through the config so it's automatic."

---

## Your 2-Minute Deep Answer

> "Storage state is how you avoid login duplication.
>
> **The Problem Without Storage State:**
>
> ```typescript
> // ❌ Every test logs in (300 tests × 1 second login = 300 seconds wasted)
> test('User creates project', async ({ page }) => {
>   await login(page, 'user@example.com', 'password');  // 1 second
>   await page.goto('/dashboard');
>   await page.getByRole('button', { name: 'Create' }).click();
>   // ... test logic
> });
>
> test('User edits project', async ({ page }) => {
>   await login(page, 'user@example.com', 'password');  // 1 second (duplicated!)
>   await page.goto('/dashboard');
>   // ... test logic
> });
>
> // 300 tests × 1 second login = 5 minutes wasted on redundant logins!
> ```
>
> **The Solution: Storage State**
>
> **Step 1: Setup Project (Once)**
> ```typescript
> // playwright.config.ts
> export default defineConfig({
>   projects: [
>     {
>       name: 'setup',
>       testMatch: /auth\\.setup\\.ts/,
>     },
>     {
>       name: 'chromium',
>       use: {
>         storageState: 'auth.json',  // Load saved session
>       },
>       dependencies: ['setup'],  // Run setup first
>     },
>   ],
> });
> ```
>
> **Step 2: Setup Test (Runs Once)**
> ```typescript
> // tests/auth.setup.ts
> import { test as setup } from '@playwright/test';
>
> setup('Authenticate user', async ({ page, context }) => {
>   // Navigate to app
>   await page.goto('http://localhost:3000');
>
>   // Fill login form
>   await page.getByTestId('email').fill('user@example.com');
>   await page.getByTestId('password').fill('password123');
>   await page.getByRole('button', { name: 'Login' }).click();
>
>   // Wait for redirect to dashboard
>   await page.waitForURL('**/dashboard');
>
>   // Save authenticated session to file
>   await context.storageState({ path: 'auth.json' });
> });
> ```
>
> **Step 3: Use in Fixtures**
> ```typescript
> // fixtures/auth.fixture.ts
> export const authenticatedUser = base.extend({
>   page: async ({ page }, use) => {
>     // Storage state already loaded (from config)
>     // Page already has cookies/localStorage from login
>     // No additional login needed!
>     await use(page);
>   },
> });
> ```
>
> **Step 4: Tests Use It**
> ```typescript
> // tests/dashboard/dashboard.spec.ts
> test('User creates project', async ({ page }) => {
>   // Already logged in! (storage state loaded)
>   await page.goto('/dashboard');
>   await page.getByRole('button', { name: 'Create' }).click();
>   // ... test logic
> });
>
> test('User edits project', async ({ page }) => {
>   // Already logged in! (storage state loaded)
>   await page.goto('/dashboard');
>   // ... test logic
> });
>
> // Result: No login duplication, 300 tests run fast!
> ```
>
> **What Gets Saved in Storage State?**
>
> ```json
> // auth.json (saved automatically)
> {
>   \"cookies\": [
>     {
>       \"name\": \"sessionId\",
>       \"value\": \"abc123...\",
>       \"domain\": \"localhost\",
>       \"path\": \"/\",
>       \"httpOnly\": true
>     }
>   ],
>   \"origins\": [
>     {
>       \"origin\": \"http://localhost:3000\",
>       \"localStorage\": [
>         {
>           \"name\": \"theme\",
>           \"value\": \"dark\"
>         }
>       ],
>       \"sessionStorage\": []
>     }
>   ]
> }
> ```
>
> **At NielsenIQ:**
> ```typescript
> // playwright.config.ts
> export default defineConfig({
>   projects: [
>     {
>       name: 'setup',
>       testMatch: /auth\\.setup\\.ts/,
>     },
>     {
>       name: 'chromium',
>       use: {
>         storageState: '.auth/chromium.json',  // Separate per browser
>       },
>       dependencies: ['setup'],
>     },
>     {
>       name: 'firefox',
>       use: {
>         storageState: '.auth/firefox.json',
>       },
>       dependencies: ['setup'],
>     },
>   ],
> });
>
> // tests/auth.setup.ts
> setup('Authenticate', async ({ page, context }) => {
>   await page.goto(BASE_URL);
>   const login = new LoginPage(page);
>   await login.login('sdet@example.com', 'password123');
>   await page.waitForURL('/dashboard');
>   await context.storageState({ path: '.auth/chromium.json' });
> });
> ```
>
> **Result:**
> - ✅ Login once per browser (not per test)
> - ✅ 300 tests load session in 30 ms (vs 1000 ms login)
> - ✅ Test execution 30x faster
> - ✅ Reliable (no login flakiness)
> - ✅ Clean (each test file doesn't have login code)"

---

## Storage State Architecture

```
Project Setup (Runs First)
│
├── tests/auth.setup.ts
│   ├── Navigate to app
│   ├── Login (email + password)
│   ├── Wait for redirect
│   └── Save storage state → .auth/chromium.json
│
Main Projects (Run After)
│
├── tests/dashboard/
│   ├── Load .auth/chromium.json (cookies + localStorage)
│   ├── Already authenticated
│   └── Tests start at /dashboard
│
└── tests/payments/
    ├── Load .auth/chromium.json
    ├── Already authenticated
    └── Tests start at /payments
```

---

## Your Real Project Example (NielsenIQ)

**playwright.config.ts:**
```typescript
export default defineConfig({
  projects: [
    {
      name: 'setup auth',
      testMatch: /auth\.setup\.ts/,
    },
    {
      name: 'chromium',
      use: {
        baseURL: 'http://localhost:3000',
        storageState: '.auth/chromium.json',
      },
      dependencies: ['setup auth'],
    },
  ],
});
```

**tests/auth.setup.ts:**
```typescript
import { test as setup } from '@playwright/test';

setup('Authenticate user', async ({ page, context }) => {
  await page.goto('/login');
  
  const login = new LoginPage(page);
  await login.login('sdet@example.com', 'password123');
  
  await page.waitForURL('/dashboard');
  
  // Save session
  await context.storageState({ path: '.auth/chromium.json' });
});
```

**Tests automatically loaded with session:**
```typescript
test('User creates project', async ({ page }) => {
  // Already logged in!
  await page.goto('/dashboard');
  await expect(page.getByText('Welcome, SDET')).toBeVisible();
});
```

**Result:** whole suite, one login.

---

## Common Interview Questions & Answers

### Q1: "What if storage state expires?"

**A:** "Setup project re-runs before tests. If session expired, login generates new one, saves new storage state. Tests always use fresh session."

### Q2: "How do you handle different user roles?"

**A:** "Multiple storage state files. login-admin.json, login-user.json. Different projects load different states. Or setup multiple users, save multiple files."

### Q3: "What if you need to test logout?"

**A:** "Logout tests use a separate context without storage state loaded. They test the logout flow independently. Main tests use authenticated state."

### Q4: "Does storage state work across browsers?"

**A:** "Partially. Cookies transfer, but browser-specific storage might not. Best practice: separate storage state files per browser (.auth/chromium.json, .auth/firefox.json)."

### Q5: "How often should you regenerate storage state?"

**A:** "It regenerates before every test run (setup project runs first). Session lifetime is usually days/weeks, so one regeneration per CI run is fine."

---

## What You Should Be Ready to Explain

✅ Why storage state eliminates login duplication  
✅ Setup project pattern (runs once)  
✅ How tests load storage state from config  
✅ What gets saved (cookies, localStorage)  
✅ Multiple storage states (different users)  
✅ Your implementation at NielsenIQ (whole suite, one login)  

---

## Weak Areas to Watch

❌ **Don't login:** In every test  
→ **Reality:** Use storage state for speed

❌ **Don't hardcode:** Login credentials  
→ **Reality:** Use environment variables

❌ **Don't share:** One storage state across browsers  
→ **Reality:** Separate files per browser

❌ **Don't forget:** Setup project in config  
→ **Reality:** Config dependencies ensure setup runs first

---

## Key Takeaway for Interview

Storage state is how enterprise test suites stay fast. Without it, login duplication kills performance at scale.

Your storage state strategy shows if you think about test efficiency, not just test correctness.

**At NielsenIQ:** Login once per 300 tests. That's engineering thinking.

**Your competitive edge:** You optimize for scale, not just features. ✨

---

**Remember:** Login once, test 300 times. 🎯
