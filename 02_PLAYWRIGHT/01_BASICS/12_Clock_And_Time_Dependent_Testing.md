# Clock And Time-Dependent Testing

## 1. What It Is

Clock testing controls browser time so tests can validate time-dependent behavior without
waiting in real time. It helps with timers, countdowns, date filters, expiry, scheduled
UI behavior, and time-based validation.

## 2. Why It Matters In Interviews

This is a deeper Playwright topic. It shows you know how to avoid slow/flaky tests when
the feature depends on time.

## 3. Complete Topic Breakdown

Time-dependent examples:

- token/session expiry UI,
- countdown timer,
- scheduled banner,
- date filter defaulting to today,
- SLA/status changing after time,
- retry/backoff UI,
- debounce/throttle behavior.

Playwright can mock browser clock behavior, install a fixed time, and move time forward.
Use it when the app behavior depends on browser-side time.

## 4. Manual Testing Angle

Manual testing of time-based behavior is slow and error-prone if you wait for real time
or keep changing system clock. Before automation, decide whether the behavior is browser
time, server time, database time, or scheduled backend behavior.

## 5. Automation Angle

Use clock control for browser-side time. Do not pretend it proves server-side schedulers,
database jobs, or real token expiry unless the test also validates those layers.

Trade-off: mocked time makes tests fast, but it can hide integration bugs if the real
system depends on server time.

## 6. Exact Surface To Remember

Common pattern:

```typescript
await page.clock.install({ time: new Date('2026-09-10T09:00:00') });
await page.goto('/offers');
await expect(page.getByText('Offer active')).toBeVisible();

await page.clock.fastForward('01:00:00');
await expect(page.getByText('Offer expired')).toBeVisible();
```

For simple date default:

```typescript
await page.clock.setFixedTime(new Date('2026-09-10T00:00:00'));
await page.goto('/reports');
await expect(page.getByLabel('Date')).toHaveValue('2026-09-10');
```

## 7. Real Project Example

> "In BASES_PLAYWRIGHT, if a UI flow depends on date filters or expiry messaging, I would
> first ask whether the logic uses browser time or server time. If it is browser-side, I
> can control clock. If it is server-side, API/test data setup is safer than pretending
> browser clock proves the backend."

## 8. Common Mistakes / Weak Areas

- Waiting real minutes in a test.
- Mocking browser time when behavior depends on server time.
- Changing system clock manually.
- Using clock control without understanding when app reads time.
- Combining clock mocks with unrelated assertions.
- Forgetting that timezone/locale can also affect displayed dates.

## 9. 30-Second Answer

For time-dependent UI, I avoid real waits. If the behavior depends on browser time, I use
Playwright clock control to set or move time and assert the UI state. If the behavior
depends on server time, I use API/test data setup or backend validation instead.

## 10. 2-Minute Answer

Time-based tests are easy to make slow or flaky. The first question is where the source
of truth lives. If the browser calculates the current date, countdown, or client-side
expiry, Playwright's clock control can make the test deterministic. If the server decides
expiry or scheduled status, mocking browser time is not enough; I need API setup or
server-side test data.

The trade-off is realism. Clock control makes UI tests fast and stable, but overusing it
can hide integration issues. So I use it for browser-side behavior and keep server-time
logic validated closer to the API/backend.

## 11. Interview Q&A

### Q: Why not use `waitForTimeout` for timer tests?

It slows the suite and can still be flaky. Control time or test the logic at a better
level.

### Q: Can browser clock prove token expiry?

Only if the expiry behavior is browser-side. Real token expiry often involves server
validation, so API/auth tests may be needed.

### Q: What else affects date UI?

Timezone, locale, server time, browser time, and test data.

### Q: When is clock mocking risky?

When it hides real integration behavior or when the app gets time from the server.

## 12. Quick Revision Points

- Time tests should not sleep.
- Ask browser time or server time.
- Clock control is for browser-side behavior.
- Server expiry needs API/backend validation.
- Timezone/locale matter for date display.
