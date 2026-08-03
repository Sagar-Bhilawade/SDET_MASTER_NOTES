# Selenium - Page Object Model & PageFactory in Java

## What This Covers

POM in the Selenium/Java world, including `PageFactory` and `@FindBy` — which don't exist in Playwright and get asked about specifically.

---

## Why It Matters (Interview Context)

"Explain your Page Object Model" is guaranteed in any Selenium interview. The differentiators are:

1. **Do you know `PageFactory`** and its trade-offs, or just the basic pattern?
2. **Do you store `By` locators or `WebElement` fields** — and can you explain why that choice affects staleness?
3. **Where do you draw the boundary** between page object and test?

That second one connects straight back to the architecture and exception notes, and it's where a coherent answer really shows.

---

## Your 30-Second Interview Answer

> "Each page gets a class holding its locators and the actions a user can perform on it. Tests call those methods and never touch WebDriver directly, so a UI change is one edit rather than fifty. I store `By` locators rather than `WebElement` fields and resolve at the point of use — that's the main defence against stale element references, because you never hold a reference across a DOM change. `PageFactory` with `@FindBy` is the alternative; it's more concise but it caches proxies and hides the resolution, so I prefer explicit locators."

---

## Your 2-Minute Deep Answer

> "**The basic pattern:**
>
> ```java
> public class LoginPage {
>     private final WebDriver driver;
>     private final WebDriverWait wait;
>
>     // Locators, not elements
>     private final By emailInput = By.cssSelector("[data-testid='email']");
>     private final By passwordInput = By.cssSelector("[data-testid='password']");
>     private final By loginButton = By.cssSelector("[data-testid='login-btn']");
>     private final By errorMessage = By.cssSelector("[data-testid='error']");
>
>     public LoginPage(WebDriver driver) {
>         this.driver = driver;
>         this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
>     }
>
>     public DashboardPage login(String email, String password) {
>         wait.until(ExpectedConditions.visibilityOfElementLocated(emailInput))
>             .sendKeys(email);
>         driver.findElement(passwordInput).sendKeys(password);
>         driver.findElement(loginButton).click();
>         return new DashboardPage(driver);       // return the next page
>     }
>
>     public String getErrorMessage() {
>         return wait.until(ExpectedConditions.visibilityOfElementLocated(errorMessage))
>                    .getText();
>     }
> }
> ```
>
> **Three deliberate choices there:**
>
> **1. `By` locators, not `WebElement` fields.** This is the important one. If you store `WebElement` in a field, you're holding a server-side reference across whatever happens next — a re-render, a table redraw, a navigation — and it goes stale. Storing the `By` and resolving at use means you never hold a reference across a DOM change. That single decision removes most staleness from a suite.
>
> **2. Returning the next page object.** `login()` returns `DashboardPage`, so the test reads as a flow and the compiler enforces that you end up where you expected. If login can fail, you'd have separate methods — `loginExpectingSuccess()` returning DashboardPage and `loginExpectingFailure()` returning LoginPage.
>
> **3. Page objects expose actions, not elements.** No `getEmailInput()`. If a test can reach the element, encapsulation is gone and the change no longer propagates from one place.
>
> ---
>
> **`PageFactory` and `@FindBy` — the Selenium-specific alternative**
>
> ```java
> public class LoginPage {
>     @FindBy(css = "[data-testid='email']")
>     private WebElement emailInput;
>
>     @FindBy(css = "[data-testid='login-btn']")
>     private WebElement loginButton;
>
>     public LoginPage(WebDriver driver) {
>         PageFactory.initElements(driver, this);
>     }
> }
> ```
>
> `initElements` injects **lazy proxies** — the field isn't resolved until you use it, and it re-resolves on each access. So it's not as stale-prone as a naively cached `WebElement`, which is the common misconception.
>
> **Where I still prefer explicit `By`:**
>
> - The waiting is hidden. With `@FindBy` you can't easily wait on a specific condition before interacting — you end up wrapping in explicit waits anyway, which loses the conciseness benefit.
> - `@FindBy` proxies still go stale if you hold a reference across a re-render mid-method.
> - It's implicit magic. Explicit `By` is more code but the behaviour is obvious to whoever reads it next.
>
> `PageFactory` also supports `@FindBys` (AND) and `@FindAll` (OR) for combining locators, and `@CacheLookup` to genuinely cache — which you should almost never use, because it reintroduces staleness deliberately.
>
> Worth knowing: `PageFactory` is somewhat de-emphasized in modern Selenium practice. Being able to say that, and why, is a better answer than either using it uncritically or not knowing it.
>
> ---
>
> **BasePage — shared behaviour**
>
> ```java
> public abstract class BasePage {
>     protected final WebDriver driver;
>     protected final WebDriverWait wait;
>
>     protected BasePage(WebDriver driver) {
>         this.driver = driver;
>         this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
>     }
>
>     protected WebElement waitAndFind(By locator) {
>         return wait.until(ExpectedConditions.visibilityOfElementLocated(locator));
>     }
>
>     protected void click(By locator) {
>         wait.until(ExpectedConditions.elementToBeClickable(locator)).click();
>     }
>
>     protected void type(By locator, String text) {
>         WebElement el = waitAndFind(locator);
>         el.clear();
>         el.sendKeys(text);
>     }
> }
> ```
>
> This centralizes the wait-then-act pattern so page objects stay readable. It's also where you'd add retry-on-stale logic once, rather than scattering it.
>
> **The risk with BasePage** is that it becomes a dumping ground. If it accumulates fifty utility methods, it's a god object. I'd keep it to interaction primitives and put domain helpers elsewhere.
>
> ---
>
> **Where the boundary sits**
>
> Page objects own: locators, interactions, navigation, and returning the next page.
>
> Page objects do **not** own: assertions.
>
> That's a boundary worth defending in an interview. A page object with `assertLoginSucceeded()` inside it means the page object now knows what the test cares about, and it can't be reused by a test that expects failure. Page objects describe *what the page can do*; tests decide *what should be true*.
>
> Some teams put assertions in page objects deliberately. I'd say it's defensible for very simple suites and doesn't scale — and being able to argue that position is better than just following a rule."

---

## `By` Locators vs `PageFactory`

| | Explicit `By` | `@FindBy` + PageFactory |
|---|---|---|
| Verbosity | More | Less |
| Staleness | Resolve at use — safest | Lazy proxies, re-resolve |
| Explicit waiting | Natural | Awkward; wrap anyway |
| Readability | Obvious | Implicit magic |
| Modern practice | Preferred | De-emphasized |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Our page objects stored `By` locators and resolved at the point of use, with a `BasePage` providing wait-then-act primitives — click, type, waitAndFind. That kept individual page objects readable and put the waiting logic in one place.
>
> The `By`-over-`WebElement` decision was driven directly by staleness. The application had dynamic grids that redrew on sort and filter, so anything holding an element reference across that died. Resolving fresh at use meant we simply never held references across DOM changes, which removed most of the problem structurally rather than patching it.
>
> I'd evaluated `PageFactory` and preferred explicit locators. The proxies are lazy so it's not as stale-prone as people assume, but the waiting becomes awkward — you end up wrapping `@FindBy` fields in explicit waits anyway, which loses the conciseness that was the point. And explicit `By` is more obvious to whoever reads it next.
>
> The boundary I held was that page objects don't contain assertions. A page object that asserts login succeeded can't be reused by a test expecting failure — the page object now knows what the test cares about. Page objects describe what the page can do; tests decide what should be true.
>
> That principle carried directly into how I build page objects in Playwright. The pattern is the same; only the locator API changed."

---

## Common Interview Questions & Answers

### Q1: "Explain your Page Object Model."

**A:** "A class per page holding locators and the actions a user can perform. Tests call those methods and never touch WebDriver directly. I store `By` locators rather than `WebElement` fields and resolve at the point of use, which is the main structural defence against stale references. Methods return the next page object so flows read naturally and the compiler catches wrong transitions."

### Q2: "Why `By` instead of `WebElement` fields?"

**A:** "Staleness. A `WebElement` field holds a server-side reference across whatever happens next — a re-render, a table redraw — and dies. Storing the `By` and resolving at use means you never hold a reference across a DOM change. On an application with dynamic grids that removed most of our stale element failures without any per-case patching."

### Q3: "What's `PageFactory` and do you use it?"

**A:** "`@FindBy` annotations with `PageFactory.initElements` injecting lazy proxies that re-resolve on access. I know it and prefer explicit `By`. The proxies aren't as stale-prone as people assume, but explicit waiting becomes awkward — you wrap them anyway, losing the conciseness. It's also somewhat de-emphasized in modern Selenium practice."

### Q4: "Should assertions go in page objects?"

**A:** "No. A page object that asserts login succeeded can't serve a test expecting failure — it's absorbed the test's intent. Page objects describe what the page can do; tests decide what should be true. Some teams do put assertions in them and it's defensible for simple suites, but it doesn't scale."

### Q5: "How do page objects handle navigation between pages?"

**A:** "Action methods return the next page object — `login()` returns `DashboardPage`. That makes flows read naturally and gives you compile-time checking that you ended up where you expected. Where an action can branch, I'd have separate methods for the expected outcomes rather than returning something ambiguous."

---

## Weak Areas to Watch

❌ **Don't store `WebElement` in fields**
→ It's the main avoidable source of staleness.

❌ **Don't expose elements via getters**
→ Encapsulation gone; changes stop propagating.

❌ **Don't put assertions in page objects**
→ Couples the page object to one test's intent.

❌ **Don't use `@CacheLookup`**
→ Deliberately reintroduces staleness.

❌ **Don't let BasePage become a god object**
→ Interaction primitives only.

---

## Key Takeaway for Interview

The `By`-versus-`WebElement` decision is where a coherent answer shows, because it connects POM to architecture and to exceptions.

The line that ties it together:

> **"Store `By` locators, not `WebElement` fields. A WebElement field holds a server-side reference across the next DOM change and dies. Resolving at the point of use means you never hold one — that removes most staleness structurally rather than patching it case by case."**

And the boundary worth defending:

> **"Page objects describe what the page can do. Tests decide what should be true. A page object that asserts success can't serve a test expecting failure."**

---

## Checklist

✅ One class per page
✅ `By` locators stored, resolved at use
✅ No `WebElement` fields
✅ No element getters exposed
✅ Action methods return the next page object
✅ No assertions inside page objects
✅ `BasePage` limited to interaction primitives
✅ Can explain PageFactory trade-offs both ways

---

**Remember:** Everything you hold across a DOM change is a stale element waiting to happen. 🎯
