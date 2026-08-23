# Code Review Exercise — Worked Examples

## What it is

The round where they hand you a piece of test code — often deliberately flawed — and ask *"review this"* or *"what's wrong with it?"* It's a senior-round staple because it reveals what you actually notice: a junior spots syntax, a senior spots the flaky sleep, the shared state, the assertion that can't fail. This note gives you a **review checklist** and several worked examples with the bugs called out.

---

## Why it matters

Code review is the highest-leverage thing a senior SDET does — it's how quality scales beyond the tests you personally write. So interviewers use it as a direct proxy for seniority. The signal they're reading: do you review for *correctness and maintainability*, or just style? Do you explain *why* something is a problem and what it costs? Can you give feedback that teaches rather than just flags?

You already have the [Mentoring & Code Review](../../06_LEADERSHIP_AND_ARCHITECTURE/03_Mentoring_And_Code_Review.md) concept note — this is the applied, "here's actual code, find the bugs" version.

---

## The review checklist (run it in order)

1. **Correctness** — does it actually test what it claims? Can the assertion ever fail?
2. **Flakiness** — hard sleeps, race conditions, shared state, order dependence.
3. **Isolation** — does it create its own data and clean up? Does it depend on another test?
4. **Waits** — sleeps vs proper waits; implicit/explicit mixing (Selenium).
5. **Assertions** — is there one? Is it specific? Web-first (Playwright) or manual?
6. **Readability/maintenance** — magic values, duplication, unclear names, hardcoded selectors.
7. **Async correctness** (TS) — every promise awaited?

Then: **lead with the highest-impact issue**, explain the cost, suggest the fix. Don't nitpick style before you've named the flaky sleep.

---

## Worked example 1 — Playwright/TS

```typescript
test('user can log in', async ({ page }) => {
  await page.goto('http://localhost:3000/login');
  await page.fill('#user', 'admin');
  await page.fill('#pass', 'admin123');
  page.click('#submit');
  await page.waitForTimeout(3000);
  expect(await page.url()).toContain('/dashboard');
});
```

**What to say, in priority order:**
1. **Missing `await` on `page.click`** (line 5) — the click returns a floating promise; the test races ahead. This is *the* classic TS automation bug. Highest impact.
2. **`waitForTimeout(3000)`** — a hard sleep. Flaky (3s might not be enough on a slow CI run) *and* slow (wasted 3s when the page is ready in 200ms). Replace with a web-first assertion: `await expect(page).toHaveURL(/\/dashboard/)` — it auto-retries.
3. **`expect(await page.url())`** — not web-first, so no retry; it reads the URL once. `expect(page).toHaveURL()` polls.
4. **Hardcoded URL and credentials** — should come from config/env, not be inlined. Maintainability, lower priority.

**Fixed:**
```typescript
test('user can log in', async ({ page }) => {
  await page.goto(`${baseURL}/login`);
  await page.fill('#user', creds.user);
  await page.fill('#pass', creds.pass);
  await page.click('#submit');
  await expect(page).toHaveURL(/\/dashboard/);
});
```

---

## Worked example 2 — Selenium/Java

```java
@Test
public void testCheckout() {
    driver.findElement(By.id("addToCart")).click();
    Thread.sleep(2000);
    driver.findElement(By.id("checkout")).click();
    driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("confirm")));
    driver.findElement(By.id("confirm")).click();
}
```

**What to say:**
1. **`Thread.sleep(2000)`** — hard sleep, flaky and slow. Replace with an explicit wait on the checkout button's clickability.
2. **Mixing implicit and explicit waits** (lines 5-6) — the single worst Selenium anti-pattern. When both are set, wait times compound unpredictably and you get erratic timeouts. Pick explicit-only. See [Waits & Synchronization](../../03_SELENIUM_JAVA/01_SELENIUM_UI/03_Waits_And_Synchronization.md).
3. **No assertion.** The test clicks through a flow but never verifies the outcome. It "passes" as long as nothing throws — it can't actually *fail* on a wrong result. A test with no assertion is the most dangerous kind: green and worthless.
4. **Setting implicit wait mid-test** — config like that belongs in setup, not scattered in the test body.

**The one to lead with: the missing assertion.** It's the difference between a test and a click script.

---

## Worked example 3 — the subtle one (shared state)

```java
public class OrderTests {
    static String orderId;   // shared across tests

    @Test(priority = 1)
    public void createOrder() {
        orderId = api.createOrder();
        assertNotNull(orderId);
    }

    @Test(priority = 2)
    public void cancelOrder() {
        api.cancel(orderId);   // depends on createOrder having run
        assertEquals(api.status(orderId), "CANCELLED");
    }
}
```

**What to say:**
1. **Order dependence via shared static state** — `cancelOrder` only works if `createOrder` ran first and set the static field. Break the order (parallel execution, a filter, running one test alone) and it NPEs. This is invisible until you parallelize, then it's a nightmare.
2. **`static` field across parallel threads** — a data race waiting to happen. See [Parallel Execution & ThreadLocal](../../03_SELENIUM_JAVA/02_TESTNG/03_Parallel_Execution_And_ThreadLocal.md).
3. **Fix:** each test creates its own order (via a fixture/setup), so they're independent and parallel-safe. The cost — slightly more setup per test — is worth it for isolation.

*This is the senior example: nothing here throws a syntax error, the tests pass today, and it's a latent disaster. Spotting it is the whole point.*

---

## How to deliver the feedback

The *manner* is assessed too. Model it as you would a real PR review:

- **Lead with impact, not nitpicks.** Flaky sleep before variable naming.
- **Explain the cost, not just the rule.** Not "don't use sleeps" but "this sleep makes it flaky on slow CI and wastes 3 seconds every run."
- **Suggest the fix**, don't just criticize.
- **Separate must-fix from nice-to-have.** "The missing assertion is a blocker; the hardcoded URL is a follow-up."
- **Be kind about it.** *"One thing that'll bite later..."* Real review teaches; interviewers watch whether you'd be someone a junior can learn from.

---

## Common interview Q&A

**Q: "What's the most important issue here?"**
Always the one that makes the test *lie*: no assertion, or a race that fails silently. Correctness before style, every time.

**Q: "Would you approve this PR?"**
"Not yet — the missing assertion is a blocker. With that added and the sleep replaced, yes. The hardcoded config I'd leave as a follow-up comment, not a block."

**Q: "How do you review without being the bottleneck?"**
Automate the mechanical stuff (linters, formatters catch style) so review focuses on logic and design. That frees you to review for what humans are actually needed for.

---

## Weak areas to watch

- **Don't start with style.** Leading with "I'd rename this variable" while missing the flaky sleep is a junior tell.
- **Don't just list problems** — prioritize them and explain the cost of each. A ranked review is a senior review.
- **The most dangerous bug is the test that can't fail.** Missing/weak assertions hide broken features. Train yourself to check "can this actually go red?" first.
- **Latent bugs beat visible ones.** Shared state and order dependence pass today and explode under parallelism — spotting those is what the round rewards.
- **Deliver it like a mentor**, not a critic. Tone is part of the assessment.

---

## Key takeaway

> **Review for correctness first, flakiness second, style last — and always name the cost, not just the rule.** The bug that wins the round is the one that makes the test *lie*: a missing assertion or a silent race that passes today and fails under parallelism. Spotting those, and giving feedback that teaches, is the direct seniority signal this round exists to measure.
