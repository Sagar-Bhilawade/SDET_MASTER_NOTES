# Playwright - Error Messages And Exceptions

## What it is

This note is the quick map from common Playwright error wording to root cause and fix.

Interviewers may ask:

> "If locator matches multiple elements, what error do you get?"
> "What happens when an element is hidden?"
> "What common Playwright failures have you debugged?"

The goal is not to memorize every stack trace. The goal is to hear the error and know what
category it belongs to.

## Why it matters

Knowing exact error types shows real framework experience. Anyone can say "test failed";
an SDET should say whether it failed because of locator ambiguity, actionability, timeout,
closed context, auth/session issue, assertion mismatch, or environment setup.

## 30-second answer

> "The common Playwright errors I watch for are strict mode violation when a locator matches
> multiple elements, timeout errors when an element or assertion does not become ready,
> actionability failures when the element is hidden, disabled, unstable, or covered, closed
> page/context errors, assertion mismatches, and environment issues like missing browser
> binaries. I debug by first classifying the error, then using trace, screenshot, locator
> inspection, and test data checks."

## 2-minute answer

> "I classify Playwright failures by category.
>
> If the error says `strict mode violation`, the locator matched multiple elements. I fix it
> by scoping to a parent row/card/dialog or using `filter({ hasText })`, not by blindly using
> `.first()`.
>
> If it is a timeout, I ask what was waiting: an action, a locator, navigation, or an
> assertion. Timeout means the expected condition never became true within the configured
> time.
>
> If it is actionability, Playwright found the element but could not safely act on it because
> it was hidden, disabled, moving, or covered by another element.
>
> If it says page/context/browser closed, something ended the browser session before the
> action completed.
>
> For assertion failures, I check whether the product is wrong, the expected value is wrong,
> or the assertion is too weak or too strict. Trace viewer is usually the fastest way to see
> the real page state."

## Common error map

| Error wording | Meaning | Usual fix |
|---|---|---|
| `strict mode violation ... resolved to 2 elements` | Locator matched multiple elements | Scope to parent, use `filter`, use unique role/name/testid |
| `TimeoutError: locator.click: Timeout ... exceeded` | Click never became possible | Check visibility, enabled state, overlay, wrong locator |
| `expect(locator).toBeVisible() with timeout` | Expected UI state did not appear | Check app state, data, API response, wrong assertion |
| `element is not visible` | Element exists but user cannot see it | Wait/assert correct state, open parent menu/modal, fix locator |
| `element is not enabled` | Element is disabled | Fill required fields, check validation/business state |
| `element is outside of the viewport` | Element is not interactable in current viewport | Scroll, fix layout/viewport, use normal locator action |
| `subtree intercepts pointer events` | Another element covers target | Close overlay, wait for animation, use correct visible target |
| `Target page, context or browser has been closed` | Test tried action after page/context closed | Check teardown, popup handling, async/await, premature close |
| `net::ERR_*` on `page.goto` | Navigation/network failure | Check URL, environment, VPN, server availability |
| `No tests found` | Runner did not find matching tests | Check test path, grep, file naming, config |
| `Executable doesn't exist ... install browsers` | Browser binaries missing | Run `npx playwright install` |
| `forbidOnly` / `test.only` failure | Focused test committed or running in CI | Remove `test.only` / `describe.only` |
| `toHaveText` mismatch | Text did not match expected value | Check exact vs partial match, whitespace, data, timing |
| `response status expected 200 received 401` | Unauthenticated API call | Check token, `request` vs `page.request`, expired session |
| `response status expected 200 received 403` | Authenticated but not permitted | Check role/permission, expected access |

## Strict mode violation

Example:

```text
locator.click: Error: strict mode violation:
getByRole('button', { name: 'Edit' }) resolved to 2 elements
```

Meaning:

> "Playwright found more than one matching element and refused to guess which one to click."

Good fix:

```typescript
const row = page.getByRole('row').filter({ hasText: 'sagar@test.com' });
await row.getByRole('button', { name: 'Edit' }).click();
```

Interview line:

> "Strict mode violation is not a bad Playwright behavior. It protects us from clicking the
> wrong element."

## Timeout errors

Example:

```text
TimeoutError: locator.click: Timeout 30000ms exceeded
```

Meaning:

> "Playwright waited for the action to become possible, but the condition never became true."

Debug checklist:

- Is locator correct?
- Is element visible?
- Is element enabled?
- Is it covered by overlay?
- Is test data creating the right state?
- Did navigation/API call fail before this step?

Interview line:

> "Timeout is a symptom, not the root cause. I check what condition Playwright was waiting
> for and why the page never reached that state."

## Actionability errors

Before actions like `click()` and `fill()`, Playwright checks actionability.

For click, important checks include:

- element is attached
- element is visible
- element is stable
- element receives events
- element is enabled

Example:

```text
locator.click: element is not visible
locator.click: element is not enabled
locator.click: subtree intercepts pointer events
```

Good answer:

> "These mean Playwright found the locator, but user-like interaction was not safe. I would
> not force click first. I would check why the element is hidden, disabled, moving, or covered."

## Assertion failures

Example:

```typescript
await expect(page.getByRole('alert')).toContainText('Saved');
```

Possible failure:

```text
Error: expect(locator).toContainText(expected) failed
```

Meaning:

> "The UI did not reach the expected state or my expected value/assertion is wrong."

Good debug:

- open trace
- check actual text
- check API response
- check if assertion is exact or partial
- check whether test data changed

## Page/context/browser closed

Example:

```text
Target page, context or browser has been closed
```

Meaning:

> "The test tried to interact after the page or context was already closed."

Common causes:

- missing `await`
- test teardown ran early
- popup/new page was closed
- context closed in fixture cleanup too soon
- unhandled async work continued after test ended

## API auth failures

401:

> "Not authenticated. Token missing, expired, invalid, or wrong request context."

403:

> "Authenticated but not authorized. User is logged in but does not have permission."

Common Playwright-specific confusion:

> "The isolated `request` fixture does not share browser session. If I need API call as the
> logged-in UI user, I use `page.request`. For pure API tests, I attach bearer token in the
> API request context."

## Setup/config errors

### Browser missing

```text
Executable doesn't exist ... Please run: npx playwright install
```

Fix:

```bash
npx playwright install
```

### No tests found

```text
Error: No tests found
```

Fix:

- check file path
- check `testMatch`
- check grep pattern
- check file naming like `.spec.ts`

### `test.only` blocked

```text
Error: item focused with test.only is not allowed due to forbidOnly
```

Fix:

> "Remove `test.only`. `forbidOnly` protects CI from accidentally running a partial suite."

## Real project example

> "In CI debugging, I first classify the failure. If it is strict mode, I check data volume
> and locator scoping. If it is timeout/actionability, I open trace and check whether the
> element was visible, enabled, stable, and not covered. If it is 401/403, I check whether I
> used isolated request context, page request, or a bearer token fixture. This avoids guessing
> and makes debugging much faster."

## Common interview Q&A

### Q1: "What error if locator matches multiple elements?"

**A:** "`strict mode violation ... resolved to 2 elements`. Playwright refuses to guess. I
fix it by narrowing the locator using parent scope, `filter({ hasText })`, or a unique test
id."

### Q2: "What do you do when click times out?"

**A:** "I check whether the locator is correct and whether actionability conditions are met:
visible, stable, enabled, and receiving events. Trace viewer usually shows if an overlay,
animation, wrong state, or data issue is the real cause."

### Q3: "Do you use force click?"

**A:** "Only as a last resort. If Playwright says it cannot click like a user, I first find
why. Force click can hide a real product or locator issue."

### Q4: "Why no stale element in Playwright?"

**A:** "Playwright locators are lazy and re-resolve on every action. Selenium WebElement
stores a reference that can go stale after DOM re-render, but Playwright locator describes
how to find the element again."

### Q5: "How do you debug 401 in API tests?"

**A:** "I check whether the call used isolated `request`, `page.request`, or a token-based
context. Isolated request has no browser session, so if the endpoint needs logged-in browser
auth, `page.request` is the correct choice."

## Weak areas to watch

- Do not say "some error comes" when exact wording is `strict mode violation`.
- Do not use `.first()` blindly to silence strict mode.
- Do not treat every timeout as a wait problem; check app state and data.
- Do not confuse 401 and 403.
- Do not force click before understanding actionability.

## Key takeaway

> "Playwright errors are useful signals. Strict mode means locator ambiguity, timeout means
> expected state never happened, actionability means user-like interaction was unsafe, and
> 401/403 means auth state or permission. Classify first, then fix."
