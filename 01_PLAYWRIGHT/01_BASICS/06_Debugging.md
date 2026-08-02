# Playwright - Debugging: Finding & Fixing Test Failures

## What is Debugging?

Debugging is figuring out WHY a test failed. Playwright gives you powerful tools to see exactly what happened:
- **Videos** → Watch the test run
- **Traces** → Complete execution log (every action, assertion, network call)
- **Screenshots** → Snapshots at failure point
- **Inspector** → Step through test line-by-line
- **Logs** → Console output and test runner logs

You're not guessing. You can SEE what went wrong.

---

## Why It Matters (Interview Context)

Interviewers ask about debugging because:
1. **Shows you can troubleshoot independently** (not just "it failed, don't know why")
2. **Demonstrates test framework maturity** (good frameworks have debugging built-in)
3. **Explains flaky test diagnosis** (how you find timing vs real bugs)
4. **Shows production readiness** (your team can maintain tests without you)

If you can't debug tests, you can't maintain them.

---

## Your 30-Second Interview Answer

> "Playwright has built-in debugging: videos, traces, screenshots. When a test fails in CI, I download the trace file and replay it locally using `npx playwright show-trace`. This shows every action, assertion, network call - exactly what happened. At NielsenIQ, traces reduced debug time from 2+ hours per failure to 5-10 minutes. No guessing."

---

## Your 2-Minute Deep Answer

> "Playwright's debugging is phenomenal because you can see everything that happened.
>
> **The Problem with Other Frameworks:**
>
> Selenium gives you:
> - Exception message: 'StaleElementReference'
> - Screenshot of the failure
> - You spend 2+ hours guessing what went wrong
>
> Playwright gives you:
> - Video of the entire test run
> - Trace file (complete execution log)
> - Inspector (step through line by line)
> - Network log (all API calls)
> - Console logs (JavaScript errors)
>
> **Debugging Strategy at NielsenIQ:**
>
> **1. LOCAL DEBUGGING (During Development)**
>
> **Use the Inspector:**
> ```bash
> npx playwright test --debug
> ```
> This opens the Inspector:
> - Step through each line of your test
> - Hover over locators to highlight them
> - See exactly where actions are clicking
> - Pause and resume as needed
>
> Example: Test is clicking the wrong button?
> - Open inspector
> - Step to the click() line
> - Hover over the locator in the code
> - See highlighted element in browser
> - Fix the locator
>
> **Use Headed Mode:**
> ```bash
> npx playwright test --headed
> ```
> Watch the browser as tests run. See what's happening in real-time.
>
> **2. CI DEBUGGING (After Test Fails)**
>
> When a test fails in CI:
> - Download the trace file
> - Open locally: `npx playwright show-trace trace.zip`
> - Replay the entire test execution
> - See every action, assertion, network call
>
> **What the trace shows:**
> - Network requests/responses (API calls)
> - Console messages (JavaScript errors)
> - Screenshots at each step
> - Timing information (which step took how long)
> - Exact assertion that failed
>
> **Example: Login test fails in CI**
> ```
> Trace shows:
> ✅ Email field filled
> ✅ Password field filled
> ✅ Submit button clicked
> ❌ Assert URL is /dashboard → Failed
>   But page URL is still /login
> → Problem: Login API returned error
>   (Trace shows API response: {\"error\": \"Invalid credentials\"})
> → Root cause: Test data expired or credentials wrong
> ```
>
> Without trace, you'd just see:
> \"Expected URL /dashboard but was /login\"
> You'd spend hours investigating.
>
> **3. DEBUGGING FLAKY TESTS**
>
> \"Sometimes tests pass, sometimes fail.\" → Flakiness
>
> Playwright helps identify the cause:
>
> **Timing Issues:**
> - Trace shows: Action at 2.3s, Element appears at 2.5s
> - Cause: Element not ready when clicked
> - Fix: Auto-wait is already built-in; check if selector is correct
>
> **State Issues:**
> - Trace shows: Test data is stale
> - Cause: Database not reset between runs
> - Fix: Add setup/teardown to reset state
>
> **Race Conditions:**
> - Trace shows: Network request succeeds sometimes, fails other times
> - Cause: External API or backend timing
> - Fix: Add explicit waits for network state
>
> **Example: Search test is flaky**
> ```
> Sometimes passes: API responds in 1s → test continues quickly
> Sometimes fails: API responds in 3s → assertion times out
>
> Fix: Add network wait
> await page.waitForLoadState('networkidle');  // Wait for all network to settle
> await expect(page.locator('[data-testid=\"results\"]')).toBeVisible();
> ```
>
> **4. DEBUGGING ASSERTIONS**
>
> When an assertion fails:
> ```typescript
> await expect(page.getByTestId('message')).toContainText('Success');
> // Fails: Expected 'Success' but page contains 'Error: Server timeout'
> ```
>
> Open trace:
> - See exact content of the element
> - See screenshot at failure point
> - See if element was even visible
> - See network errors before the assertion
>
> **5. VIDEO RECORDING (Simple Debugging)**
>
> Configure video recording:
> ```typescript
> // playwright.config.ts
> use: {
>   video: 'on-failure',  // Record only failed tests
> }
> ```
>
> Watch the video:
> - See test run in real browser
> - See what user would see
> - Much clearer than logs
>
> **Common Debugging Scenarios at NielsenIQ:**
>
> **Scenario 1: \"Button click did nothing\"**
> - Open trace
> - See button was actually disabled
> - Check assertion that enables button
> - Fix: Wait for button to be enabled before clicking
>
> **Scenario 2: \"Wrong element selected\"**
> - Open inspector
> - Hover over locator
> - See which element highlighted
> - Fix: Adjust locator or add data-testid
>
> **Scenario 3: \"Test passes locally, fails in CI\"**
> - Compare local trace with CI trace
> - Check if data/environment is different
> - Check API responses are different
> - Fix: Add data setup or handle both scenarios
>
> **Scenario 4: \"Form submission hangs\"**
> - Open trace
> - See network request never completes
> - Check if backend is down
> - Fix: Add timeout and error handling
>
> **Result at NielsenIQ:**
> - ✅ Debug time: 5-10 minutes per failure (vs 2+ hours with Selenium)
> - ✅ Flaky tests: Identified and fixed in 30 minutes
> - ✅ Root cause: Always found, never guessing"

---

## Debugging Tools Comparison

| Tool | What It Shows | When to Use | Time to Debug |
|------|---------------|------------|---------------|
| **Inspector** | Step-by-step execution with browser | During development | 5-10 min |
| **Trace** | Complete execution log (actions, network, assertions) | After CI failure | 10-15 min |
| **Video** | Browser playback of test run | Visual debugging | 5-10 min |
| **Screenshot** | Single snapshot at failure | Quick check | 2-3 min |
| **Headed Mode** | Watch browser as test runs | Real-time feedback | 10-20 min |
| **Logs** | Console + test runner output | Network/API issues | 5 min |

---

## Your Real Project Example (NielsenIQ)

**Scenario: Payment test fails in CI**

```
Test failure message:
❌ Expected page to navigate to /order-confirmation but stayed on /payment

Without Playwright debugging (Selenium approach):
- 2 hours of investigation
- Check test data
- Check environment
- Check backend logs
- Still don't know why

With Playwright debugging (Our approach):
```

**Step 1: Download trace file from CI**
```bash
# CI downloads: trace-payment-test.zip
```

**Step 2: Replay trace locally**
```bash
npx playwright show-trace trace-payment-test.zip
```

**Step 3: Open trace and see:**
```
✅ Navigate to payment page
✅ Fill card details
✅ Click submit button
❌ Assert navigation to /order-confirmation → FAILED

Trace shows:
- Network request: POST /api/payment
- Response: 402 Payment Required (insufficient funds)
- Page stays on /payment (correct behavior)

Root cause: Test credit card has no balance
Fix: Use test data with sufficient balance
```

**Time: 5 minutes diagnosis**

---

**Another Scenario: Login test flaky (50% pass rate)**

```
Without debugging: "Test is unreliable, disable it"

With Playwright debugging:
```

**Successful run trace:**
- POST /api/login → Response in 0.8s ✅
- Navigation to /dashboard → 1.2s ✅
- All assertions pass

**Failed run trace:**
- POST /api/login → Response in 3.5s (slow backend)
- Auto-wait timeout (5s default) → Still waiting
- Navigation finally happens at 3.6s
- Assertions pass (but took longer than test timeout)

**Root cause:** Backend sometimes slow

**Fix:** Increase timeout or add network wait
```typescript
await page.waitForLoadState('networkidle');
```

**Time: 10 minutes diagnosis + fix**

---

## Debugging Commands (Interview Ready)

| Command | What It Does | Interview Use |
|---------|-------------|---------------|
| `npx playwright test --debug` | Opens inspector, step through test | "I debug locally with inspector" |
| `npx playwright test --headed` | Watch browser as test runs | "I watch tests in headed mode during dev" |
| `npx playwright show-trace <file>` | Replay CI failure locally | "I download traces from CI and replay them" |
| `npx playwright test --grep @flaky` | Run only flaky tests | "I isolate flaky tests for debugging" |
| `npx playwright test --project=chromium` | Run on specific browser | "I reproduce issues on specific browsers" |

---

## Common Interview Questions & Answers

### Q1: "A test fails in CI but passes locally. How do you debug?"

**A:** "Download the trace file from CI and replay it locally with `npx playwright show-trace`. Compare the trace to a local trace. Usually it's data/environment differences - staging API vs prod, test data expired, etc. The trace shows exactly where it diverged."

### Q2: "How do you identify flaky tests?"

**A:** "Run the test multiple times: `npx playwright test --repeat-each=5`. If it fails sometimes, download traces from failed runs. Compare traces to see what's different - usually timing (API too slow) or state (data not reset). Fix the actual cause, not the symptom."

### Q3: "Test times out but doesn't show what it's waiting for."

**A:** "Open the trace. It shows exactly what the test was doing when timeout occurred. Usually: waiting for element that never appeared, or network request that didn't complete. Then I either fix the wait condition or add proper network state handling."

### Q4: "How do you share debugging information with your team?"

**A:** "Upload the trace file to the bug ticket. Team can replay it locally and see exactly what happened. Way more useful than screenshots or logs. No guessing needed."

### Q5: "What's the most common debugging issue you've found?"

**A:** "Elements not being ready when expected. The fix: trust auto-waiting. Make sure your locators are correct - if the locator is right, auto-waiting will work. If tests still fail, the element really isn't appearing, which is a real bug (UI is broken)."

---

## What You Should Be Ready to Explain

✅ How to use Inspector for local debugging  
✅ How to use Traces for CI debugging  
✅ How videos help understand test failures  
✅ Difference between flaky tests and real bugs  
✅ How to diagnose timing vs state issues  
✅ Your debugging process at NielsenIQ (5-10 min per failure)  

---

## Weak Areas to Watch

❌ **Don't say:** "I add console.log statements to debug"  
→ **Reality:** Use traces and inspector - way more powerful

❌ **Don't claim:** "I can't debug CI failures"  
→ **Reality:** Download the trace and replay locally

❌ **Don't ignore:** Video/trace recording  
→ **Reality:** They're your best debugging tool

❌ **Don't spend:** Hours investigating without traces  
→ **Reality:** Traces show the answer in 5 minutes

---

## Key Takeaway for Interview

Debugging in Playwright is not "trying stuff." It's:

1. **See everything** → Video, trace, screenshot
2. **Replay failures** → Exact reproduction of the issue
3. **No guessing** → Data shows the problem
4. **Fast resolution** → 5-10 minutes vs 2+ hours

This is why Playwright tests are maintainable long-term. Your team can debug without you.

**At NielsenIQ:** Debug time dropped from 2+ hours (Selenium) to 5-10 minutes (Playwright). That's a 10x improvement in maintainability.

**Your competitive edge:** You can solve complex test failures because you have visibility into what actually happened. ✨

---

## Debugging Workflow (For Reference)

```typescript
// 1. CONFIGURE DEBUGGING (playwright.config.ts)
export default defineConfig({
  use: {
    video: 'on-failure',  // Record failed tests
    trace: 'on-first-retry',  // Trace on first failure
    screenshot: 'only-on-failure',  // Screenshot on failure
  },
});

// 2. RUN WITH DEBUGGING DURING DEVELOPMENT
// npx playwright test --debug
// Step through, use inspector

// 3. RUN TESTS
// npx playwright test

// 4. TEST FAILS IN CI
// Download trace from CI artifact

// 5. REPLAY LOCALLY
// npx playwright show-trace trace.zip
// See exactly what happened

// 6. IDENTIFY ROOT CAUSE
// - Element not visible? Check locator
// - Network error? Check API
// - Timing issue? Auto-wait should handle it
// - State issue? Add setup/teardown

// 7. FIX AND RE-RUN
// npx playwright test specific-test.spec.ts --debug
// Verify fix works

// Result: Test is stable and maintainable
```

---

**Remember:** With Playwright's debugging tools, every test failure has a clear answer. Find it. Fix it. Ship it. 🎯
