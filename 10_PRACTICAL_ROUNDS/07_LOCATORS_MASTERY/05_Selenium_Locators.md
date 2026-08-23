# Selenium Locators — The `By` Strategies

## What it is

Selenium's element-location model: the eight `By` strategies, how `findElement` works, and the stale-element problem that defines Selenium locating. This is your legacy-stack locator knowledge — a Java-shop interviewer will expect you to know the `By` methods and, critically, why stale elements happen and how you handle them. Complements the general [CSS](02_CSS_Selectors_Complete.md) and [XPath](03_XPath_Complete.md) references.

---

## Why it matters

Selenium locating is lower-level than Playwright — you get a raw `WebElement` reference and manage its lifecycle yourself, which is exactly why staleness and waits are constant concerns. Two years on the legacy stack means an interviewer expects fluency here, and the stale-element question is one of the most common Selenium interview questions there is. This connects to [WebDriver Architecture](../../03_SELENIUM_JAVA/01_SELENIUM_UI/01_WebDriver_Architecture.md) and [Locators & XPath](../../03_SELENIUM_JAVA/01_SELENIUM_UI/02_Locators_And_XPath.md).

---

## The 8 By strategies

```java
driver.findElement(By.id("submit"));
driver.findElement(By.name("email"));
driver.findElement(By.className("btn-primary"));      // single class only
driver.findElement(By.tagName("input"));
driver.findElement(By.linkText("Forgot password"));   // exact link text
driver.findElement(By.partialLinkText("Forgot"));     // partial link text
driver.findElement(By.cssSelector("input[name='email']"));
driver.findElement(By.xpath("//button[text()='Submit']"));
```

| Strategy | Use | Notes |
|----------|-----|-------|
| `By.id` | Best when ids are stable/meaningful | Fastest; beware auto-generated ids |
| `By.name` | Form fields | Common and stable |
| `By.className` | Single class | **One class only** — `By.className("a b")` fails; use CSS for multi-class |
| `By.tagName` | Broad selection | Rarely unique |
| `By.linkText` | `<a>` by exact text | Anchors only |
| `By.partialLinkText` | `<a>` by partial text | Anchors only |
| `By.cssSelector` | The workhorse | Prefer for most cases |
| `By.xpath` | Text match, upward traversal | Last resort, most powerful |

**Priority in Selenium:** `id` → `name` → `cssSelector` → `xpath`. Same principle as everywhere: stable, meaningful hooks first; structural/positional last.

**The `By.className` gotcha:** it accepts a *single* class name only. For an element with `class="btn btn-primary active"`, `By.className("btn active")` throws. Use `By.cssSelector(".btn.active")` for multiple classes. A frequent catch.

---

## findElement vs findElements

```java
WebElement one = driver.findElement(By.id("x"));      // first match; THROWS NoSuchElement if none
List<WebElement> all = driver.findElements(By.cssSelector(".item"));  // list; EMPTY list if none
```
- `findElement` returns the first match and **throws `NoSuchElementException`** if nothing matches.
- `findElements` returns a **list** (possibly empty — never throws). Use it to check existence without a try/catch: `if (!driver.findElements(By.id("x")).isEmpty())`.

*This difference is a common question — the "how do you check if an element is present?" answer is `findElements(...).isEmpty()`, not a try/catch around `findElement`.*

---

## The stale element problem (THE Selenium locator question)

```java
WebElement row = driver.findElement(By.id("row-1"));  // reference to a DOM node NOW
// ... page re-renders (AJAX update, navigation) ...
row.click();  // StaleElementReferenceException — the node it pointed to is gone
```
**Why it happens:** a `WebElement` is a *reference* to a specific DOM node captured at find time. If the DOM re-renders — even rebuilding the same-looking element — the old reference points at a node that no longer exists in the live DOM. Selenium throws `StaleElementReferenceException`.

**How you handle it:**
1. **Re-find the element** after the DOM changes (don't cache across re-renders).
2. **Use explicit waits** on a fresh locator: `wait.until(ExpectedConditions.elementToBeClickable(By.id("row-1")))` re-finds each poll.
3. **PageFactory `@FindBy`** uses lazy proxies that re-locate on access — helps, but doesn't fully immunize you.
4. Don't store `WebElement`s as long-lived fields when the page mutates.

**The senior contrast:** Playwright doesn't have this problem *because its locators are lazy* — they re-resolve on every action instead of holding a reference. "Selenium hands you a reference that can go stale; Playwright hands you a description that re-resolves." That compare-and-contrast is a strong answer and ties your two stacks together — see [Playwright Locators](04_Playwright_Locators.md).

---

## Relative Locators (Selenium 4)

Selenium 4 added locating by visual position:
```java
import static org.openqa.selenium.support.locators.RelativeLocator.with;

driver.findElement(with(By.tagName("input")).below(By.id("email")));
driver.findElement(with(By.tagName("button")).toRightOf(By.id("cancel")));
driver.findElement(with(By.tagName("input")).near(By.id("label")));
```
Methods: `above`, `below`, `toLeftOf`, `toRightOf`, `near`. Useful when the DOM has no good hook but the visual layout is stable. Mention it to show Selenium-4 currency — but note it's position-based, so it's brittle if the layout shifts.

---

## PageFactory (the `@FindBy` model)

```java
public class LoginPage {
    @FindBy(id = "email")        private WebElement email;
    @FindBy(css = "button.login") private WebElement loginBtn;

    public LoginPage(WebDriver driver) { PageFactory.initElements(driver, this); }
}
```
`@FindBy` with lazy proxies — the element is located when first used, not at construction. Trade-off vs plain `By`: cleaner page objects, but the proxy indirection can obscure staleness and makes some dynamic scenarios harder. See [Page Object Model (Java)](../../03_SELENIUM_JAVA/01_SELENIUM_UI/06_Page_Object_Model_Java.md).

---

## Common interview Q&A

**Q: "What are the locator strategies in Selenium?"**
The 8 `By` methods. Priority: id → name → css → xpath. Meaningful hooks first.

**Q: "findElement vs findElements?"**
`findElement` returns first / throws if none; `findElements` returns a list / empty if none. Use `findElements(...).isEmpty()` to check presence without exceptions.

**Q: "What is StaleElementReferenceException and how do you fix it?"**
A cached `WebElement` reference whose DOM node was re-rendered away. Fix: re-find the element, use explicit waits on a locator (re-finds each poll), don't cache across re-renders.

**Q: "Why doesn't Playwright have stale elements?"**
Lazy locators re-resolve on each action; Selenium holds a reference that can go stale. The architectural contrast.

**Q: "By.className with two classes?"**
Fails — single class only. Use `By.cssSelector(".a.b")`.

**Q: "Locate by visible text in Selenium?"**
XPath `text()`/`contains()`, or `By.linkText`/`partialLinkText` for anchors. No CSS text option.

---

## Weak areas to watch

- **The stale-element question is near-guaranteed** — nail the cause (reference to a re-rendered node) and the fixes (re-find, explicit wait, don't cache).
- **`By.className` is single-class only** — know the CSS fallback.
- **`findElements(...).isEmpty()` for presence checks** — not try/catch around `findElement`.
- **Priority: id → name → css → xpath** — same meaning-first principle.
- **Have the Playwright contrast ready** — lazy re-resolution vs stale references links your two stacks.
- **Selenium 4 relative locators** show currency, but flag them as position-brittle.

---

## Key takeaway

> **Selenium hands you a `WebElement` reference you manage yourself — which is exactly why staleness is the defining locator question: a re-rendered DOM invalidates your cached reference.** Know the 8 `By` strategies (id→name→css→xpath), the `findElement`/`findElements` throw-vs-empty difference, and above all the stale-element cause and fixes — then land the Playwright contrast (lazy re-resolution has no stale problem) to show you understand *why* across both stacks.
