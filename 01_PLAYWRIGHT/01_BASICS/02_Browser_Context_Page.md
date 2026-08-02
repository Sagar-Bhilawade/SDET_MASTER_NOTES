# Playwright - Browser, Context, and Page Lifecycle

## What Are Browser, Context, and Page?

These are the three core concepts that form Playwright's architecture. Understanding their relationship is fundamental to everything else in Playwright.

```
BROWSER (Instance of the browser engine)
    ↓
CONTEXT (Isolated session within that browser)
    ↓
PAGE (A single tab/window within that context)
```

Think of it like:
- **Browser** = Opening Chrome/Firefox application
- **Context** = Opening a new incognito/private window
- **Page** = A tab within that window

Each level is isolated and independent.

---

## Why It Matters (Interview Context)

Interviewers ask this because:
1. **Shows you understand Playwright architecture** (not just "running tests")
2. **Explains multi-browser testing** (how you run Chrome, Firefox, Safari separately)
3. **Demonstrates isolation strategy** (why contexts are useful for parallel tests)
4. **Explains state management** (how you handle login/cookies/storage)

If you can't explain this clearly, you'll struggle explaining your framework design in interviews.

---

## Your 30-Second Interview Answer

> "In Playwright, Browser is the browser engine instance. Context is an isolated session within that browser, similar to an incognito window. Page is a single tab within that context. This architecture lets us run parallel tests with complete isolation - each test gets its own context and page, preventing test interference."

---

## Your 2-Minute Deep Answer

> "Playwright has a three-level hierarchy: Browser, Context, and Page.
>
> **Browser:** This is the browser engine instance - Chrome, Firefox, or Safari. Once you launch a browser, it stays open for the entire test run or until you explicitly close it.
>
> **Context:** A context is an isolated session within a browser. It's like opening an incognito window in Chrome. Each context has its own:
> - Cookies and storage
> - Authentication state
> - Browsing history
> - Network interception rules
>
> Multiple contexts can exist in one browser, and they don't share any state. This is crucial for parallel testing.
>
> **Page:** A page is a single tab or window within a context. When you navigate to a URL, interact with elements, or take screenshots, you're doing it on a specific page.
>
> **Why this matters at NielsenIQ:**
>
> We run 300+ tests in parallel on Azure DevOps VMs. If all tests shared the same browser/context/page, they'd interfere with each other - one test's login would affect another test's state, causing flakiness.
>
> Instead, we:
> 1. Launch one browser
> 2. Create separate contexts for each test (or share contexts if tests need same login)
> 3. Each context has its own page
> 4. Tests run in parallel without interference
>
> This architecture gives us stable, parallel execution. In Selenium, you'd have to create separate WebDriver instances, which is slower and heavier. Playwright's context model is lightweight and efficient."

---

## Architecture Diagram

```
BROWSER (One per session)
│
├── CONTEXT 1 (Test 1)
│   └── PAGE 1.1 (The test's tab)
│       ├── Cookies: [specific to context 1]
│       ├── Storage: [specific to context 1]
│       └── Logged in as: [user from context 1]
│
├── CONTEXT 2 (Test 2)
│   └── PAGE 2.1 (Another test's tab)
│       ├── Cookies: [specific to context 2]
│       ├── Storage: [specific to context 2]
│       └── Logged in as: [different user]
│
└── CONTEXT 3 (Test 3)
    └── PAGE 3.1 (Yet another test's tab)
        ├── Cookies: [completely isolated]
        ├── Storage: [isolated]
        └── Logged in as: [third user]

Result: All 3 tests run in parallel, no interference ✅
```

---

## Browser vs Context vs Page (Comparison)

| Aspect | Browser | Context | Page |
|--------|---------|---------|------|
| **What is it?** | Browser engine (Chrome/Firefox) | Isolated session in browser | Tab/window in context |
| **Lifetime** | Long (full test run) | Variable (per test or shared) | Per navigation/test |
| **State sharing** | Shared by all contexts | Isolated from other contexts | Isolated from other pages |
| **Cookies** | Not stored here | Stored per context | Accessed from context |
| **Performance** | Launch once | Lightweight to create | Very lightweight |
| **Use case** | One per test run | One per test (usually) | Main interaction point |

---

## Your Real Project Example (NielsenIQ)

**Our parallel test execution:**

```
Test Suite: 300+ tests
Azure DevOps VM: Running all tests in parallel

Architecture:
1 Browser (launched once)
    ↓
300+ Contexts (one per test, running simultaneously)
    ↓
300+ Pages (each test interacts with its own page)

Each test:
✅ Has its own isolated context (cookies, storage, auth separate)
✅ Has its own page to interact with
✅ Cannot interfere with other tests
✅ Runs in 8-10 minutes total (vs 25-30 min with Selenium)

Why this works:
- Context isolation prevents state leakage
- Multiple pages in one context share state (intentional)
- Parallel execution because each test is isolated
- Memory efficient (one browser, lightweight contexts)
```

**Contrast with Selenium:**
In Selenium, you'd create separate WebDriver instances for each test. That means:
- ❌ 300+ separate browser processes (heavy!)
- ❌ Takes 25-30 minutes to run
- ❌ High memory usage

Playwright's context model is why we can run 300+ tests in 8-10 minutes.

---

## Common Interview Questions & Answers

### Q1: "What's the difference between Context and Page?"

**A:** "Context is an isolated session in a browser, like an incognito window. Page is a tab within that context. You can have multiple pages within one context (they share cookies/storage), or one page per context (completely isolated). At NielsenIQ, we usually do one context per test for isolation."

### Q2: "Why do we need Context if we have Page?"

**A:** "Because multiple pages in the same context share state (cookies, storage). If you have multiple pages in one context, logging in on page 1 logs in page 2 too. But if you create separate contexts, each is completely isolated. For parallel tests, we need isolation, so we use separate contexts."

### Q3: "Can we launch one browser and run all tests in parallel?"

**A:** "Yes! That's exactly what we do at NielsenIQ. We launch one browser, create separate contexts for each parallel test, and each test gets its own page. This is much more efficient than Selenium's approach of one WebDriver per test."

### Q4: "What happens if a test doesn't close its context?"

**A:** "The context stays open consuming memory. Best practice: close contexts after each test (or use fixtures to auto-close). We handle this with Playwright fixtures in our framework."

### Q5: "How does storage state work with Context?"

**A:** "Storage state (cookies, localStorage) is specific to a context. You can save a context's storage after login and reuse it in other contexts via `storageState`. At NielsenIQ, we do this for tests that share the same user session."

---

## What You Should Be Ready to Explain

✅ Browser vs Context vs Page (the hierarchy)  
✅ Why context isolation matters (prevents test interference)  
✅ How parallel testing works (one browser, multiple contexts)  
✅ State sharing (what's shared, what's isolated)  
✅ Storage state and reusing login (optimization technique)  
✅ Your real example from NielsenIQ (300+ tests in parallel)  

---

## Weak Areas to Watch

❌ **Don't say:** "Browser and Context are the same thing"  
→ **Reality:** They're different levels in the hierarchy

❌ **Don't confuse:** "Page is like a Context"  
→ **Reality:** Page is a tab within Context, not a browser concept

❌ **Don't claim:** "We create one Context and run all tests in it"  
→ **Reality:** That would cause test interference; you need isolation

❌ **Don't overlook:** How this enables your 8-10 minute execution time  
→ **Reality:** Lightweight contexts are why Playwright is fast

---

## Key Takeaway for Interview

The Browser → Context → Page hierarchy is what makes Playwright powerful:

1. **Browser:** Heavy, launch once
2. **Context:** Lightweight, launch per test for isolation
3. **Page:** Where the test actually interacts

This is why you can run 300+ parallel tests in 8-10 minutes. Selenium can't do this because it requires separate browser instances.

**That's your competitive advantage in interviews:** You understand the architecture, not just the syntax. ✨

---

## Practical Code Pattern (For Reference)

```typescript
// This is NOT in your notes, but helps you understand the flow

// Launch browser once
const browser = await chromium.launch();

// Test 1: Create its own context and page
const context1 = await browser.newContext();
const page1 = await context1.newPage();
await page1.goto('https://example.com');
// ... run test 1
await context1.close();

// Test 2: Create its own context and page (in parallel)
const context2 = await browser.newContext();
const page2 = await context2.newPage();
await page2.goto('https://example.com');
// ... run test 2
await context2.close();

// Close browser after all tests
await browser.close();
```

This is the pattern that makes 300+ parallel tests possible!

---

**Remember:** When practicing, focus on explaining the ARCHITECTURE and WHY it matters for your project. Not the code syntax. 🎯
