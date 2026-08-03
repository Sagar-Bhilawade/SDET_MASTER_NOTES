# Selenium - Locators & XPath

## What This Covers

Finding elements in Selenium — the eight locator strategies, when to use each, and XPath depth (which interviewers love to probe).

---

## Why It Matters (Interview Context)

Locators are the most-asked Selenium topic, and XPath specifically is where interviewers dig. Expect:

> "Write an XPath for a button inside a table row where the first cell says 'Sagar'."

That's a real question and it tests whether you can navigate relationships, not just write `//button`.

The senior layer is knowing that **locator strategy determines maintenance cost** — the same argument as in Playwright, and you can now make it from both sides.

---

## Your 30-Second Interview Answer

> "Selenium has eight strategies — id, name, className, tagName, linkText, partialLinkText, cssSelector, and xpath. My priority is id first when it's stable and unique, then CSS selectors, and XPath only when I need something CSS can't express — mainly text matching or traversing to a parent. CSS is faster than XPath because browsers have native optimized engines for it. The bigger point is that locator choice drives maintenance cost: anything tied to styling or DOM position breaks on every redesign."

---

## Your 2-Minute Deep Answer

> "The eight strategies, and how I actually prioritize:
>
> **1. `By.id`** — fastest and most stable, *if* the ID is genuinely unique and not auto-generated. Frameworks like older versions of GWT or some React setups generate IDs that change between builds — those are worse than useless.
>
> **2. `By.cssSelector`** — my default for everything else.
>
> ```java
> By.cssSelector("input[name='email']")
> By.cssSelector("#login-form .submit-btn")
> By.cssSelector("[data-testid='submit']")
> ```
>
> Faster than XPath because browsers implement `querySelector` natively and it's heavily optimized.
>
> **3. `By.xpath`** — when CSS can't do it. Two things CSS genuinely can't:
> - **Match on text content**
> - **Traverse upward to a parent or ancestor**
>
> **4-8.** `name`, `className`, `tagName`, `linkText`, `partialLinkText` — occasionally convenient, rarely my first choice. `linkText` is brittle because copy changes.
>
> ---
>
> **XPath — the part interviewers probe**
>
> **Absolute vs relative:**
> ```
> /html/body/div[2]/form/input[1]      ❌ absolute — breaks if anyone adds a wrapper div
> //input[@name='email']                ✅ relative
> ```
> Absolute XPath is always wrong in practice.
>
> **Text matching:**
> ```
> //button[text()='Submit']                        exact
> //button[contains(text(),'Sub')]                 partial
> //button[normalize-space()='Submit']             handles whitespace
> ```
>
> `normalize-space()` is the one worth knowing — it strips leading/trailing whitespace and collapses internal runs. HTML frequently has newlines and indentation inside elements, so `text()='Submit'` fails where `normalize-space()='Submit'` works. That's a very common real-world gotcha.
>
> **Attribute matching:**
> ```
> //input[@type='text' and @name='email']
> //div[contains(@class,'error')]
> //a[starts-with(@href,'/admin')]
> ```
>
> **Axes — the reason XPath still earns its place:**
> ```
> //label[text()='Email']/following-sibling::input
> //span[text()='Total']/parent::div
> //td[text()='Sagar']/ancestor::tr
> //td[text()='Sagar']/ancestor::tr//button[text()='Delete']
> ```
>
> That last one is the classic table question: find the row whose cell says 'Sagar', then find the delete button within that row. CSS cannot do this — there's no parent selector — and it's exactly why XPath survives.
>
> **Indexing:**
> ```
> (//input[@type='text'])[2]      second match — note the parentheses
> //table//tr[2]/td[3]
> ```
> The parentheses matter. `//input[2]` means "an input that is the second child of its parent", which is not the same as "the second input on the page".
>
> ---
>
> **`findElement` vs `findElements`**
>
> ```java
> driver.findElement(By.id("x"));    // throws NoSuchElementException if absent
> driver.findElements(By.id("x"));   // returns empty list — never throws
> ```
>
> That difference is useful: to assert something is *absent*, use `findElements` and check the list is empty. Using `findElement` in a try/catch for that is the version I'd flag in review — it works but it's slow, because it waits out the implicit timeout before throwing.
>
> ---
>
> **Chaining and scoping:**
> ```java
> WebElement form = driver.findElement(By.id("login-form"));
> WebElement email = form.findElement(By.name("email"));   // scoped within form
> ```
> Scoping is good practice — it avoids matching a similarly-named element elsewhere on the page. But it also means holding two element references, which doubles the stale-element surface.
>
> ---
>
> **The maintenance argument:**
>
> The same principle as Playwright: a CSS class is a styling implementation detail, and binding a test to it is a bet the design won't change. In Selenium there's no built-in role-based locator, so the practical equivalent is pushing for `data-testid` attributes and locating on those:
>
> ```java
> By.cssSelector("[data-testid='submit-order']")
> ```
>
> That was our approach — get test IDs treated as a contract with the dev team rather than QA retrofitting selectors after every redesign."

---

## Locator Priority

| Priority | Strategy | Use when |
|:---:|----------|----------|
| 1 | `By.id` | Stable, unique, not auto-generated |
| 2 | `By.cssSelector` on `data-testid` | The maintainable default |
| 3 | `By.cssSelector` | Structural, when testid unavailable |
| 4 | `By.xpath` | Text matching or upward traversal |
| — | Absolute XPath | Never |

---

## XPath Cheat Sheet

| Need | Expression |
|------|-----------|
| Exact text | `//button[text()='Submit']` |
| Text, whitespace-safe | `//button[normalize-space()='Submit']` |
| Partial text | `//button[contains(text(),'Sub')]` |
| Attribute contains | `//div[contains(@class,'error')]` |
| Multiple conditions | `//input[@type='text' and @name='email']` |
| Next sibling | `//label[text()='Email']/following-sibling::input` |
| Parent | `//span[text()='Total']/parent::div` |
| Ancestor row | `//td[text()='Sagar']/ancestor::tr` |
| Nth match | `(//input[@type='text'])[2]` |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "On the legacy application, a lot of the UI was table-heavy — grids of records with row-level actions. That's where XPath axes were genuinely necessary rather than a fallback. Finding the row whose identifier cell matches, then locating the action button within that row, isn't expressible in CSS because there's no parent selector.
>
> For everything else I defaulted to CSS, because it's faster — browsers implement `querySelector` natively and it's heavily optimized — and it's more readable in review.
>
> The maintenance lesson transferred directly to how I work in Playwright now. Locators tied to styling break on every redesign, so we pushed for `data-testid` attributes and located on those. Playwright gives you role-based locators natively, which is better, but the underlying principle is identical: don't couple tests to things that are designed to change.
>
> The gotcha I'd mention is `normalize-space()`. HTML routinely has newlines and indentation inside elements, so exact `text()` matching fails in ways that look inexplicable. That one cost me time before I understood it."

---

## Common Interview Questions & Answers

### Q1: "CSS or XPath — which do you prefer and why?"

**A:** "CSS by default, because browsers implement `querySelector` natively so it's faster, and it's more readable. XPath when I need something CSS can't express — matching on text content, or traversing upward to a parent or ancestor. Those two capabilities are the only reason XPath still earns its place."

### Q2: "Write an XPath to find the delete button in the row where the name is 'Sagar'."

**A:** `//td[normalize-space()='Sagar']/ancestor::tr//button[normalize-space()='Delete']`

"Find the cell with that text, walk up to its row, then find the delete button within that row. I'd use `normalize-space()` rather than `text()` because table cells often carry surrounding whitespace."

### Q3: "Difference between `findElement` and `findElements`?"

**A:** "`findElement` throws `NoSuchElementException` if nothing matches; `findElements` returns an empty list and never throws. That makes `findElements` the right tool for asserting absence — check the list is empty. Using `findElement` in a try/catch for that works but is slow, because it waits out the implicit timeout first."

### Q4: "Why avoid absolute XPath?"

**A:** "It encodes the entire DOM path, so adding a single wrapper div anywhere above your element breaks it. It's also unreadable — nobody can tell what `/html/body/div[2]/form/input[1]` targets without opening the page. There's no situation where relative XPath isn't better."

### Q5: "How do you make locators maintainable?"

**A:** "Stop coupling to things designed to change. CSS classes exist to be restyled, so a test bound to one will break by design. The practical answer in Selenium is pushing for `data-testid` attributes as a contract with the dev team, and encapsulating every locator in page objects so a change is one edit rather than fifty."

---

## Weak Areas to Watch

❌ **Don't use absolute XPath**
→ Breaks on any DOM insertion.

❌ **Don't forget `normalize-space()`**
→ Whitespace in HTML breaks exact text matching constantly.

❌ **Don't write `//input[2]` for "second input"**
→ That's positional within parent. Use `(//input)[2]`.

❌ **Don't use `findElement` in try/catch to check absence**
→ `findElements` and check empty. Faster and clearer.

❌ **Don't claim XPath is always slower**
→ It's slower, but the gap is small; readability and stability matter more.

---

## Key Takeaway for Interview

CSS by default, XPath for what CSS can't do — text and upward traversal.

The answer that shows practical experience:

> **"`normalize-space()` rather than `text()`. HTML routinely carries newlines and indentation inside elements, so exact text matching fails in ways that look inexplicable until you know."**

And the table question, which you should be able to write instantly:

> **`//td[normalize-space()='X']/ancestor::tr//button[normalize-space()='Delete']`**

---

## Checklist

✅ Can name all eight locator strategies
✅ CSS as default, XPath for text and axes
✅ Absolute XPath never used
✅ `normalize-space()` for text matching
✅ Can write the table-row XPath from memory
✅ `findElements` used for absence checks
✅ All locators encapsulated in page objects

---

**Remember:** XPath survives because CSS has no parent selector. That's the whole story. 🎯
