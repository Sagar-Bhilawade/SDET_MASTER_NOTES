# Playwright - Test Organization: Folder Structure & Naming Conventions

## What is Test Organization?

How you organize folders and name files determines if teammates can find tests and add new ones easily.

Bad organization: tests scattered everywhere, unclear naming, impossible to find related tests.
Good organization: feature-based folders, clear naming patterns, anyone can add tests in 5 minutes.

---

## Why It Matters (Interview Context)

Interviewers ask about organization because:
1. **Shows scalability thinking** (100 tests? 1000 tests? How do you scale?)
2. **Demonstrates team collaboration** (can others find and modify your tests?)
3. **Proves structural discipline** (not just "running tests")

Organization matters as much as the tests themselves.

---

## Your 30-Second Interview Answer

> "At NielsenIQ, tests are organized by feature: tests/auth/, tests/dashboard/, tests/payments/. Each feature has related tests together. Files named action-scenario.spec.ts (login-valid-credentials.spec.ts). Page objects in pages/PascalCase.ts, utilities in utils/camelCase.ts. New dev opens tests/auth/ and finds everything about authentication."

---

## Your 2-Minute Deep Answer

> "Organization is about making it easy to ADD tests, not just run them.
>
> **Bad Organization (Scattered):**
> ```
> tests/
>   ├── test1.spec.ts
>   ├── test2.spec.ts
>   ├── test3.spec.ts
>   ├── positive/
>   │   ├── login.spec.ts
>   │   ├── dashboard.spec.ts
>   ├── negative/
>   │   ├── login-error.spec.ts
>   │   ├── dashboard-error.spec.ts
>   ├── smoke/
>   │   ├── login.spec.ts
>   │   ├── dashboard.spec.ts
>
> Problem: Login tests are in 3 places!
> Feature spread across positive/negative/smoke.
> Impossible to find all login-related tests.
> ```
>
> **Good Organization (Feature-Based):**
> ```
> tests/
>   ├── auth/
>   │   ├── login-valid-credentials.spec.ts
>   │   ├── login-invalid-password.spec.ts
>   │   ├── login-user-locked.spec.ts
>   │   ├── logout.spec.ts
>   │   ├── password-reset.spec.ts
>   ├── dashboard/
>   │   ├── display-widgets.spec.ts
>   │   ├── edit-widget-preferences.spec.ts
>   │   ├── delete-widget.spec.ts
>   ├── payments/
>   │   ├── create-payment-valid-card.spec.ts
>   │   ├── create-payment-invalid-card.spec.ts
>   │   ├── refund-payment.spec.ts
>
> Benefit: All auth tests together. Easy to find, modify, add new.
> New dev: \"I need to add a password reset test.\" Opens tests/auth/, adds file.
> ```
>
> **File Naming Conventions:**
>
> **1. TEST FILES: action-scenario.spec.ts**
>
> ```
> ✅ login-valid-credentials.spec.ts  (action: login, scenario: valid credentials)
> ✅ create-payment-insufficient-funds.spec.ts  (action: create-payment, scenario: insufficient-funds)
> ✅ delete-widget-requires-confirmation.spec.ts
>
> ❌ test1.spec.ts  (unclear)
> ❌ login-test.spec.ts  (redundant \"test\")
> ❌ auth.spec.ts  (too vague)
> ```
>
> Why action-scenario?
> - Describes what the test does
> - Group similar tests visually (all login tests together)
> - Easy to search (grep for \"login-\")
>
> **2. PAGE OBJECTS: PascalCase.ts**
>
> ```
> pages/
>   ├── LoginPage.ts
>   ├── DashboardPage.ts
>   ├── PaymentPage.ts
>   ├── BasePage.ts
>
> ✅ Matches class names
> ✅ Clear hierarchy (classes are capitalized)
> ```
>
> **3. UTILITIES: camelCase.ts**
>
> ```
> utils/
>   ├── apiHelpers.ts
>   ├── dbHelpers.ts
>   ├── dateUtils.ts
>   ├── selectDropdownValue.ts
>
> ✅ Standard JavaScript naming
> ✅ Functions are lowercase
> ```
>
> **4. FIXTURES: descriptive-name.fixture.ts**
>
> ```
> fixtures/
>   ├── auth.fixture.ts
>   ├── database.fixture.ts
>   ├── api.fixture.ts
>
> ✅ Clear what the fixture does
> ✅ .fixture suffix distinguishes from other files
> ```
>
> **Folder Structure at NielsenIQ:**
>
> ```
> project/
>   ├── tests/
>   │   ├── auth/
>   │   │   ├── login-valid-credentials.spec.ts
>   │   │   ├── login-invalid-password.spec.ts
>   │   │   ├── logout.spec.ts
>   │   ├── dashboard/
>   │   │   ├── display-widgets.spec.ts
>   │   │   ├── edit-preferences.spec.ts
>   │   ├── payments/
>   │   │   ├── create-payment-valid.spec.ts
>   │   │   ├── create-payment-error.spec.ts
>   │
>   ├── pages/
>   │   ├── LoginPage.ts
>   │   ├── DashboardPage.ts
>   │   ├── PaymentPage.ts
>   │   ├── BasePage.ts
>   │
>   ├── fixtures/
>   │   ├── auth.fixture.ts
>   │   ├── database.fixture.ts
>   │   ├── api.fixture.ts
>   │
>   ├── utils/
>   │   ├── apiHelpers.ts
>   │   ├── dbHelpers.ts
>   │   ├── testData.ts
>   │
>   ├── data/
>   │   ├── testData.json
>   │   ├── users.json
>   │
>   ├── config/
>   │   ├── playwright.config.ts
>   │   ├── constants.ts
>   │
>   ├── playwright-report/  (Generated)
>   ├── test-results/       (Generated)
> ```
>
> **Why This Structure?**
>
> 1. **Feature-based**: Related tests grouped together
> 2. **Clear separation**: Tests don't mix with pages/utils
> 3. **Scalable**: Add 100 tests without restructuring
> 4. **Learnable**: New dev understands within 1 hour
> 5. **Findable**: grep for \"tests/dashboard/\" finds all dashboard tests
>
> **Best Practices:**
>
> **1. ONE TEST FILE PER FEATURE/SCENARIO**
> ```
> ✅ tests/auth/login-valid-credentials.spec.ts (Tests valid login)
> ✅ tests/auth/login-invalid-password.spec.ts (Tests invalid password)
>
> ❌ tests/auth/login-all-scenarios.spec.ts (Mixes scenarios)
> ```
>
> **2. KEEP TEST FILES FOCUSED**
> ```
> ✅ 5-10 tests per file (Related scenario tests)
>
> ❌ 50 tests per file (Too many, hard to navigate)
> ```
>
> **3. USE NESTED FOLDERS FOR LARGE FEATURES**
> ```
> tests/payments/
>   ├── create/
>   │   ├── valid-card.spec.ts
>   │   ├── invalid-card.spec.ts
>   ├── refund/
>   │   ├── refund-success.spec.ts
>   │   ├── refund-error.spec.ts
> ```
>
> **4. SHARED DATA IN SEPARATE FOLDER**
> ```
> data/
>   ├── users.json
>   ├── testData.json
>   ├── paymentCards.json
>
> Used by tests and fixtures
> Not mixed with test files
> ```"

---

## Folder Structure Template

```
project/
├── tests/                          # All test files
│   ├── auth/                       # Feature folder
│   │   ├── login-valid.spec.ts
│   │   ├── login-invalid.spec.ts
│   │   ├── logout.spec.ts
│   ├── dashboard/
│   │   ├── display-widgets.spec.ts
│   │   ├── edit-preferences.spec.ts
│   ├── payments/
│   │   ├── create-payment.spec.ts
│   │   ├── refund-payment.spec.ts
│
├── pages/                          # Page Objects
│   ├── LoginPage.ts
│   ├── DashboardPage.ts
│   ├── PaymentPage.ts
│   ├── BasePage.ts
│
├── fixtures/                       # Reusable setup
│   ├── auth.fixture.ts
│   ├── database.fixture.ts
│   ├── api.fixture.ts
│
├── utils/                          # Helper functions
│   ├── apiHelpers.ts
│   ├── dbHelpers.ts
│   ├── testData.ts
│
├── data/                           # Test data
│   ├── users.json
│   ├── payments.json
│
├── config/                         # Configuration
│   ├── playwright.config.ts
│   ├── constants.ts
│
├── playwright.config.ts            # Main config
├── package.json
```

---

## Naming Convention Summary

| Type | Pattern | Example |
|------|---------|---------|
| **Test files** | action-scenario.spec.ts | login-valid-credentials.spec.ts |
| **Page objects** | PascalCase.ts | LoginPage.ts |
| **Utilities** | camelCase.ts | apiHelpers.ts |
| **Fixtures** | name.fixture.ts | auth.fixture.ts |
| **Data files** | descriptive.json | users.json |

---

## Your Real Project Example (NielsenIQ)

**File structure:**
```
tests/
├── auth/
│   ├── login-valid-credentials.spec.ts
│   ├── login-invalid-password.spec.ts
│   ├── login-user-locked.spec.ts
│   ├── logout-clears-session.spec.ts
├── dashboard/
│   ├── display-user-widgets.spec.ts
│   ├── edit-widget-preferences.spec.ts
│   ├── delete-widget.spec.ts
```

**Why this works:**
- Login tests grouped together
- File names describe scenario (valid, invalid, locked)
- Easy to add new test (create file in auth/)
- Easy to find all dashboard tests
- New dev: \"I need login tests.\" Opens tests/auth/.

---

## Common Interview Questions & Answers

### Q1: "How do you organize tests as they grow?"

**A:** "Feature-based folders. tests/auth/, tests/dashboard/, etc. Each feature has all related tests together. Not by test type (positive/negative) - that scatters related tests. Scale from 100 to 1000 tests without restructuring."

### Q2: "What do you name your test files?"

**A:** "action-scenario.spec.ts format. login-valid-credentials.spec.ts describes what it tests. Makes files searchable and grouped logically. Also clear what failed if CI shows 'login-invalid-password.spec.ts failed'."

### Q3: "Should page objects be in tests/ folder?"

**A:** "No, separate pages/ folder. Tests describe WHAT, pages describe HOW. Don't mix them. Same for utils, fixtures, data. Clear separation makes code easier to navigate."

### Q4: "How do you handle shared test data?"

**A:** "Separate data/ folder with JSON files (users.json, paymentCards.json). Tests and fixtures import from there. Not hardcoded in tests, not scattered everywhere."

### Q5: "What if a feature has many tests?"

**A:** "Nested folders. tests/payments/create/ and tests/payments/refund/. Keep each file focused. 5-10 tests per file is ideal - larger files become hard to navigate."

---

## What You Should Be Ready to Explain

✅ Feature-based organization  
✅ Naming conventions (action-scenario.spec.ts)  
✅ Folder structure reasoning  
✅ Scalability (100 to 1000 tests)  
✅ How it enables team collaboration  
✅ Your structure at NielsenIQ  

---

## Weak Areas to Watch

❌ **Don't organize:** By test type (positive/negative)  
→ **Reality:** Organize by feature

❌ **Don't mix:** Tests, pages, utils in same folder  
→ **Reality:** Separate concerns into different folders

❌ **Don't name:** test1.spec.ts, login-test.spec.ts  
→ **Reality:** Use action-scenario format

❌ **Don't scatter:** Test data across files  
→ **Reality:** Centralize in data/ folder

---

## Key Takeaway for Interview

Organization is infrastructure. Good organization makes it easy to add tests, find tests, modify tests.

At 300+ tests, organization determines if you can maintain it or if it's chaos.

**At NielsenIQ:** Your feature-based structure scales. New dev adds tests in 5 minutes.

**Your competitive edge:** You think about scaling from day 1. ✨

---

**Remember:** Spend 1 hour organizing, save 100 hours maintaining. 🎯
