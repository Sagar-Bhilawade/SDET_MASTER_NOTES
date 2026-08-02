# Playwright - Assertions: Validating Test Outcomes

## What Are Assertions?

Assertions are checks you make to verify that something happened correctly. After you perform an action, you assert the result.

Basic flow:
```
1. Perform action (click, type, etc.)
2. Assert the result (check what happened)
3. Test passes or fails
```

Examples:
- Assert a button is visible → `await expect(button).toBeVisible();`
- Assert text appears on page → `await expect(page).toContainText('Welcome');`
- Assert URL changed → `await expect(page).toHaveURL('/dashboard');`

**Key difference from actions:** Actions DO something. Assertions CHECK that something happened.

---

## Why It Matters (Interview Context)

Interviewers ask about assertions because:
1. **Shows you understand test design** (what to check, not just what to do)
2. **Demonstrates test stability** (which assertions are flaky vs reliable)
3. **Explains test maintainability** (writing meaningful assertions, not brittle checks)
4. **Shows automation maturity** (testing behavior, not implementation)

Your assertion strategy determines if tests catch real bugs or just run without failing.

---

## Your 30-Second Interview Answer

> "In Playwright, assertions are checks that verify expected behavior. I use `expect()` for all assertions because it has auto-waiting and clear failure messages. Common assertions: `toBeVisible()`, `toContainText()`, `toHaveURL()`, `toHaveCount()`. At NielsenIQ, proper assertions reduced false negatives by 35% - tests now catch real bugs, not just run without errors."

---

## Your 2-Minute Deep Answer

> "Playwright's `expect()` API is powerful because it combines checking AND waiting.
>
> **Why Playwright's Expect is Different:**
>
> In Selenium, you'd write:
> ```java
> WebElement element = driver.findElement(By.id('message'));
> assert element.isDisplayed();  // No waiting - fails immediately if not visible
> assert element.getText().equals('Welcome');
> ```
>
> In Playwright:
> ```typescript
> await expect(page.getByTestId('message')).toBeVisible();  // Auto-waits up to 30s
> await expect(page.getByTestId('message')).toContainText('Welcome');
> ```
>
> See the difference? Playwright's `expect()` auto-waits. If the element isn't visible yet, it waits (default 30 seconds) before failing. This eliminates timing issues.
>
> **Common Assertions:**
>
> **1. VISIBILITY ASSERTIONS**
> ```
> await expect(page.getByTestId('success-message')).toBeVisible();
> await expect(page.getByTestId('error-modal')).toBeHidden();
> await expect(page.getByTestId('spinner')).not.toBeVisible();
> ```
> When to use:
> - After actions that show/hide elements
> - Waiting for modals, alerts, spinners
> - Verifying elements appear/disappear
>
> **2. TEXT ASSERTIONS**
> ```
> await expect(page).toContainText('Welcome, Sagar');
> await expect(page.getByTestId('dashboard-title')).toHaveText('Dashboard');
> await expect(page.locator('button')).toContainText('Submit');
> ```
> `toHaveText()` = exact match
> `toContainText()` = partial match
>
> When to use:
> - Verify success messages
> - Check form labels
> - Validate page content
>
> **3. URL ASSERTIONS**
> ```
> await expect(page).toHaveURL('/dashboard');
> await expect(page).toHaveURL(/\\/dashboard\\/.*/);  // Regex match
> ```
> When to use:
> - After navigation
> - Verify correct page loaded
> - Check URL parameters
>
> **4. COUNT ASSERTIONS**
> ```
> await expect(page.locator('li')).toHaveCount(5);  // Exactly 5 items
> ```
> When to use:
> - Verify list has correct number of items
> - Check table rows
>
> **5. ATTRIBUTE ASSERTIONS**
> ```
> await expect(page.getByTestId('input')).toHaveValue('john@example.com');
> await expect(page.getByTestId('checkbox')).toBeChecked();
> await expect(page.getByRole('button')).toBeDisabled();
> await expect(page.getByRole('button')).toBeEnabled();
> ```
> When to use:
> - Verify form values
> - Check checkbox/radio states
> - Verify disabled/enabled state
>
> **6. CLASS/ATTRIBUTE ASSERTIONS**
> ```
> await expect(page.getByTestId('item')).toHaveClass('selected');
> await expect(page.getByTestId('link')).toHaveAttribute('href', '/home');
> ```
> When to use:
> - Verify CSS classes applied
> - Check HTML attributes
> - Validate styling state
>
> **7. SCREENSHOT/VISUAL ASSERTIONS**
> ```
> await expect(page).toHaveScreenshot();
> ```
> When to use:
> - Visual regression testing
> - Verify layout didn't break
> - Compare against baseline
>
> **Best Practice: Assert Behavior, Not Implementation**
>
> ❌ Don't assert:
> ```
> await expect(page.locator('.modal-overlay')).toBeVisible();  // CSS class - fragile
> await expect(page.locator('div[data-id=\"123\"]')).toContainText('Item');  // Implementation detail
> ```
>
> ✅ Do assert:
> ```
> await expect(page.getByRole('dialog')).toBeVisible();  // User sees a dialog
> await expect(page.getByText('Welcome, Sagar')).toBeVisible();  // User sees welcome message
> ```
>
> **At NielsenIQ:**
>
> Our assertion strategy:
> 1. Assert visible outcomes (what user sees)
> 2. Use high-level assertions (toBeVisible, toContainText)
> 3. Avoid implementation details (CSS selectors in assertions)
> 4. Always use `expect()` with auto-waiting
> 5. One assertion per test step (clear failure messages)
>
> Result:
> - ✅ 300+ tests with clear pass/fail
> - ✅ Assertions auto-wait (no flakiness)
> - ✅ Test failures point to real bugs
> - ✅ False negatives reduced by 35%"

---

## Assertion Types Comparison

| Assertion | Syntax | Auto-Waits | When to Use |
|-----------|--------|-----------|------------|
| **Visibility** | `toBeVisible()` | Yes | After actions that show elements |
| **Hidden** | `toBeHidden()` | Yes | After actions that hide elements |
| **Text Match** | `toHaveText('exact')` | Yes | Verify exact text |
| **Text Contains** | `toContainText('partial')` | Yes | Verify partial text |
| **URL** | `toHaveURL('/path')` | Yes | After navigation |
| **Count** | `toHaveCount(5)` | Yes | Verify list/table length |
| **Value** | `toHaveValue('input value')` | Yes | Check form input values |
| **Checked** | `toBeChecked()` | Yes | Verify checkbox is checked |
| **Enabled** | `toBeEnabled()` | Yes | Verify button is clickable |
| **Disabled** | `toBeDisabled()` | Yes | Verify element is disabled |
| **Class** | `toHaveClass('active')` | Yes | Verify CSS class |
| **Attribute** | `toHaveAttribute('href', '/url')` | Yes | Check HTML attributes |
| **Screenshot** | `toHaveScreenshot()` | Yes | Visual regression testing |

---

## Your Real Project Example (NielsenIQ)

**Login Test with Proper Assertions:**

```typescript
async function testLogin(page: Page) {
  // 1. Action: Fill form
  await page.getByTestId('email-input').fill('user@example.com');
  await page.getByTestId('password-input').fill('password123');
  
  // 2. Action: Submit
  await page.getByRole('button', { name: 'Sign In' }).click();
  
  // 3. ASSERTIONS: Verify login succeeded
  
  // ✅ Assert user is logged in (URL changed)
  await expect(page).toHaveURL('/dashboard');
  
  // ✅ Assert dashboard loaded (main element visible)
  await expect(page.getByTestId('dashboard-header')).toBeVisible();
  
  // ✅ Assert user name displayed
  await expect(page.getByText('Welcome, Sagar')).toBeVisible();
  
  // ✅ Assert logout button present (only logged-in users see this)
  await expect(page.getByRole('button', { name: 'Logout' })).toBeEnabled();
}
```

**Why these assertions are good:**
- They check what the USER sees (not implementation)
- They auto-wait (no flakiness)
- They're specific (clear failure messages)
- They test behavior (user is logged in)

**Contrast with bad assertions:**

```typescript
// ❌ BAD: Asserts implementation details
await expect(page.locator('.user-profile-modal')).toBeVisible();  // CSS class fragile
await expect(page.locator('div.hidden')).toBeHidden();  // Fragile CSS

// ❌ BAD: No auto-waiting
console.assert(element.isDisplayed());  // Old Selenium style

// ❌ BAD: Too specific
await expect(page.getByText('Welcome, Sagar Bhilawade (SDET)')).toHaveText('exact string');  // Breaks if name format changes
```

---

## Common Interview Questions & Answers

### Q1: "What's the difference between `toHaveText()` and `toContainText()`?"

**A:** "`toHaveText()` checks for exact match. `toContainText()` checks if the text contains that string. For example, if the page says 'Welcome to Dashboard', `toHaveText('Welcome')` fails but `toContainText('Welcome')` passes. Use `toHaveText()` for exact values, `toContainText()` for flexible matching."

### Q2: "How does Playwright's `expect()` handle waits?"

**A:** "Playwright's `expect()` auto-waits up to 30 seconds (configurable). If the assertion would fail, it keeps retrying until the timeout. This eliminates timing issues. In Selenium, if an element isn't ready, the assertion fails immediately with no retries."

### Q3: "Should I assert every action?"

**A:** "Not every action needs an assertion. After every USER-VISIBLE action, yes. For example: click submit → assert success message. But don't assert internal state or implementation details. Assert behavior, not implementation."

### Q4: "What if an assertion fails? How do you debug it?"

**A:** "Playwright gives a clear failure message. For example: 'Expected element to be visible but it was not.' The trace/video shows exactly what happened. We don't need to guess - the failure message points to the issue."

### Q5: "How do you test dynamic content?"

**A:** "Use `toContainText()` instead of exact text matching. For example, if the page shows 'You have 5 items' but the count changes, use `toContainText('You have')` instead of matching the exact number."

---

## What You Should Be Ready to Explain

✅ Why `expect()` is better than manual assertions  
✅ How auto-waiting eliminates timing issues  
✅ Difference between toHaveText and toContainText  
✅ Common assertions and when to use them  
✅ Why you assert behavior, not implementation  
✅ Your assertion strategy at NielsenIQ (35% false negative reduction)  

---

## Weak Areas to Watch

❌ **Don't say:** "I assert everything, even internal state"  
→ **Reality:** Assert user-visible behavior only

❌ **Don't use:** Fragile CSS selectors in assertions  
→ **Reality:** Use semantic locators (getByRole, getByTestId)

❌ **Don't write:** Multiple assertions per step  
→ **Reality:** One clear assertion per step (better error messages)

❌ **Don't ignore:** Auto-waiting  
→ **Reality:** It's your biggest advantage over Selenium

---

## Key Takeaway for Interview

Assertions are not just "checks at the end." They're:

1. **Auto-waiting** → No timing issues
2. **Behavior-focused** → Test what users see
3. **Clear failures** → Easy debugging
4. **Maintainable** → Don't rely on implementation details

Your assertion strategy shows if you write tests that catch real bugs or just run without errors.

**At NielsenIQ:** Your assertions catch 35% more bugs than before. That's the power of a good assertion strategy.

**Your competitive edge:** You assert behavior, not implementation. That's mature testing. ✨

---

## Practical Code Pattern (For Reference)

```typescript
// ✅ GOOD: Clear, behavior-focused assertions
async function testCheckout(page: Page) {
  // Action
  await page.getByRole('button', { name: 'Proceed to Checkout' }).click();
  
  // Assertions (what user should see)
  await expect(page).toHaveURL(/.*checkout/);
  await expect(page.getByRole('heading', { name: 'Order Summary' })).toBeVisible();
  await expect(page.getByTestId('total-price')).toContainText('$99.99');
  await expect(page.getByRole('button', { name: 'Place Order' })).toBeEnabled();
}

// ✅ GOOD: Dynamic content
async function testSearch(page: Page) {
  await page.getByTestId('search-input').fill('laptop');
  await page.getByRole('button', { name: 'Search' }).click();
  
  // Auto-waits for results to load
  await expect(page.getByTestId('search-results')).toBeVisible();
  
  // Assert count (flexible for dynamic results)
  await expect(page.locator('[data-testid="result-item"]')).toHaveCount(
    await page.locator('[data-testid="result-item"]').count()
  );
}

// ❌ AVOID: Implementation details
await expect(page.locator('.hidden-implementation-detail')).toBeHidden();
await expect(page.locator('button.btn-primary.px-4.py-2')).toContainText('Submit');
```

---

**Remember:** Your assertions are what make tests USEFUL. Write them for behavior, not implementation. 🎯
