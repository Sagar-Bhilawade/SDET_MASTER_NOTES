# CSS Selectors — Complete Reference

## What it is

Every CSS selector method you need for automation, with examples. CSS is the preferred locator for ~90% of cases — faster and more readable than XPath — so fluency here is expected. When an interviewer says "write a CSS selector for this," this is what you draw on. What CSS *can't* do (select by text, traverse upward) is where [XPath](03_XPath_Complete.md) takes over.

---

## Why it matters

CSS selectors are the default in both Playwright and Selenium, and they're what you'll write most. An interviewer asking you to build one on the spot is checking real hands-on experience — you can't fake fluent combinator and attribute-selector usage. Knowing the full toolkit (not just `#id` and `.class`) lets you write a precise, resilient selector instead of falling back to brittle XPath.

---

## Basic selectors

| Selector | Matches | Example |
|----------|---------|---------|
| `tag` | All elements of that type | `button` |
| `#id` | Element with that id | `#submit` |
| `.class` | Elements with that class | `.btn-primary` |
| `*` | Any element | `*` |
| `tag.class` | Tag with class | `input.form-control` |
| `.a.b` | Both classes | `.btn.active` (has both) |

```css
#login-form            /* id="login-form" */
.error-message         /* class="error-message" */
button.primary         /* <button class="primary"> */
```

---

## Attribute selectors (the powerful, resilient ones)

| Selector | Matches |
|----------|---------|
| `[attr]` | Has the attribute | `[disabled]` |
| `[attr='value']` | Exact value | `[type='submit']` |
| `[attr^='value']` | **Starts with** | `[id^='user-']` |
| `[attr$='value']` | **Ends with** | `[id$='-input']` |
| `[attr*='value']` | **Contains** | `[class*='btn']` |
| `[attr~='value']` | Contains word (space-separated) | `[class~='active']` |
| `[attr\|='value']` | Value or value-prefixed | `[lang\|='en']` |

```css
[data-testid='login-button']    /* the gold standard */
input[name='email']
[id^='dynamic-']                /* handles auto-generated ids with a stable prefix */
[class*='alert']                /* matches alert-success, alert-error, etc. */
[aria-label='Close']
```
**The `^` `$` `*` operators are the answer to dynamic IDs** — when an app generates `user-12345-input`, you match on the stable `[id^='user-']` or `[id$='-input']` part. This is a common "how do you handle dynamic elements?" answer.

---

## Combinators (relationships between elements)

| Combinator | Meaning | Example |
|------------|---------|---------|
| `A B` | **Descendant** (any depth) | `form input` |
| `A > B` | **Direct child** (one level) | `ul > li` |
| `A + B` | **Adjacent sibling** (immediately after) | `label + input` |
| `A ~ B` | **General sibling** (any after) | `h2 ~ p` |

```css
.form-group input           /* any input inside .form-group */
.menu > li                  /* direct li children only */
label[for='email'] + input  /* the input right after that label */
```
*The combinators are how you scope: `.modal button.confirm` targets the confirm button *inside the modal*, avoiding the identical button elsewhere on the page. Scoping with combinators is often how you make a non-unique selector unique.*

---

## Pseudo-classes (state and position)

| Pseudo-class | Matches |
|--------------|---------|
| `:first-child` / `:last-child` | First/last child of its parent |
| `:nth-child(n)` | The nth child (1-indexed) |
| `:nth-child(2n)` / `:nth-child(odd)` | Even/odd children |
| `:nth-of-type(n)` | nth of that tag type |
| `:not(selector)` | Elements NOT matching |
| `:checked` / `:disabled` / `:enabled` | Form state |
| `:focus` / `:hover` | Interaction state |
| `:empty` | No children |

```css
tr:nth-child(2)             /* second row */
li:first-child
input:not([disabled])       /* only enabled inputs */
input:checked               /* checked checkboxes/radios */
tr:nth-child(even)          /* alternating rows */
```

### The `:nth-child` vs `:nth-of-type` gotcha
```html
<div><p>A</p><span>B</span><p>C</p></div>
```
- `p:nth-child(2)` → **matches nothing** (the 2nd child is the `<span>`, not a `<p>`).
- `p:nth-of-type(2)` → matches `<p>C</p>` (the 2nd `<p>`).

`nth-child` counts *all* siblings; `nth-of-type` counts only that tag. A classic interview catch — know the difference.

**But:** positional selectors are brittle (order changes break them). Use them only when there's no better hook, and prefer `nth-of-type` when you do.

---

## What CSS canNOT do (know these limits)

1. **Select by visible text.** There is no `:contains('text')` in standard CSS (it was dropped). This is the #1 reason to switch to XPath (or a framework text locator).
2. **Traverse upward** to a parent/ancestor. CSS only goes down and sideways, never up. `:has()` (newer) partially addresses this — see below — but classic CSS can't.
3. **Match on text content** at all.

### The `:has()` pseudo-class (modern, changes the game)
```css
tr:has(td:nth-child(1)) /* rows that contain a first cell */
div:has(> button.active) /* a div that has a direct active-button child */
```
`:has()` is the "parent selector" CSS never had — it selects an element *based on its descendants*. Supported in modern browsers and Playwright. Mentioning it shows current knowledge, but note older environments/Selenium may not support it, so XPath is still the fallback for upward traversal.

---

## Building a robust selector — worked example

Target: the "Delete" button in a specific user's table row, where the row has the user's email.

- **CSS struggles** here because it can't match the row *by its text* (the email). This is exactly where you'd say: *"CSS can't select the row by its email text, so I'd use XPath here"* — and pivot to [XPath](03_XPath_Complete.md).
- **If there's a test id:** `[data-testid='delete-user-42']` — clean, done. The lesson: a good `data-testid` removes the need for clever selectors entirely.

---

## Common interview Q&A

**Q: "Write a CSS selector for an input with name 'email' inside the login form."**
`#login-form input[name='email']` (or `form.login input[name='email']`).

**Q: "How do you handle a dynamically-generated ID?"**
Match the stable part: `[id^='user-']` (starts-with), `[id$='-input']` (ends-with), or `[id*='user']` (contains).

**Q: "Difference between `nth-child` and `nth-of-type`?"**
`nth-child` counts all siblings regardless of tag; `nth-of-type` counts only that tag. The `<p>/<span>/<p>` example.

**Q: "Can CSS select by text?"**
No — standard CSS has no text selector. That's when I use XPath's `text()`/`contains()` or a framework text locator.

**Q: "`>` vs space?"**
`>` is a direct child (one level); space is any descendant (any depth).

---

## Weak areas to watch

- **Know the attribute operators cold** (`^` `$` `*`) — they're the dynamic-ID answer.
- **`nth-child` vs `nth-of-type`** — the classic gotcha; know which counts what.
- **CSS can't do text or upward traversal** — state the limit and pivot to XPath, don't force a bad CSS selector.
- **Scope with combinators** to make non-unique selectors unique (`.modal .confirm`).
- **`:has()` exists but isn't universal** — great to mention, but keep XPath as the upward-traversal fallback.
- **Avoid positional selectors** where a stable attribute exists.

---

## Key takeaway

> **CSS handles ~90% of locating — master the attribute operators (`^`/`$`/`*` for dynamic IDs), the combinators (for scoping), and the `nth-child`-vs-`nth-of-type` distinction.** Its hard limits — no text matching, no upward traversal — are precisely where you pivot to XPath, and saying "CSS can't do that here, so I'd use XPath" is itself the right answer, not a failure.
