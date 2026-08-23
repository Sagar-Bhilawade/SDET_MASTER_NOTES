# XPath — Complete Reference

## What it is

Every XPath technique you need for automation: syntax, axes, functions, and the patterns interviewers ask you to write on the spot (the table-row lookup being the classic). XPath is the most powerful locator — it can do what CSS can't (match by text, traverse upward) — and also the most brittle, so it's the last-resort tool you must nonetheless know cold.

---

## Why it matters

"Write an XPath for this element" is a live interview task, and the table-row XPath specifically comes up again and again. XPath is also where the deepest DOM understanding shows — axes, `contains()`, `text()`, predicates. On the legacy Selenium stack this was daily work, so an interviewer probing XPath expects real fluency, not "I usually use CSS." Know it well enough to *build* one live.

---

## Absolute vs relative

```
/html/body/div[2]/form/input      ← ABSOLUTE — never use this
//input[@name='email']            ← RELATIVE — always use this
```
- **Absolute** (`/` from root) breaks if *anything* above the element changes. The canonical anti-pattern.
- **Relative** (`//` = "anywhere in the document") is resilient. Every good XPath starts with `//`.

---

## Basic syntax

```
//tag[@attribute='value']
```
| XPath | Matches |
|-------|---------|
| `//input` | All inputs anywhere |
| `//input[@name='email']` | Input with name=email |
| `//*[@id='submit']` | Any tag with id=submit |
| `//button[@class='primary']` | Button with exact class |
| `//div[@data-testid='cart']` | The test-id gold standard |

`*` is the wildcard tag: `//*[@id='x']` matches any element with that id.

---

## Text matching (XPath's superpower — CSS can't do this)

```
//button[text()='Submit']              exact text
//button[contains(text(),'Sub')]       partial text
//a[.='Login']                         . = normalized string value of the node
//label[contains(.,'Email')]           . searches descendant text too
//h1[normalize-space()='Welcome']      trims/collapses whitespace
```
**`text()` vs `.`:** `text()` matches the direct text node only; `.` (dot) matches the element's entire string value including descendants. For `<button>Click <span>here</span></button>`, `text()='Click here'` fails but `.='Click here'` works. Know this distinction — it's a favorite question.

**`normalize-space()`** trims leading/trailing whitespace and collapses internal runs — essential because HTML is full of stray whitespace. `//td[normalize-space()='John']` matches `  John  `.

---

## The functions worth knowing

| Function | Use | Example |
|----------|-----|---------|
| `contains(a, b)` | Partial match (text or attribute) | `//div[contains(@class,'alert')]` |
| `text()` | Direct text node | `//span[text()='OK']` |
| `normalize-space()` | Trim/collapse whitespace | `//td[normalize-space()='John']` |
| `starts-with(a, b)` | Prefix match | `//input[starts-with(@id,'user-')]` |
| `last()` | Last element | `//tr[last()]` |
| `position()` | Positional | `//tr[position()=2]` |
| `not(cond)` | Negation | `//input[not(@disabled)]` |
| `count(nodeset)` | Count | `count(//tr)` |

```
//div[contains(@class,'error')]              class contains (handles multi-class)
//input[starts-with(@id,'field-')]           dynamic id prefix
//button[contains(text(),'Add') and @enabled] combined conditions
//li[not(contains(@class,'hidden'))]         exclude hidden
```
*`contains(@class,'x')` is important: an element with `class="btn btn-primary active"` won't match `[@class='btn']` (exact) but will match `contains(@class,'btn')`. Use contains for multi-class elements.*

---

## Predicates (the `[...]` filters)

```
//input[@type='text']              attribute predicate
//tr[3]                             third row (1-indexed!)
//tr[last()]                        last row
//tr[position()<=3]                 first three rows
//button[@class='ok'][@enabled]     multiple predicates (AND)
//div[@id='x' or @id='y']           OR inside one predicate
```
**XPath is 1-indexed** (`[1]` is first), unlike CSS `nth-child` which is also 1-based but unlike array indexing. `//tr[1]` is the first row.

---

## Axes (XPath's other superpower — traverse ANY direction)

This is what CSS fundamentally cannot do: move **up** and **sideways** through the tree.

| Axis | Direction | Example |
|------|-----------|---------|
| `child::` | Down (default) | `//ul/child::li` |
| `parent::` or `..` | **Up one level** | `//span[text()='X']/parent::div` |
| `ancestor::` | **Up any levels** | `//td[.='John']/ancestor::table` |
| `following-sibling::` | Siblings after | `//label/following-sibling::input` |
| `preceding-sibling::` | Siblings before | `//input/preceding-sibling::label` |
| `following::` | Anything after in document | `//h2/following::p` |
| `preceding::` | Anything before | `//p/preceding::h2` |
| `descendant::` | Any depth down | `//form/descendant::input` |

```
//label[text()='Email']/following-sibling::input   the input next to a label
//td[text()='John']/parent::tr                      the row containing John
//span[@class='price']/ancestor::div[@class='card'] climb to the card
//button/..                                          the button's parent (shorthand)
```
**`following-sibling` and `parent`/`ancestor` are the money axes** — "find the input next to the label that says Email" and "find the row containing this cell" are extremely common real tasks that *require* axes. This is why XPath survives despite its brittleness.

---

## The classic interview XPaths (memorize these)

### The table-row lookup (THE one they ask)
*"Get the value in the 3rd column of the row whose first cell is 'John'."*
```
//table[@id='orders']//tr[td[text()='John']]/td[3]
```
Read it: "the tr that has a td with text John, then that row's 3rd td." The `tr[td[text()='John']]` predicate — "a row containing a matching cell" — is the key pattern. Be able to build and explain this live.

### The button in a specific card/row
```
//div[@class='card'][.//h3[text()='Product A']]//button[text()='Buy']
```
"The card containing the heading Product A, then the Buy button inside it."

### Input by its label
```
//label[text()='Password']/following-sibling::input
//input[@id=//label[text()='Password']/@for]        matching label's 'for' to input id
```

### Dynamic element with stable text
```
//button[contains(text(),'Delete')]
```

---

## contains for attributes AND the multi-condition patterns

```
//a[contains(@href,'/logout')]                       partial href
//div[contains(@class,'modal') and contains(@class,'open')]  both classes
//input[@name='q' and @type='search']                two attributes
//*[@data-testid='x' or @id='x']                     either hook
```

---

## Common interview Q&A

**Q: "Write an XPath for the 3rd cell of the row containing 'John'."**
`//tr[td[text()='John']]/td[3]`. Explain the predicate: a row that has a cell with that text.

**Q: "text() vs dot (.)?"**
`text()` = direct text node only; `.` = full string value including descendant text. For nested elements, `.` works where `text()` fails.

**Q: "How do you get a parent element?"**
`parent::` or `..`, or `ancestor::` for any level up. This is what CSS can't do — the reason XPath exists.

**Q: "Element next to a label?"**
`//label[text()='Email']/following-sibling::input`.

**Q: "Absolute vs relative XPath?"**
Absolute starts at root (`/html/...`) and is brittle; relative (`//`) matches anywhere and is resilient. Always relative.

**Q: "Handle a dynamic ID in XPath?"**
`contains(@id,'stable-part')` or `starts-with(@id,'prefix')`.

---

## Weak areas to watch

- **Always `//` (relative), never `/html/body/...`** — the absolute path is the anti-pattern they're checking you avoid.
- **Master the table-row XPath** — `//tr[td[text()='X']]/td[n]` — it's the single most-asked one.
- **`text()` vs `.`** — the nested-element distinction.
- **`contains(@class,...)` for multi-class elements** — exact `@class` match fails on `class="a b c"`.
- **Axes are the reason XPath exists** — `following-sibling`, `parent`, `ancestor`. Know them; CSS can't do them.
- **XPath is 1-indexed** — `[1]` is first, not `[0]`.
- **`normalize-space()`** for whitespace-laden text — HTML is messy.

---

## Key takeaway

> **XPath is the last-resort locator you must still know cold, because it does the two things CSS can't: match by visible text and traverse upward via axes.** Memorize the table-row pattern (`//tr[td[text()='X']]/td[n]`), the `text()`-vs-`.` distinction, `contains(@class,...)` for multi-class elements, and the `following-sibling`/`ancestor` axes — then always write it relative (`//`), never absolute.
