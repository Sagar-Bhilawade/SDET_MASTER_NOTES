# Assertions

## 1. What It Is

Assertions verify expected behavior. In Playwright, `expect` has web-first assertions
that auto-retry for UI state, plus normal generic assertions for values, objects, arrays,
and API response data.

Actions make something happen. Assertions prove the expected result happened.

## 2. Why It Matters In Interviews

Assertions decide whether tests catch real bugs or only perform steps. Interviewers often
probe exact matcher differences, auto-retry behavior, soft assertions, API assertions,
and weak assertion traps.

## 3. Complete Topic Breakdown

### Auto-retrying locator assertions

| Assertion | Checks |
|---|---|
| `toBeVisible()` | Element is visible |
| `toBeHidden()` | Element is hidden or not visible |
| `toBeAttached()` | Element exists in DOM |
| `toBeChecked()` | Checkbox/radio is checked |
| `toBeDisabled()` | Element is disabled |
| `toBeEnabled()` | Element is enabled |
| `toBeEditable()` | Element can be edited |
| `toBeEmpty()` | Element/input has no content |
| `toBeFocused()` | Element has focus |
| `toBeInViewport()` | Element intersects viewport |
| `toHaveText()` | Exact/full text match |
| `toContainText()` | Partial text match |
| `toHaveValue()` | Input value |
| `toHaveValues()` | Selected values in multi-select |
| `toHaveCount()` | Locator resolves to exact count |
| `toHaveAttribute()` | Attribute value |
| `toHaveClass()` | Class list/string |
| `toContainClass()` | Contains class without exact full match |
| `toHaveCSS()` | Computed CSS property |
| `toHaveId()` | Element id |
| `toHaveJSProperty()` | JS property |
| `toHaveRole()` | ARIA role |
| `toHaveAccessibleName()` | Accessible name |
| `toHaveAccessibleDescription()` | Accessible description |
| `toHaveScreenshot()` | Element screenshot baseline |
| `toMatchAriaSnapshot()` | ARIA tree snapshot |

### Page and response assertions

| Assertion | Checks |
|---|---|
| `expect(page).toHaveURL()` | Current URL |
| `expect(page).toHaveTitle()` | Page title |
| `expect(page).toHaveScreenshot()` | Page screenshot baseline |
| `expect(page).toMatchAriaSnapshot()` | Page ARIA snapshot |
| `expect(response).toBeOK()` | API response status is OK |

### Generic non-retrying assertions

Use these for values already captured, not for waiting on UI:

- `toBe`, `toEqual`, `toStrictEqual`, `toMatchObject`
- `toContain`, `toContainEqual`, `toHaveLength`, `toHaveProperty`, `toMatch`
- `toBeTruthy`, `toBeFalsy`, `toBeDefined`, `toBeUndefined`, `toBeNull`, `toBeNaN`
- `toBeGreaterThan`, `toBeGreaterThanOrEqual`, `toBeLessThan`, `toBeLessThanOrEqual`
- `toBeCloseTo`, `toBeInstanceOf`, `toThrow`

### Assertion control

- `.not`
- `expect.soft`
- custom assertion message
- `expect.configure`
- `expect.poll`
- `expect.toPass`
- `expect.extend`
- asymmetric matchers: `expect.any`, `expect.anything`, `expect.objectContaining`,
  `expect.arrayContaining`, `expect.stringContaining`, `expect.stringMatching`

## 4. Manual Testing Angle

Manual testers ask: what proves this behavior? Automation assertions should mirror that.
Do not assert that a container exists if the real requirement is that the submitted
status, saved data, or permission behavior is correct.

## 5. Automation Angle

Prefer web-first assertions for UI state:

```typescript
await expect(page.getByRole('alert')).toContainText('Saved');
```

Avoid pulling a value too early and asserting synchronously:

```typescript
expect(await locator.textContent()).toBe('Saved'); // can be flaky
```

Use `expect.poll` for eventually consistent non-UI state:

```typescript
await expect.poll(async () => {
  const response = await page.request.get('/api/jobs/123');
  return response.status();
}).toBe(200);
```

## 6. Exact Surface To Remember

```typescript
await expect(locator).toBeVisible();
await expect(locator).toBeHidden();
await expect(locator).toHaveText('Submitted');
await expect(locator).toContainText('Submit');
await expect(locator).toHaveCount(3);
await expect(locator).toHaveValue('sagar@example.com');
await expect(locator).toBeChecked();
await expect(locator).toBeEnabled();
await expect(locator).toBeDisabled();
await expect(locator).toHaveAttribute('href', '/dashboard');
```

```typescript
await expect(page).toHaveURL(/.*dashboard/);
await expect(page).toHaveTitle(/Dashboard/);
await expect(response).toBeOK();
```

```typescript
await expect.soft(locator).toHaveText('Success');
await expect(locator, 'success message should appear').toBeVisible();
const slowExpect = expect.configure({ timeout: 10000 });
await slowExpect(locator).toHaveText('Ready');
```

## 7. Real Project Example

> "In BASES_PLAYWRIGHT, I do not stop at clicking through a flow. After each meaningful
> action, I assert the user-visible or API-backed outcome: status text, URL, row count,
> saved field value, or response status. A green test with a weak assertion is dangerous
> because it creates false confidence."

## 8. Common Mistakes / Weak Areas

- Asserting only visibility when the real behavior is data correctness.
- Using generic assertions for async UI state.
- Forgetting `await` before web-first assertions.
- Using exact text for dynamic messages.
- Overusing soft assertions.
- Treating screenshot assertions as functional validation.
- Using `.not` without first proving the right state/change.

## 9. 30-Second Answer

Playwright assertions use `expect`. For UI, I prefer web-first assertions like
`toBeVisible`, `toHaveText`, `toHaveURL`, and `toHaveCount` because they auto-retry until
the expected state appears. Generic assertions like `toEqual` do not auto-retry, so I use
them for captured values or API bodies, not waiting on UI.

## 10. 2-Minute Answer

My assertion strategy is behavior-first. I assert what proves the requirement: success
message, URL, persisted field value, permission result, row count, or API response. In
Playwright, locator/page assertions are async and retry until timeout, which makes them
better for dynamic UI than reading text once and using `toBe`.

For APIs or computed values, I use generic assertions like `toMatchObject`,
`toContainEqual`, or `toHaveProperty`. For eventually consistent backend behavior, I use
`expect.poll` or `expect.toPass`. The trade-off is not to assert everything. Too many
low-value assertions make tests noisy; too few make them meaningless.

## 11. Interview Q&A

### Q: `toHaveText` vs `toContainText`?

`toHaveText` expects the complete text match. `toContainText` accepts partial text. Use
exact when the full value matters; use contains for dynamic or longer text.

### Q: `toBeVisible` vs `toBeAttached`?

Attached means present in DOM. Visible means user can see it. A hidden element can be
attached but not visible.

### Q: Why avoid `expect(await locator.textContent()).toBe()`?

It reads text once. Web-first assertions retry, so `await expect(locator).toHaveText()`
is safer for async UI.

### Q: When use soft assertions?

For collecting multiple independent checks in one flow, but not when later steps depend
on earlier checks passing.

### Q: When use `expect.poll`?

When the thing to retry is not a simple locator state, such as polling API/job status.

## 12. Quick Revision Points

- Web-first assertions auto-retry.
- Generic assertions do not auto-retry.
- Assert behavior, not implementation.
- `toBeAttached` is DOM; `toBeVisible` is user-visible.
- Use `expect.poll` for eventually consistent non-UI checks.
