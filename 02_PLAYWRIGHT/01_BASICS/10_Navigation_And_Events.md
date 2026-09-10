# Navigation And Events

## 1. What It Is

Navigation is moving the page to a new URL or document state. Events are browser/page
signals such as popup, download, dialog, request, response, console, and worker events.

## 2. Why It Matters In Interviews

Many flaky Playwright tests come from waiting for navigation or events incorrectly.
Interviewers often ask popup/download/API-call patterns because they reveal practical
experience.

## 3. Complete Topic Breakdown

| Surface | Syntax | Use |
|---|---|---|
| Navigate | `page.goto('/path')` | Open a URL |
| URL assertion | `expect(page).toHaveURL()` | Verify navigation result |
| Load state | `page.waitForLoadState()` | Wait for document/load state |
| Popup | `page.waitForEvent('popup')` | New tab/window |
| Download | `page.waitForEvent('download')` | Export/download |
| Dialog | `page.once('dialog')` | Alert/confirm/prompt |
| Request | `page.waitForRequest()` | Verify request fired |
| Response | `page.waitForResponse()` | Verify API response |
| Listener | `page.on()` / `page.off()` | Observe ongoing events |

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

### Q: Is `waitForLoadState('networkidle')` always good?

No. Apps with background polling may never become idle or may make it slow. Prefer
specific state.

### Q: How handle popup?

Start `page.waitForEvent('popup')`, click, then await the popup page and assert it.

## 12. Quick Revision Points

- Wait for signal, not time.
- Event wait before action.
- URL/content/response are better signals than sleep.
- Use `page.once` for one-time events.
- Be careful with `networkidle`.
