# Playwright Locators — The Modern API

## What it is

Playwright's locator API — the built-in, recommended way to find elements, which layers user-facing locators (role, text, label) on top of raw CSS/XPath. This is your lead-stack locator knowledge: when you position Playwright-first, an interviewer expects you to know `getByRole` and why it beats a raw selector. Complements the general [CSS](02_CSS_Selectors_Complete.md) and [XPath](03_XPath_Complete.md) references.

---

## Why it matters

Playwright deliberately pushes you toward resilient, user-facing locators — it's one of the framework's biggest advantages over Selenium's raw-`By` model. Knowing the built-in locators and, crucially, *why they're recommended* (resilience, auto-waiting, strictness) is a direct signal of real Playwright experience. It also connects to the [Locators basics note](../../02_PLAYWRIGHT/01_BASICS/03_Locators.md) in your Playwright section.

---

## The recommended locators (in priority order)

```typescript
page.getByRole('button', { name: 'Submit' })   // 1. role + accessible name — BEST
page.getByLabel('Email')                        // 2. form control by its label
page.getByPlaceholder('Enter email')            // 3. by placeholder text
page.getByText('Welcome back')                  // 4. by visible text
page.getByAltText('Company logo')               // 5. images by alt
page.getByTitle('Close')                        // 6. by title attribute
page.getByTestId('login-button')                // 7. data-testid — the stable fallback
```
**Why role-first:** `getByRole` finds elements the way a user or screen reader does — by what the element *is* (a button, a checkbox) and its accessible name (its visible label). It's resilient to restyling and DOM refactors, and it doubles as an accessibility check (if `getByRole` can't find it, a screen reader can't either).

Below these, CSS/XPath escape hatches:
```typescript
page.locator('css=.some-class')      // or just page.locator('.some-class')
page.locator('xpath=//div[@id="x"]') // or page.locator('//div[@id="x"]')
```

---

## getByRole — the one to know deeply

```typescript
page.getByRole('button', { name: 'Sign in' })
page.getByRole('textbox', { name: 'Email' })
page.getByRole('checkbox', { name: 'Remember me' })
page.getByRole('link', { name: 'Forgot password' })
page.getByRole('heading', { name: 'Dashboard', level: 1 })
page.getByRole('button', { name: 'Delete', exact: true })  // exact match
```
Common roles: `button`, `link`, `textbox`, `checkbox`, `radio`, `heading`, `list`, `listitem`, `dialog`, `alert`, `tab`, `menuitem`. The `name` is the accessible name — usually the visible text or `aria-label`. `exact: true` forces exact (default is case-insensitive substring).

---

## Chaining and filtering (scoping to unique)

```typescript
// chain: find within a container
page.getByRole('listitem').filter({ hasText: 'Product A' })
    .getByRole('button', { name: 'Buy' });

// filter by text or by a child locator
page.getByRole('row').filter({ hasText: 'John' }).getByRole('cell').nth(2);
page.locator('.card').filter({ has: page.getByText('Sale') });

// scope with locator chaining
page.locator('.modal').getByRole('button', { name: 'Confirm' });
```
`filter({ hasText })` and `filter({ has: locator })` are the Playwright answer to the table-row problem — the clean, readable equivalent of the XPath `//tr[td[text()='John']]`. This is how you handle "the button in the row containing X" without XPath.

---

## Positional and nth

```typescript
page.getByRole('listitem').first();
page.getByRole('listitem').last();
page.getByRole('listitem').nth(2);    // 0-indexed! (unlike XPath's 1-indexed)
```
*Note the index base: Playwright's `.nth()` is **0-indexed**; XPath predicates are 1-indexed. A subtle cross-tool trap.*

Prefer `filter` over `nth` — positional is brittle (order changes break it). Use `nth` only when items are genuinely indistinguishable.

---

## Strictness — a key Playwright behavior

```typescript
await page.getByRole('button').click();  // THROWS if >1 button matches
```
Playwright locators are **strict by default**: if a locator resolves to multiple elements and you act on it, it *errors* rather than silently acting on the first. This is a feature — it catches ambiguous locators at test time instead of producing flaky "acted on the wrong element" bugs. To act on one of many, be explicit: `.first()`, `.nth()`, or narrow the locator. Explaining strictness is a strong signal of real Playwright understanding.

The wording to remember:

```text
Error: strict mode violation: locator('...') resolved to 2 elements
```

You may also see it attached to the action:

```text
locator.click: Error: strict mode violation: getByRole('button', { name: 'Edit' }) resolved to 2 elements
```

What it means:

> "Playwright found more than one matching element, and because actions require a unique
> target, it refused to guess."

How to fix:

```typescript
// Better: narrow to the row/card/dialog first
const row = page.getByRole('row').filter({ hasText: 'sagar@test.com' });
await row.getByRole('button', { name: 'Edit' }).click();

// Acceptable only when first item is truly the requirement
await page.getByRole('button', { name: 'Edit' }).first().click();

// Acceptable only when position is stable by requirement
await page.getByRole('button', { name: 'Edit' }).nth(1).click();
```

Interview answer:

> "The error is a strict mode violation. Playwright resolved the locator to multiple
> elements and refused to click because the target was ambiguous. My first fix is not
> `.first()` blindly; I scope to the correct parent row, card, modal, or section and then
> locate the child action."

---

## Locators are lazy (and why that matters)

```typescript
const btn = page.getByRole('button', { name: 'Save' });  // resolves NOTHING yet
await btn.click();  // NOW it finds the element, with auto-waiting
```
A `Locator` is a *description*, not a found element. It re-resolves every time you act, which is why:
- It **auto-waits** — retries finding + actionability until timeout.
- It **never goes stale** — unlike a Selenium `WebElement`, re-resolving each time means a re-rendered DOM isn't a problem.

This laziness is the architectural reason Playwright doesn't have `StaleElementReferenceException`. Contrast that with [Selenium's stale-element problem](05_Selenium_Locators.md) — a great compare-and-contrast answer.

---

## Text matching options

```typescript
page.getByText('Submit')                    // substring, case-insensitive
page.getByText('Submit', { exact: true })   // exact
page.getByText(/^Submit$/)                  // regex
page.locator('button', { hasText: 'Save' }) // tag + text
```

---

## Common interview Q&A

**Q: "Why does Playwright recommend getByRole over CSS?"**
It locates by what the element is to a user (resilient to restyling), doubles as an accessibility check, and reads clearly. CSS ties you to structure/classes that change.

**Q: "How do you handle 'the button in the row containing X'?"**
`getByRole('row').filter({ hasText: 'X' }).getByRole('button', {...})` — chaining and filter, no XPath needed.

**Q: "What happens if your locator matches two elements?"**
Strict mode throws on action with wording like `strict mode violation ... resolved to 2 elements`. That's a feature. I'd narrow it with filter or a parent container first; if items are truly identical by requirement, then `.first()` or `.nth()` is acceptable.

**Q: "Why no stale-element exceptions in Playwright?"**
Locators are lazy — they re-resolve on every action instead of holding a reference, so a re-rendered DOM isn't stale. Selenium's `WebElement` holds a reference, which goes stale.

**Q: "getByTestId — when?"**
When there's no good role/text hook, or for elements with dynamic/duplicated text. It's the stable, explicit contract — configure the attribute name via `testIdAttribute`.

---

## Weak areas to watch

- **Lead with `getByRole`** and explain *why* (resilience + accessibility), not just that it exists.
- **`filter({ hasText })` / chaining** is the Playwright table-row answer — know it instead of reaching for XPath.
- **Strictness is a feature** — locators throw on ambiguity; be ready to explain why that's good.
- **`.nth()` is 0-indexed** in Playwright vs XPath's 1-indexed — don't cross them up.
- **Laziness → no stale elements** — the architectural contrast with Selenium is a strong answer.
- **CSS/XPath are escape hatches** — know they exist (`page.locator('xpath=...')`) but reach for them last.

---

## Key takeaway

> **Playwright's locators push you toward resilient, user-facing selection — lead with `getByRole` and explain why (it locates by meaning, doubles as an accessibility check, and reads clearly).** The killer contrasts to have ready: `filter`/chaining replaces the XPath table-row hack, strict-mode throwing on ambiguity is a feature not a bug, and lazy re-resolution is *why* Playwright has no stale-element exception where Selenium does.
