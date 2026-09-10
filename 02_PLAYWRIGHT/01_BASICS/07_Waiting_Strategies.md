# Waiting Strategies

## 1. What It Is

Waiting strategy is how a test waits for the application to reach the expected state.
Playwright has auto-waiting for actions and web-first assertions, but some waits still
need explicit event, navigation, response, or polling logic.

## 2. Why It Matters In Interviews

"Playwright auto-waits" is a common shallow answer. Strong answer explains where
auto-waiting helps and where it stops.

## 3. Complete Topic Breakdown

### Actionability checks

Before many actions, Playwright checks:

- visible,
- stable,
- receives events,
- enabled,
- editable where relevant.

### Actionability matrix

| Action | Visible | Stable | Receives Events | Enabled | Editable |
|---|:---:|:---:|:---:|:---:|:---:|
| `click`, `dblclick`, `check`, `uncheck`, `setChecked`, `tap` | Yes | Yes | Yes | Yes | - |
| `hover`, `dragTo` | Yes | Yes | Yes | - | - |
| `screenshot` | Yes | Yes | - | - | - |
| `fill`, `clear` | Yes | - | - | Yes | Yes |
| `selectOption` | Yes | - | - | Yes | - |
| `selectText` | Yes | - | - | - | - |
| `scrollIntoViewIfNeeded` | - | Yes | - | - | - |
| `blur`, `dispatchEvent`, `focus`, `press`, `pressSequentially`, `setInputFiles` | - | - | - | - | - |

This is the follow-up point: Playwright auto-waits, but **not every action waits for the
same things**.

### Wait types

| Wait | Syntax | Use |
|---|---|---|
| Action auto-wait | `locator.click()` | Waits for actionability |
| Web assertion | `expect(locator).toBeVisible()` | Waits for UI state |
| URL wait | `expect(page).toHaveURL()` | Navigation result |
| Event wait | `page.waitForEvent()` | Popup/download/dialog |
| Request/response wait | `page.waitForRequest/Response()` | API side effect |
| Polling | `expect.poll()` | Eventually consistent state |
| Function wait | `page.waitForFunction()` | Custom browser condition |
| Hard wait | `page.waitForTimeout()` | Avoid except temporary debugging |

## 4. Manual Testing Angle

Manual testers wait based on meaningful states: spinner gone, result visible, file
downloaded, status changed. Automation should wait for those same signals, not arbitrary
seconds.

## 5. Automation Angle

Prefer state-based waits:

```typescript
await expect(page.getByRole('status')).toHaveText('Completed');
```

Use event pre-wait:

```typescript
const responsePromise = page.waitForResponse('**/api/search');
await page.getByRole('button', { name: 'Search' }).click();
const response = await responsePromise;
```

Avoid:

```typescript
await page.waitForTimeout(5000);
```

## 6. Exact Surface To Remember

```typescript
await expect(locator).toBeVisible();
await expect(locator).toHaveText('Ready');
await expect(page).toHaveURL(/dashboard/);
await page.waitForLoadState('networkidle');
await page.waitForFunction(() => window.appReady === true);
```

```typescript
const popupPromise = page.waitForEvent('popup');
await page.getByRole('link', { name: 'Open report' }).click();
const popup = await popupPromise;
```

## 7. Real Project Example

> "In BASES_PLAYWRIGHT, I removed wasted hard waits and replaced them with web-first
> assertions and event waits. That helped reduce CI feedback time because tests waited for
> real app state instead of sleeping blindly."

## 8. Common Mistakes / Weak Areas

- Saying auto-waiting covers API response or custom business state.
- Using hard waits as normal strategy.
- Waiting for `networkidle` for every page without reason.
- Starting event wait after the click.
- Reading text once and using generic assertion.
- Increasing timeout instead of fixing data/state.

## 9. 30-Second Answer

Playwright auto-waits for actionability before actions and retries web-first assertions.
But it does not automatically know every business/API state, so I use explicit waits for
URL, response, popup/download events, or polling when needed. I avoid hard waits.

## 10. 2-Minute Answer

Waiting is about waiting for the right signal. For UI actions, Playwright checks
actionability: visible, stable, receives events, enabled, and editable where relevant.
For expected UI state, I use web-first assertions. For navigation, download, popup, or
API side effects, I start the wait before the action and then await the result.

The trade-off is timeout policy. Longer timeout can hide slowness and make CI slow.
Shorter timeout can create false failures. So I use default timeouts for normal flows and
targeted waits where the app genuinely needs time.

## 11. Interview Q&A

### Q: If Playwright auto-waits, why tests still fail?

Auto-waiting waits for actionability and assertions, not wrong data, failed API, bad
requirements, environment down, or custom business state.

### Q: Which actionability checks does click perform?

Click waits for one matching element, visible, stable, receives events, and enabled. It
does not wait for business state like "data saved".

### Q: Does `press()` wait like `click()`?

No. Keyboard-style actions such as `press()` do not perform the full click actionability
matrix. If the element/state must be visible or ready, assert that separately.

### Q: Why is hard wait bad?

It is either too short and flaky or too long and slow. It waits for time, not condition.

### Q: When use `waitForFunction`?

When the state exists in the browser but is not easily represented by a locator or URL.

### Q: Why start event wait before action?

If the event happens immediately, starting the wait after the action can miss it.

## 12. Quick Revision Points

- Auto-waiting is not magic.
- Different actions have different actionability checks.
- Wait for state, not seconds.
- Web-first assertions retry.
- Event waits start before action.
- Hard waits are temporary debugging only.
