# Playwright - Configuration: playwright.config.ts, forbid-only, retries, reporters

## What is Configuration?

Playwright configuration is how you define test runner behavior: which browsers to test, timeouts, retries, reporters, CI detection, etc.

All config lives in `playwright.config.ts` - one central place.

---

## Why It Matters (Interview Context)

Interviewers ask about config because:
1. **Shows CI/CD thinking** (retries on CI, forbid-only for QA)
2. **Demonstrates best practices** (timeout strategy, reporting)
3. **Explains production readiness** (handles flakiness gracefully)

Good config = tests pass reliably. Bad config = tests flake unexpectedly.

---

## Your 30-Second Interview Answer

> "Playwright config is the control center. forbid-only prevents test.only() from merging, retries handle flakiness on CI, reporters generate HTML reports. At NielsenIQ, we set forbid-only: true on CI, retries: 2 on CI only, reporters to generate artifacts. Config is environment-aware using process.env.CI."

---

## Your 2-Minute Deep Answer

> "playwright.config.ts is your test infrastructure as code.
>
> **The Problem Without Configuration:**
>
> Everyone runs tests differently:
> - Dev runs 1 retry, CI runs 0
> - Dev sees console output, CI has no reports
> - Dev tests Chrome, CI tests Chrome/Firefox/Safari
> - Dev has 30s timeout, CI has 10s timeout
> - Inconsistent behavior, hard to debug
>
> **The Solution: playwright.config.ts**
>
> ```typescript
> // playwright.config.ts
> export default defineConfig({
>   // 1. WHICH TESTS TO RUN
>   testDir: './tests',
>   testMatch: '**/*\\.spec\\.ts',
>
>   // 2. BROWSERS
>   projects: [
>     { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
>     { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
>     { name: 'webkit', use: { ...devices['Desktop Safari'] } },
>   ],
>
>   // 3. TIMEOUTS
>   use: {
>     navigationTimeout: 30000,
>     actionTimeout: 10000,
>   },
>
>   // 4. RETRIES (Flakiness Handling)
>   retries: process.env.CI ? 2 : 0,  // Retry on CI, not local
>
>   // 5. FORBID-ONLY (Prevent Accidental test.only)
>   forbidOnly: !!process.env.CI,  // Error if test.only() on CI
>
>   // 6. WORKERS (Parallelization)
>   workers: process.env.CI ? 1 : undefined,  // 1 worker on CI (safer), default locally
>
>   // 7. REPORTERS (Test Output)
>   reporter: [
>     ['html'],  // HTML report
>     ['json', { outputFile: 'test-results.json' }],
>     ['junit', { outputFile: 'junit.xml' }],  // For CI
>   ],
>
>   // 8. DEBUGGING (Captures)
>   use: {
>     screenshot: 'only-on-failure',
>     video: 'retain-on-failure',
>     trace: 'on-first-retry',
>   },
> });
> ```
>
> **Key Configuration Concepts:**
>
> **1. RETRIES (Flakiness Management)**
>
> ```typescript
> retries: process.env.CI ? 2 : 0
> ```
>
> Why?
> - Local dev: 0 retries. Fail fast, debug immediately.
> - CI: 2 retries. Handle flakiness gracefully (network latency, timing).
>
> Pattern:
> ```
> Test fails → Retry 1 → If still fails → Retry 2 → If still fails → Mark failed
> ```
>
> **2. FORBID-ONLY (Prevent Accidental test.only())**
>
> ```typescript
> forbidOnly: !!process.env.CI
> ```
>
> Problem: Dev adds test.only() for debugging, forgets to remove, commits it.
> CI runs only THAT test, thinks all tests pass.
> Later, main branch test suite is broken.
>
> Solution: forbid-only: true on CI
> If anyone commits test.only(), CI fails immediately with error:
> \"test.only() is not allowed on CI\"
>
> Dev fixes it, re-runs. Done.
>
> **3. WORKERS (Parallel Execution)**
>
> ```typescript
> workers: process.env.CI ? 1 : 4  // 1 worker on CI (stability), 4 local (speed)
> ```
>
> Why different settings?
> - Local: Faster feedback. Use multiple workers.
> - CI: Stability matters. Use 1 worker to avoid resource contention.
>
> Actually, modern CI has plenty of power. This is optional optimization.
>
> **4. REPORTERS (Test Results)**
>
> ```typescript
> reporter: [
>   ['html'],  // HTML report (view in browser)
>   ['json'],  // JSON (parse programmatically)
>   ['junit'],  // JUnit XML (for Jenkins, etc.)
>   ['dot'],   // Console output (minimal)
> ]
> ```
>
> Which to use?
> - Local: 'html' (pretty, clickable) + 'dot' (quick feedback)
> - CI: 'html' (artifact), 'junit' (integrate with CI), 'json' (custom tools)
>
> **5. TIMEOUT CONFIGURATION**
>
> ```typescript
> use: {
>   navigationTimeout: 30000,  // How long to wait for page.goto()
>   actionTimeout: 10000,      // How long to wait for click(), fill()
> }
> ```
>
> These are GLOBAL defaults. You can override per test:
> ```typescript
> test.setTimeout(60000);  // This test gets 60s
> ```
>
> **6. BASE URL (For Tests)**
>
> ```typescript
> use: {
>   baseURL: process.env.BASE_URL || 'http://localhost:3000',
> }
> ```
>
> Then in tests:
> ```typescript
> await page.goto('/login');  // Becomes http://localhost:3000/login
> ```
>
> **At NielsenIQ:**
> ```typescript
> export default defineConfig({
>   testDir: './tests',
>   testMatch: '**/*\\.spec\\.ts',
>
>   projects: [
>     { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
>     { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
>   ],
>
>   use: {
>     baseURL: process.env.BASE_URL || 'http://localhost:3000',
>     navigationTimeout: 30000,
>     actionTimeout: 10000,
>     screenshot: 'only-on-failure',
>     video: 'retain-on-failure',
>     trace: 'on-first-retry',
>   },
>
>   retries: process.env.CI ? 2 : 0,
>   forbidOnly: !!process.env.CI,
>   workers: process.env.CI ? 1 : 4,
>
>   reporter: [
>     ['html'],
>     ['json', { outputFile: 'test-results/results.json' }],
>     ['junit', { outputFile: 'test-results/junit.xml' }],
>   ],
> });
> ```
>
> **Result:**
> - ✅ Consistent behavior everywhere (local, CI)
> - ✅ Retries handle flakiness automatically
> - ✅ forbid-only catches mistakes
> - ✅ HTML reports for debugging
> - ✅ JUnit XML for CI integration"

---

## Key Configuration Options

| Option | Default | CI Value | Why |
|--------|---------|----------|-----|
| **retries** | 0 | 2 | Handle flakiness on CI |
| **forbidOnly** | false | true | Prevent test.only() accidents |
| **workers** | CPU count | 1 | Stability on CI |
| **timeout** | 30s | Same | Consistent across environments |
| **screenshot** | off | only-on-failure | Debug failures |
| **trace** | off | on-first-retry | Detailed failure logs |

---

## Your Real Project Example (NielsenIQ)

**playwright.config.ts:**
```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  testMatch: '**/*.spec.ts',

  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
  ],

  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    navigationTimeout: 30000,
    actionTimeout: 10000,
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'on-first-retry',
  },

  retries: process.env.CI ? 2 : 0,
  forbidOnly: !!process.env.CI,
  workers: process.env.CI ? 1 : 4,

  reporter: [
    ['html', { outputFolder: 'playwright-report' }],
    ['json', { outputFile: 'test-results/results.json' }],
    ['junit', { outputFile: 'test-results/junit.xml' }],
  ],

  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

**Result:**
- Local: 4 workers, 0 retries, fast feedback
- CI: 1 worker, 2 retries, stable, HTML/JUnit reports
- forbid-only prevents test.only() mistakes
- Base URL keeps tests clean (no hardcoded URLs)

---

## Common Interview Questions & Answers

### Q1: "Why forbid-only on CI?"

**A:** "Developer adds test.only() for local debugging, commits by accident. CI runs only that test, passes. Main branch test suite is actually broken. forbidOnly: true catches this immediately with an error."

### Q2: "Why different retry counts locally vs CI?"

**A:** "Local: fail fast for quick debugging. CI: 2 retries because CI has network latency, resource contention. If test fails consistently (real bug), it fails after retries. If it's timing flakiness, retries fix it."

### Q3: "What's the difference between trace and video?"

**A:** "Video is a browser recording (what user sees). Trace is a complete execution log (every action, network call, assertion). Video is easier to understand, trace is more detailed. Both on-first-retry for debugging failures."

### Q4: "Why configure baseURL in config instead of hardcoding in tests?"

**A:** "Flexibility. Same tests run against localhost, staging, prod. Just change baseURL. Tests stay clean - no URL logic. Also easier to parallelize (each test environment has its own config)."

### Q5: "Should you use different reporters for local vs CI?"

**A:** "Yes. Local: HTML (pretty, browsable). CI: JUnit (integrates with CI systems), JSON (custom tools). HTML is great for debugging locally but heavy on CI."

---

## What You Should Be Ready to Explain

✅ Why retries are environment-aware (CI vs local)  
✅ forbid-only prevents accidents  
✅ Reporter strategy (HTML, JUnit, JSON)  
✅ Timeout configuration  
✅ Why baseURL matters  
✅ Your config strategy at NielsenIQ  

---

## Weak Areas to Watch

❌ **Don't ignore:** forbid-only on CI  
→ **Reality:** Catches test.only() accidents

❌ **Don't set:** High retries locally  
→ **Reality:** Fail fast during development

❌ **Don't hardcode:** URLs in tests  
→ **Reality:** Use baseURL for flexibility

❌ **Don't skip:** Video/trace on failures  
→ **Reality:** Essential for debugging CI failures

---

## Key Takeaway for Interview

Configuration is how you standardize test behavior across environments. Good config makes tests reliable and debuggable.

**At NielsenIQ:** Your config prevents mistakes (forbid-only), handles flakiness (retries), and provides debugging tools (traces).

**Your competitive edge:** Configuration is infrastructure thinking, not just test writing. ✨

---

**Remember:** One config file, consistent behavior everywhere. 🎯
