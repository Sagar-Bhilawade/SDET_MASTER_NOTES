# Actions

## 1. What It Is

Actions are Playwright operations that interact with the page: click, fill, check, press,
select, upload, drag, hover, and keyboard/mouse operations.

Actions are not raw DOM commands. Before many actions, Playwright checks actionability so
it does not click an element that is hidden, moving, disabled, or covered.

## 2. Why It Matters In Interviews

Interviewers ask actions to check if you understand real user interaction, auto-waiting,
and when low-level actions are risky.

## 3. Complete Topic Breakdown

| Action | Syntax | Use |
|---|---|---|
| Click | `locator.click()` | Buttons, links, menu items |
| Double-click | `locator.dblclick()` | Grid/table interactions |
| Right-click | `locator.click({ button: 'right' })` | Context menus |
| Fill | `locator.fill('text')` | Clear and set input value |
| Type/press sequentially | `locator.pressSequentially('text')` | Keyboard-event-heavy fields |
| Press key | `locator.press('Enter')` | Keyboard shortcuts/forms |
| Check/uncheck | `locator.check()`, `locator.uncheck()` | Checkbox/radio |
| Select option | `locator.selectOption('IN')` | Select dropdowns |
| Hover | `locator.hover()` | Menus/tooltips |
| Upload | `locator.setInputFiles(path)` | File inputs |
| Drag | `locator.dragTo(target)` | Drag-drop UI |
| Focus/blur | `locator.focus()`, `locator.blur()` | Focus behavior |

## 4. Manual Testing Angle

Manual testers know the difference between a user-visible action and an implementation
shortcut. Automation should model the user where behavior matters, especially for forms,
keyboard access, validation, and menus.

## 5. Automation Angle

Prefer high-level locator actions. Use low-level keyboard/mouse only when the behavior
depends on those events.

Examples:

- `fill()` for normal inputs because it clears and sets value.
- `press()` for Enter/search behavior.
- `hover()` for tooltip/menu behavior.
- `setInputFiles()` for upload controls.
- `force: true` only with clear reason, not as a normal fix.

## 6. Exact Surface To Remember

```typescript
await page.getByRole('button', { name: 'Submit' }).click();
await page.getByLabel('Email').fill('sagar@example.com');
await page.getByLabel('Country').selectOption('IN');
await page.getByRole('checkbox', { name: 'Accept' }).check();
await page.getByRole('button', { name: 'Save' }).press('Enter');
await page.getByLabel('Upload').setInputFiles('fixtures/report.pdf');
```

Event-first pattern:

```typescript
const downloadPromise = page.waitForEvent('download');
await page.getByRole('button', { name: 'Export' }).click();
const download = await downloadPromise;
```

Force click:

```typescript
await locator.click({ force: true });
```

Use `force` only when you intentionally bypass actionability, because it can hide real UI
bugs.

## 7. Real Project Example

> "In BASES_PLAYWRIGHT, I use high-level actions like `click`, `fill`, `selectOption`,
> and upload/download event patterns. I avoid `force` unless the test deliberately needs
> to bypass actionability, because otherwise it can hide overlay or layout defects."

## 8. Common Mistakes / Weak Areas

- Using `type`/keyboard events for every input when `fill` is enough.
- Clicking before waiting for the correct event.
- Using `force: true` to silence real problems.
- Clicking CSS targets instead of user-facing controls.
- Not asserting the result after action.
- Forgetting upload/download flows need event handling.

## 9. 30-Second Answer

Actions are how Playwright interacts with elements. I prefer high-level locator actions
like `click`, `fill`, `check`, `selectOption`, and `setInputFiles`. Playwright performs
actionability checks before actions, so I avoid hard waits and use event-first patterns
for downloads, popups, and network waits.

## 10. 2-Minute Answer

In Playwright, actions behave like user interactions and include built-in waiting. Before
a click, Playwright checks whether the element is visible, stable, enabled, and able to
receive events. That prevents many Selenium-style timing failures. I use high-level
actions for normal user behavior and only use lower-level keyboard/mouse APIs when the
feature specifically depends on keyboard or mouse behavior.

The trade-off is that forcing actions or bypassing actionability can create green tests
that ignore real UI defects. If a normal click fails, I first inspect whether the element
is covered, disabled, hidden, or not the correct target.

## 11. Interview Q&A

### Q: Why `fill()` instead of typing?

`fill()` clears and sets value directly for normal inputs. Use keyboard-style input only
when the app behavior depends on key events.

### Q: What does `force: true` do?

It bypasses some actionability checks. Use it rarely because it can hide real UI issues.

### Q: How do you handle download after click?

Start `page.waitForEvent('download')` before the click, then await the promise after the
click.

### Q: Why can click time out?

The element may be hidden, disabled, moving, covered by overlay, outside viewport, or the
locator may point to the wrong element.

## 12. Quick Revision Points

- Actions do; assertions check.
- Prefer locator actions.
- `fill` for normal text fields.
- Event waits must start before the action.
- Avoid `force` as a normal fix.
