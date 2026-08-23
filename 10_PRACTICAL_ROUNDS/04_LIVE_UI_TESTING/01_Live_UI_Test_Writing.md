# Live UI Test Writing — Approach + Examples

## What it is

The round where they show you a web page (or describe a feature) and say *"write automation tests for this."* Sometimes on a real screen with Playwright/Selenium set up, sometimes on a whiteboard. They're testing two things at once: your **test design** (what cases matter) and your **automation craft** (can you write clean, non-flaky test code).

---

## Why it matters

This is the round that most directly mirrors the actual job, so it carries weight. A pure developer can write code but doesn't instinctively enumerate boundary cases; a manual tester enumerates cases but can't automate them. The SDET does both — and this round is where you prove you're genuinely the intersection, not one wearing the other's badge. The candidates who stall are the ones who jump straight to code without designing the cases first.

---

## The approach (design before you type)

**Step 1 — Clarify the feature.** What does it do? What are the inputs and the success/failure states? Ask, don't assume.

**Step 2 — Enumerate test cases out loud, before any code.** This is the SDET signal. For any feature:
- **Happy path** — the main success flow.
- **Boundaries** — empty, max length, min/max values, zero.
- **Negative** — invalid input, wrong types, missing required fields.
- **Edge/state** — already-logged-in, expired session, duplicate submission, slow network.

**Step 3 — Prioritize.** "I'd automate the happy path and the top 2-3 negatives first; these lower-value edge cases I'd add later or leave manual." Naming what you'd *skip* is as senior as naming what you'd cover.

**Step 4 — Pick the level.** *"This validation logic I'd test at the API level — it's 200ms vs 10s through the UI and less fragile. I'd keep one thin UI test to confirm the wiring."* This trade-off is pure 20 LPA signal.

**Step 5 — Write it clean.** Good locators, proper waits, real assertions, isolated data.

---

## Worked example — a login form

**Feature:** email + password, submit → dashboard on success, inline error on failure.

**Cases I'd name (out loud, before coding):**
| Case | Type | Automate? |
|------|------|-----------|
| Valid credentials → dashboard | Happy | Yes, UI |
| Wrong password → error message | Negative | Yes, UI |
| Empty email / empty password | Boundary | Yes (API-level validation + 1 UI) |
| Invalid email format | Negative | API level |
| SQL-injection-ish input | Security | Note it, usually not UI-automated |
| Already logged in → redirect | State | Yes, medium priority |

**The happy path (Playwright/TS):**
```typescript
test('valid credentials land on dashboard', async ({ page }) => {
  await page.goto(`${baseURL}/login`);
  await page.getByLabel('Email').fill(user.email);
  await page.getByLabel('Password').fill(user.password);
  await page.getByRole('button', { name: 'Sign in' }).click();
  await expect(page).toHaveURL(/\/dashboard/);
  await expect(page.getByRole('heading', { name: 'Welcome' })).toBeVisible();
});
```

**The negative path:**
```typescript
test('wrong password shows an error', async ({ page }) => {
  await page.goto(`${baseURL}/login`);
  await page.getByLabel('Email').fill(user.email);
  await page.getByLabel('Password').fill('wrong-password');
  await page.getByRole('button', { name: 'Sign in' }).click();
  await expect(page.getByRole('alert')).toContainText('Invalid credentials');
  await expect(page).toHaveURL(/\/login/);   // did NOT navigate away
});
```

**Talk track while writing:**
- *"I'm using `getByRole` and `getByLabel` — role/label locators over CSS because they survive styling changes and match how a user finds things."* See [Locators](../../02_PLAYWRIGHT/01_BASICS/03_Locators.md).
- *"`expect().toHaveURL` and `.toBeVisible` are web-first — they auto-retry, so no sleeps."*
- *"The negative test also asserts we stayed on `/login` — a good negative test checks the bad thing didn't happen, not just that an error showed."*

---

## Selenium/Java version (if that's their stack)

```java
@Test
public void validLoginLandsOnDashboard() {
    driver.get(baseUrl + "/login");
    driver.findElement(By.id("email")).sendKeys(user.email);
    driver.findElement(By.id("password")).sendKeys(user.password);
    driver.findElement(By.id("submit")).click();
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    wait.until(ExpectedConditions.urlContains("/dashboard"));
    assertTrue(driver.findElement(By.cssSelector("h1")).getText().contains("Welcome"));
}
```
Talk track: explicit wait (never `Thread.sleep`), never mix with implicit, and if this grew you'd move the locators into a Page Object.

---

## Levelling up: mention the framework

Don't just write one test in isolation — show you think in frameworks:
> "In a real suite these locators live in a `LoginPage` page object, the credentials come from a data file or a fixture, and each test gets a fresh session so they run in parallel without colliding."

That one sentence turns "I can write a test" into "I design test suites."

---

## Common interview Q&A

**Q: "How many tests would you write for this?"**
Not "as many as possible." *"I'd automate the happy path and top negatives at UI level, push field validation to the API layer, and leave the rare edge cases manual until they earn automation. Coverage is about risk, not count."*

**Q: "Why role-based locators?"**
Resilient to styling/DOM changes, accessible-by-default, and they match user intent. CSS/XPath tied to structure break on every refactor.

**Q: "How do you keep these from being flaky?"**
Web-first assertions instead of sleeps, isolated test data per test, no shared state. See [Test Flakiness Mastery](../../04_TEST_ENGINEERING/01_Test_Flakiness_Mastery.md).

**Q: "This form talks to a payment API — how do you test it?"**
Mock the external API (`route`/`fulfill`), test *your* handling of its responses. Don't hit a real payment provider in a UI test. See [Network Mocking](../../02_PLAYWRIGHT/02_FRAMEWORK/08_Network_Mocking.md).

---

## Weak areas to watch

- **Don't code before you design.** Enumerate cases out loud first — that's the round's main signal. Jumping to code makes you look like a dev who tests, not an SDET.
- **A negative test must assert the bad thing didn't happen** — stayed on the page, no navigation, error shown. Just asserting the error text isn't enough.
- **Don't test everything through the UI.** Push validation to the API level and say why (speed, fragility). The trade-off is the senior marker.
- **Real assertions, not smoke.** Clicking through without verifying outcome is a click script, not a test.
- **Mention the framework** even when writing a single test — page objects, fixtures, data isolation. Shows you think at suite scale.
- **Good locators from the first line.** Reaching for a brittle CSS selector under pressure undoes the rest.

---

## Key takeaway

> **Design the cases out loud before you write a line, choose the right test level, then write it clean with resilient locators and web-first waits.** This round is the job in miniature — prove you're the genuine intersection of test design and automation craft by enumerating boundaries like a tester and coding isolation like an engineer, and by naming what you'd *push to the API layer or skip*, not just what you'd cover.
