# Navigation And Events

## 1. What It Is

Navigation is moving the page to a new URL or document state. Events are browser/page
signals such as popup, download, dialog, request, response, console, and worker events.

## 2. Why It Matters In Interviews

Many flaky Playwright tests come from waiting for navigation or events incorrectly.
Interviewers often ask popup/download/API-call patterns because they reveal practical
experience.

## 3. Complete Topic Breakdown

| Surface                 | Syntax                            | Use                                   |
| ----------------------- | --------------------------------- | ------------------------------------- |
| Navigate                | `page.goto('/path')`            | Open a URL                            |
| URL assertion           | `expect(page).toHaveURL()`      | Verify navigation result              |
| Load state              | `page.waitForLoadState()`       | Wait for document/load state          |
| Popup                   | `page.waitForEvent('popup')`    | New tab/window                        |
| Any new page in context | `context.waitForEvent('page')`  | `target="_blank"` or unknown opener |
| Download                | `page.waitForEvent('download')` | Export/download                       |
| Dialog                  | `page.once('dialog')`           | Alert/confirm/prompt                  |
| Request                 | `page.waitForRequest()`         | Verify request fired                  |
| Response                | `page.waitForResponse()`        | Verify API response                   |
| Listener                | `page.on()` / `page.off()`    | Observe ongoing events                |

### 3a. `waitForLoadState` in depth

Signature: `await page.waitForLoadState(state?, { timeout? })`. It waits until the page
reaches a **document-loading milestone**, then resolves. If that milestone was already
reached, it resolves immediately.

| State                  | Resolves when                                                         | Notes                                                                   |
| ---------------------- | --------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| `'load'` (default)   | The `load` event fired: HTML, stylesheets, images, scripts finished | What you get if you pass nothing                                        |
| `'domcontentloaded'` | HTML parsed and the DOM built; images/styles may still be loading     | Earliest and fastest of the three                                       |
| `'networkidle'`      | No network connections for at least 500 ms                            | Playwright docs discourage it for tests; see the trade-off in section 5 |

Things that make it less useful than it looks:

- `page.goto()` **already waits for `load`**, so calling `waitForLoadState()` right after
  it is usually redundant. The same `waitUntil` choices exist on `goto` itself
  (`'load' | 'domcontentloaded' | 'networkidle' | 'commit'`; `'commit'` exists only there).
- Actions that trigger a navigation (a `click` on a link) wait for that navigation, so you
  rarely need it after them.
- **SPAs:** route changes use `pushState`, no new document loads, so no `load` event fires.
  Waiting for a load state tells you nothing. Use `expect(page).toHaveURL()` plus an
  assertion on a visible element.
- **"Loaded" is not "ready":** `load` can fire before API data renders. Assert the
  specific element or response you need, not the load state.
- Where it genuinely helps: a freshly opened popup / new tab (`newPage.waitForLoadState()`),
  or a non-SPA page you want settled before reading something from the document.

Rule of thumb: `domcontentloaded` or `load` occasionally; `networkidle` almost never;
web-first assertions most of the time.

### 3b. Listeners: `page.on`, `page.once`, `page.off` in depth

A listener is a callback the browser calls **every time** an event fires, in the
background, while your test keeps running.

| Method                        | Behaviour                                                                                                                    |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `page.on(event, handler)`   | Stays registered; runs on every matching event                                                                               |
| `page.once(event, handler)` | Runs on the first event, then removes itself                                                                                 |
| `page.off(event, handler)`  | Removes a listener. Needs the **same function reference** you registered, so an inline arrow function can't be removed |

```typescript
const errors: string[] = [];
const onConsole = (msg: ConsoleMessage) => {
  if (msg.type() === 'error') errors.push(msg.text());
};
page.on('console', onConsole);

await page.goto('/dashboard');
// ... test steps ...

page.off('console', onConsole);          // stop listening
expect(errors).toEqual([]);              // assert in the test body
```

Commonly used events:

| Event                      | Fires when                               | Typical use                      |
| -------------------------- | ---------------------------------------- | -------------------------------- |
| `console`                | Page logs to the console                 | Collect JS errors, debug         |
| `pageerror`              | Uncaught exception in the page           | Fail the test on real JS crashes |
| `request` / `response` | Any request is sent / response received  | Log or inspect traffic           |
| `requestfailed`          | A request fails (network error, blocked) | Spot failing calls               |
| `dialog`                 | alert/confirm/prompt opens               | Accept/dismiss                   |
| `popup`                  | The page opens a new tab/window          | Handle new page                  |
| `download`               | A download starts                        | Capture the file                 |
| `framenavigated`         | A frame navigates                        | Track navigation                 |
| `close` / `crash`      | The page closes or crashes               | Diagnose unexpected teardown     |

Also exists on the context: `context.on('page', ...)` catches **any** new page in that
browser context.

**`on` vs `waitForEvent`:**

- `waitForEvent` / `waitForResponse` give you a promise to `await` and assert on. Use them
  when your test **needs** that one event.
- `on` is for **ongoing observation or side effects**, such as collecting errors or
  logging. Don't use `on` as a way to wait; there is nothing to await.

**Pitfalls to remember:**

- **Dialog handlers must act.** If you register a `dialog` listener, you must call
  `accept()` or `dismiss()`, or the page hangs. With no listener, Playwright dismisses
  dialogs automatically.
- **Register before the action** that triggers the event, or you miss it.
- **Listeners belong to one page object.** A popup is a different page and needs its own
  listener, or use `context.on('page')`. Listeners are gone when the page closes.
- **Don't assert inside an async handler.** An error thrown there is not awaited by your
  test. Collect the data into an array, then assert in the test body (as above).
- **Don't let listeners pile up.** `on` inside a loop or a repeated step registers
  duplicates on the same page. Use `once` for one-time handling, or `off` when done.
- Put always-on diagnostics (console errors, page errors) in a fixture so every test gets
  them without repeating the setup.

## 4. Manual Testing Angle

Manual testers wait for meaningful outcomes: new page loaded, export downloaded, popup
opened, API-backed result shown. Automation should wait for those same observable
signals.

## 5. Automation Angle

Use pre-wait for events that may happen immediately.

```typescript
const popupPromise = page.waitForEvent('popup');
await page.getByRole('link', { name: 'Open report' }).click();
const popup = await popupPromise;
```

When the new tab can be created anywhere inside the same browser context:

```typescript
const pagePromise = context.waitForEvent('page');
await page.getByText('open new tab').click();
const newPage = await pagePromise;
await newPage.waitForLoadState();
```

Trade-off: do not overuse `networkidle`. Modern apps may keep long polling or background
requests open.

## 6. Exact Surface To Remember

```typescript
await page.goto('/dashboard');
await expect(page).toHaveURL(/dashboard/);
await page.waitForLoadState('domcontentloaded');
```

```typescript
const responsePromise = page.waitForResponse(resp =>
  resp.url().includes('/api/search') && resp.status() === 200
);
await page.getByRole('button', { name: 'Search' }).click();
const response = await responsePromise;
```

```typescript
page.on('console', msg => console.log(msg.text()));
page.once('dialog', dialog => dialog.accept());
```

## 7. Real Project Example

> "In BASES_PLAYWRIGHT, export and search flows use event/response waits. I start the wait
> before clicking because the event can fire immediately. Then I assert the URL, response,
> popup, or downloaded file instead of sleeping." 

## 8. Common Mistakes / Weak Areas

- Starting event wait after the action.
- Using `waitForTimeout` after navigation.
- Using `networkidle` everywhere.
- Asserting URL with string when regex is more stable.
- Not checking response status/body after API-driven UI action.
- Leaving event listeners active longer than needed.

## 9. 30-Second Answer

For navigation, I use `page.goto`, `expect(page).toHaveURL`, and targeted load-state or
response waits. For events like popup, download, dialog, request, and response, I start
the wait before the action, then await the event and assert the result.

## 10. 2-Minute Answer

Navigation and events are timing-sensitive. The safe pattern is to wait for the real
browser signal, not arbitrary time. For page navigation, I usually assert URL or page
content. For downloads/popups/API calls, I create the promise first, perform the action,
then await the promise. For ongoing diagnostics, I use `page.on`, and for one-time events
like dialog I use `page.once`.

The trade-off is choosing the right signal. Waiting for full network idle can be slow or
unreliable in apps with background polling. A specific URL, locator assertion, or API
response is usually better.

## 11. Interview Q&A

### Q: Why create event promise before click?

Because the event may fire immediately. If the wait starts after the click, the test can
miss it.

### Q: When use `waitForResponse`?

When the UI action depends on a specific API response and you need to verify it happened
or inspect status/body.

### Q: What are the `waitForLoadState` states and when do you use it?

`load` (default), `domcontentloaded`, `networkidle` (no network for 500 ms, discouraged).
`goto` already waits for `load`, and SPAs don't fire it on route change, so I mostly use
web-first assertions. I use it for a new popup/tab, and only occasionally otherwise.

### Q: Is `waitForLoadState('networkidle')` always good?

No. Apps with background polling may never become idle or may make it slow. Prefer
specific state.

### Q: `page.on` vs `page.once` vs `page.waitForEvent`?

`on` stays registered and runs every time, for ongoing observation like collecting console
errors. `once` runs one time and removes itself, which suits dialogs. `waitForEvent` gives
a promise I await and assert on when the test needs that event. To remove a listener with
`off`, I need the original function reference.

### Q: What happens if you register a `dialog` listener and don't handle it?

The page hangs, because the dialog stays open. Once you add a listener you must call
`accept()` or `dismiss()`. With no listener, Playwright dismisses dialogs by itself.

### Q: How handle popup?

Start `page.waitForEvent('popup')`, click, then await the popup page and assert it.

### Q: `page.waitForEvent('popup')` vs `context.waitForEvent('page')`?

Use popup when the new page is opened by the current page. Use context page event when
you need to catch any new page created in that browser context.

## 12. Quick Revision Points

- Wait for signal, not time.
- Event wait before action.
- `on` = ongoing, `once` = one-time, `off` needs the same function reference.
- A `dialog` listener must accept/dismiss; assert outside handlers, not inside.
- URL/content/response are better signals than sleep.
- `popup` is page-specific; `context.waitForEvent('page')` catches any new page in the
  context.
- Use `page.once` for one-time events.
- Be careful with `networkidle`.
