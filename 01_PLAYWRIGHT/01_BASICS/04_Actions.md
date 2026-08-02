# Playwright - Actions: Interacting with Elements

## What Are Actions?

Actions are the things you DO with elements: click, type, select, drag, etc. When Playwright finds an element with a locator, you perform an action on it.

Basic flow:
```
1. Find element (locator)
2. Do something (action)
3. Assert result
```

Examples:
- Click a button → `page.getByRole('button', { name: 'Submit' }).click()`
- Type in a field → `page.getByTestId('email').fill('test@example.com')`
- Select from dropdown → `page.getByLabel('Country').selectOption('India')`

---

## Why It Matters (Interview Context)

Interviewers ask about actions because:
1. **Shows you understand Playwright's waiting model** (auto-waiting before actions)
2. **Demonstrates how to write stable tests** (which actions work best when)
3. **Explains flakiness prevention** (why you use `.fill()` not keyboard typing)
4. **Shows framework design thinking** (how you handle waits + actions)

Understanding actions deeply = understanding why your tests are stable.

---

## Your 30-Second Interview Answer

> "In Playwright, actions are methods you call on elements to interact with them. The key is that Playwright auto-waits for elements to be ready before performing actions. We use high-level actions like `click()`, `fill()`, `selectOption()` instead of low-level keyboard events because they're more stable. At NielsenIQ, this auto-waiting is why we reduced test flakiness from 15% (Selenium) to 2%."

---

## Your 2-Minute Deep Answer

> "Playwright has two types of actions: high-level and low-level.
>
> **HIGH-LEVEL ACTIONS (Preferred):**
>
> These wait for the element to be ready before acting:
>
> **1. CLICK**
> ```
> await page.getByRole('button', { name: 'Submit' }).click();
> ```
> What Playwright does automatically:
> - Waits for element to be in DOM
> - Waits for element to be visible
> - Waits for element to be not disabled
> - Scrolls to element if needed
> - Clicks at the center of the element
> - Waits for navigation (if button navigates)
>
> This is why it's stable. You don't write explicit waits.
>
> **2. FILL**
> ```
> await page.getByTestId('email').fill('test@example.com');
> ```
> What Playwright does:
> - Waits for input to be ready
> - Clears existing value (important!)
> - Types the new value
> - Validates it's been entered
>
> This is better than typing character-by-character because:
> - Faster
> - No delays between keystrokes
> - Works even if input validates in real-time
>
> **3. SELECT OPTION**
> ```
> await page.getByLabel('Country').selectOption('India');
> ```
> What Playwright does:
> - Waits for dropdown to be visible
> - Finds the option matching 'India'
> - Selects it
> - Triggers change events
>
> Way better than manually clicking dropdown + option.
>
> **4. CHECK/UNCHECK (Checkboxes)**
> ```
> await page.getByTestId('agree-terms').check();
> await page.getByTestId('newsletter').uncheck();
> ```
> Playwright:
> - Waits for checkbox to be ready
> - Checks/unchecks appropriately
> - Triggers events correctly
>
> **5. DRAG AND DROP**
> ```
> await page.getByTestId('item-1').dragTo(page.getByTestId('item-2'));
> ```
> Handles:
> - Mouse down on source
> - Mouse move to target
> - Mouse up at target
> - Triggers all drag/drop events
>
> **6. HOVER**
> ```
> await page.getByTestId('menu-item').hover();
> ```
> Waits for element and hovers over it. Triggers hover states and dropdowns.
>
> **7. FOCUS**
> ```
> await page.getByTestId('input-field').focus();
> ```
> Gives focus to an element. Useful before typing or for testing focus states.
>
> **LOW-LEVEL ACTIONS (Rarely Use):**
>
> These perform raw keyboard/mouse events without waiting:
> ```
> await page.keyboard.type('Hello')  // ❌ Don't do this
> await page.mouse.click(100, 200)   // ❌ Don't do this
> ```
> Problems:
> - No waiting for element readiness
> - Unreliable (timing issues)
> - Slower (character-by-character typing)
> - Only use if high-level actions don't work
>
> **Auto-Waiting Magic:**
>
> Before Playwright performs ANY high-level action, it auto-waits:
> 1. Element in DOM? Wait.
> 2. Element visible? Wait.
> 3. Element enabled (not disabled)? Wait.
> 4. Element clickable? Wait.
> 5. No animations/transitions running? Wait.
>
> This is what makes Playwright stable. In Selenium, you'd write:
> ```java
> WebDriverWait wait = new WebDriverWait(driver, 10);
> wait.until(ExpectedConditions.visibilityOfElement(element));
> element.click();
> ```
>
> In Playwright, it's just:
> ```typescript
> await page.getByRole('button', { name: 'Submit' }).click();
> ```
>
> One line. All the waiting is built-in.
>
> **At NielsenIQ:**
> We use high-level actions exclusively. Why?
> - 300+ tests need to be stable
> - Developers add `data-testid` to elements
> - Our actions auto-wait, so flakiness is eliminated
> - If a test fails, it's a real issue, not timing
> - Our test success rate went from 85% (Selenium) to 98% (Playwright)"

---

## High-Level vs Low-Level Actions

| Action | Type | Auto-Waits | Stable | When to Use |
|--------|------|-----------|--------|------------|
| `click()` | High-level | Yes | ⭐⭐⭐⭐⭐ | Always for clicking |
| `fill()` | High-level | Yes | ⭐⭐⭐⭐⭐ | Always for text input |
| `selectOption()` | High-level | Yes | ⭐⭐⭐⭐⭐ | Always for dropdowns |
| `check()` / `uncheck()` | High-level | Yes | ⭐⭐⭐⭐⭐ | Always for checkboxes |
| `dragTo()` | High-level | Yes | ⭐⭐⭐⭐ | Drag and drop |
| `hover()` | High-level | Yes | ⭐⭐⭐⭐ | Hover effects/menus |
| `focus()` | High-level | Yes | ⭐⭐⭐⭐ | Focus tests |
| `keyboard.type()` | Low-level | No | ⭐⭐ | Avoid (only edge cases) |
| `mouse.click()` | Low-level | No | ⭐⭐ | Avoid (only edge cases) |

---

## Your Real Project Example (NielsenIQ)

**Scenario: Login Test**

```typescript
// ✅ OUR APPROACH (Stable)
async function login(page: Page, email: string, password: string) {
  // Auto-waits, clears field, types, validates
  await page.getByTestId('email-input').fill(email);
  
  // Auto-waits, clears field, types
  await page.getByTestId('password-input').fill(password);
  
  // Auto-waits for button to be clickable, waits for navigation
  await page.getByRole('button', { name: 'Sign In' }).click();
  
  // Auto-waits for dashboard to load
  await page.getByTestId('dashboard-header').isVisible();
}

// Result: Test is stable. No explicit waits needed.
// Success rate: 98%+
```

**Compare to Selenium approach:**

```java
// ❌ SELENIUM (What we had to do)
WebDriverWait wait = new WebDriverWait(driver, 10);

// Explicit wait before filling
WebElement emailInput = wait.until(ExpectedConditions.presenceOfElementLocated(By.id("email")));
emailInput.clear();
emailInput.sendKeys("user@example.com");

// Explicit wait before filling password
WebElement passwordInput = wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("password")));
passwordInput.clear();
passwordInput.sendKeys("password123");

// Explicit wait and click
WebElement signInBtn = wait.until(ExpectedConditions.elementToBeClickable(By.xpath("//button[text()='Sign In']")));
signInBtn.click();

// Explicit wait for navigation
wait.until(ExpectedConditions.presenceOfElementLocated(By.id("dashboard")));

// Result: Verbose. Easy to get waits wrong. Test failures at 15%+
```

**The difference:**
- **Playwright:** 7 lines, all auto-waits, stable
- **Selenium:** 15+ lines, manual waits, flaky

This is why Playwright cut our flakiness from 15% to 2%.

---

## Common Interview Questions & Answers

### Q1: "Why `fill()` instead of `type()`?"

**A:** "`fill()` is Playwright's high-level action that clears the field and types the value. It auto-waits for the input to be ready. `type()` (or `keyboard.type()`) is low-level - it just sends keystrokes with no waiting. `fill()` is faster, more reliable, and triggers proper validation events. At NielsenIQ, we use `fill()` exclusively."

### Q2: "What does auto-waiting actually do?"

**A:** "Before performing an action like `click()`, Playwright waits for: (1) element in DOM, (2) element visible, (3) element not covered by other elements, (4) element not disabled, (5) no animations running. Only then does it click. This eliminates 80% of test flakiness because you're never clicking a not-ready element."

### Q3: "How do you handle a modal that takes 2 seconds to appear?"

**A:** "I don't write explicit waits. I just use the action. For example, `await page.getByTestId('modal').isVisible()` will auto-wait (default 30 seconds) for the modal to appear. If the modal takes 2 seconds, it waits 2 seconds and then checks. If it takes longer than the timeout, the test fails - which is correct behavior."

### Q4: "What if auto-waiting times out?"

**A:** "The test fails with a clear error like 'Timeout waiting for element to be visible'. This is good - it means something is actually wrong (element really isn't there/visible). In Selenium, you'd get confusing errors like 'StaleElementReference' that hide the real issue. Clear failures are better."

### Q5: "How do you interact with dynamic elements that appear/disappear?"

**A:** "Playwright's auto-waiting handles this. When you call an action on an element, it waits up to the timeout (default 30s) for that element to be ready. If it's dynamic and appears after 1-2 seconds, Playwright waits and then acts. No code changes needed."

---

## What You Should Be Ready to Explain

✅ Why high-level actions are better than low-level  
✅ How auto-waiting works (and why it eliminates flakiness)  
✅ Difference between `fill()` and `type()`  
✅ Why action order matters (click dropdown before selecting option)  
✅ How actions trigger events (change, submit, etc.)  
✅ Your stability improvements at NielsenIQ (15% → 2% flakiness)  

---

## Weak Areas to Watch

❌ **Don't say:** "`keyboard.type()` is fine for most cases"  
→ **Reality:** Use `fill()` - it's more stable and faster

❌ **Don't confuse:** Auto-waiting with explicit waits  
→ **Reality:** Auto-waiting is built-in; explicit waits are redundant

❌ **Don't use:** `mouse.click()` for testing  
→ **Reality:** It doesn't wait; use `click()` instead

❌ **Don't forget:** Actions trigger events (change, submit)  
→ **Reality:** Form validation depends on these events

---

## Key Takeaway for Interview

Actions in Playwright are not just "doing stuff." They're:

1. **Intelligent** → Auto-wait for readiness
2. **Stable** → No flakiness from timing
3. **Fast** → No unnecessary waits
4. **Event-aware** → Trigger proper browser events

This is why your NielsenIQ automation went from 85% stable (Selenium) to 98% stable (Playwright). Your action strategy shows mature test design.

**Your competitive edge:** You understand why Playwright is stable, not just how to use it. ✨

---

## Practical Code Pattern (For Reference)

```typescript
// ✅ GOOD: High-level actions with auto-waiting
await page.getByTestId('email').fill('test@example.com');
await page.getByTestId('password').fill('password123');
await page.getByRole('button', { name: 'Sign In' }).click();
await page.getByTestId('dashboard').isVisible();

// ⚠️  AVOID: Low-level actions with no waiting
await page.keyboard.type('test@example.com');  // Don't do this
await page.mouse.click(100, 200);  // Don't do this

// ✅ COMPLEX: Multi-step interaction
await page.getByLabel('Country').selectOption('India');
await page.getByTestId('state-dropdown').click();
await page.getByRole('option', { name: 'Maharashtra' }).click();
await page.getByTestId('city-search').fill('Mumbai');
await page.getByRole('option', { name: 'Mumbai' }).click();

// ✅ DRAG: Use high-level dragTo()
await page.getByTestId('task-1').dragTo(page.getByTestId('in-progress'));
```

---

**Remember:** Your action choices determine test stability. Always choose high-level, always trust auto-waiting. 🎯
