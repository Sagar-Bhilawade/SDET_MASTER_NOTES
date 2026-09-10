# Locators

## 1. What It Is

A locator is Playwright's lazy way to find an element when an action or assertion runs.
It is not a stored WebElement. It re-resolves against the current DOM, which helps when
modern apps re-render.

## 2. Why It Matters In Interviews

Locator strategy decides whether a test suite survives UI change. Interviewers check if
you know semantic locators, strict mode, filters, test ids, CSS/XPath fallback, and the
accessibility connection.

## 3. Complete Topic Breakdown

| Priority | Locator | Use for |
|---|---|---|
| 1 | `getByRole` | Buttons, links, headings, checkboxes, dialogs |
| 2 | `getByLabel` | Form fields with labels |
| 3 | `getByPlaceholder` | Inputs without good labels |
| 4 | `getByText` | Visible non-interactive text |
| 5 | `getByAltText` | Images/areas with alt text |
| 6 | `getByTitle` | Elements with title attribute |
| 7 | `getByTestId` | Stable automation contract |
| 8 | `locator('css/xpath')` | Fallback/special DOM cases |

### Core syntax

```typescript
page.getByRole('button', { name: 'Submit' });
page.getByLabel('Email');
page.getByPlaceholder('name@example.com');
page.getByText('Welcome');
page.getByAltText('Product image');
page.getByTitle('Close');
page.getByTestId('submit-button');
page.locator('button.primary');
page.locator('//button[text()="Submit"]');
```

### Filtering and chaining

```typescript
await page
  .getByRole('listitem')
  .filter({ hasText: 'Product 2' })
  .getByRole('button', { name: 'Add to cart' })
  .click();
```

```typescript
await page
  .getByRole('row', { name: /Sagar/ })
  .getByRole('button', { name: 'Edit' })
  .click();
```

Advanced filters and operators:

```typescript
await page.getByRole('listitem').filter({ hasText: 'Product 2' });
await page.getByRole('listitem').filter({ hasNotText: 'Out of stock' });
await page.getByRole('listitem').filter({
  has: page.getByRole('heading', { name: 'Product 2' })
});
await page.getByRole('listitem').filter({
  hasNot: page.getByText('Archived')
});
```

```typescript
const subscribeButton = page
  .getByRole('button')
  .and(page.getByTitle('Subscribe'));

const newButton = page.getByRole('button', { name: 'New' });
const securityDialog = page.getByText('Confirm security settings');
await expect(newButton.or(securityDialog).first()).toBeVisible();
```

List and visibility handling:

```typescript
await expect(page.getByRole('listitem')).toHaveCount(3);
await expect(page.getByRole('listitem')).toHaveText(['apple', 'banana', 'orange']);
await page.getByRole('listitem').nth(1).click();
await page.locator('button').filter({ visible: true }).click();
```

Use `first()`, `last()`, `nth()`, and visible filters carefully. They can hide locator
ambiguity if a better user-facing locator exists.

### Strict mode

If an action locator matches multiple elements, Playwright fails instead of guessing.
That is a quality signal: make the locator specific.

## 4. Manual Testing Angle

Manual testers describe elements like users do: "click Save", "open row for Sagar",
"fill Email". Good Playwright locators follow that same language.

## 5. Automation Angle

Use stable, user-meaningful locators:

- role/name for interactive elements,
- label for fields,
- text for messages,
- test id for stable app contracts,
- filters for repeated rows/cards,
- CSS/XPath only as fallback.

Trade-off: role/label locators need accessible UI. That is a good pressure because it
improves product quality.

## 6. Exact Surface To Remember

```typescript
await page.getByRole('button', { name: 'Save' }).click();
await page.getByLabel('Password').fill('secret');
await expect(page.getByText('Submitted')).toBeVisible();
```

```typescript
await page.getByText('Submitted', { exact: true }).click();
await page.getByRole('button', { name: /submit/i }).click();
```

```typescript
await page.locator('article', {
  has: page.getByRole('heading', { name: 'Billing' })
});
```

```typescript
await page
  .getByRole('listitem')
  .filter({ hasNotText: 'Out of stock' })
  .getByRole('button', { name: 'Buy' })
  .click();
```

Config custom test id:

```typescript
export default defineConfig({
  use: { testIdAttribute: 'data-pw' }
});
```

## 7. Real Project Example

> "In BASES_PLAYWRIGHT, I prefer role locators for user-facing actions and test ids where
> we need stable contracts. For repeated rows/cards, I first scope to the row/card and
> then locate the action inside it."

## 8. Common Mistakes / Weak Areas

- Using CSS classes as default.
- Using `nth()` to hide ambiguity.
- Using `getByText` for buttons where `getByRole` is better.
- Not passing accessible name with role locator.
- Treating locator as a stored element.
- Using long XPath because of Selenium habit.
- Ignoring strict mode instead of fixing the locator.

## 9. 30-Second Answer

Playwright locators are lazy queries that find elements at action/assertion time. My
priority is role with accessible name, label for fields, text for messages, test ids for
stable contracts, and CSS/XPath only as fallback. Strict mode catches ambiguous locators.

## 10. 2-Minute Answer

Locator strategy is a maintainability decision. I use locators close to user behavior:
`getByRole('button', { name: 'Save' })`, `getByLabel('Email')`, and
`getByText('Submitted')`. For stable app-specific controls, I use `getByTestId` as a
contract with developers. For repeated UI, I scope first and then act inside the scope
using `filter`, `hasText`, or chaining.

CSS and XPath are supported, but I avoid long DOM chains because they break when markup
changes. The trade-off is that semantic locators need accessible UI, but that is useful:
if a button has no role/name, it may be an accessibility issue too.

## 11. Interview Q&A

### Q: Preferred locator strategy?

Role/name first, label for fields, text for messages, test id for stable contracts,
CSS/XPath as fallback.

### Q: What is strict mode violation?

The locator matched multiple elements when Playwright needed one. Fix by scoping or
filtering, not blindly adding `first()`.

### Q: When use `getByTestId`?

When user-facing locators are dynamic, localized, or not specific enough, and the team
wants an explicit automation contract.

### Q: How do you handle repeated buttons?

Scope to the row/card first, then locate the button inside it.

### Q: When would you use `or()` locator?

When either of two states may appear, such as the expected button or a security dialog.
If both appear, it can still cause strict mode, so handle that deliberately.

### Q: Is `nth()` bad?

Not always, but it is fragile if order can change. Use it only when position/order is
itself the behavior or no better unique locator exists.

## 12. Quick Revision Points

- Locator is lazy.
- Role/name is usually best.
- Test id is stable but less user-facing.
- Strict mode is useful.
- `has`, `hasNot`, `hasText`, `hasNotText`, `and`, `or` are important follow-up syntax.
- `first/last/nth` are fallback tools, not locator strategy.
- CSS/XPath are fallback.
