# Browser, Context, And Page

## 1. What It Is

Browser, context, and page are Playwright's core runtime objects.

- Browser = actual browser process.
- Browser context = isolated browser profile/session.
- Page = tab inside a context.

One browser can have many contexts. One context can have many pages.

## 2. Why It Matters In Interviews

This is the architecture question. Interviewers use it to test whether you understand
isolation, parallelism, storage state, and multi-user testing.

## 3. Complete Topic Breakdown

| Object | Meaning | Common use |
|---|---|---|
| `browser` | Browser process | Create extra contexts |
| `context` | Isolated session/profile | Cookies, storage, permissions, viewport |
| `page` | Tab | Navigate, locate, act, assert |

### Browser

Normally provided by Playwright Test.

```typescript
test('manual context', async ({ browser }) => {
  const context = await browser.newContext();
  const page = await context.newPage();
  await context.close();
});
```

### Context

Context owns session-level state:

- cookies,
- local storage/session storage,
- storage state,
- permissions,
- geolocation,
- viewport,
- locale/timezone,
- extra headers.

### Page

Page is where most test actions happen:

```typescript
await page.goto('/dashboard');
await page.getByRole('button', { name: 'Create' }).click();
await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
```

## 4. Manual Testing Angle

Manual testing naturally separates sessions: logged-in user, admin, clean browser, logged
out user, another tab. Playwright contexts model that cleanly.

## 5. Automation Angle

Use context isolation for independent tests and multi-user flows.

```typescript
test('admin sees user request', async ({ browser }) => {
  const userContext = await browser.newContext({ storageState: '.auth/user.json' });
  const adminContext = await browser.newContext({ storageState: '.auth/admin.json' });

  const userPage = await userContext.newPage();
  const adminPage = await adminContext.newPage();

  await userPage.goto('/requests');
  await adminPage.goto('/admin/requests');

  await userContext.close();
  await adminContext.close();
});
```

Trade-off: extra contexts/pages cost memory and time, so create them only when the
scenario needs them.

## 6. Exact Surface To Remember

```typescript
const context = await browser.newContext();
const page = await context.newPage();
await context.close();
```

```typescript
test.use({ storageState: '.auth/user.json' });
```

```typescript
await context.storageState({ path: '.auth/admin.json' });
```

Built-in fixtures:

- `page`
- `context`
- `browser`
- `browserName`

## 7. Real Project Example

> "In BASES_PLAYWRIGHT, most tests use the built-in `page` fixture with storage state.
> For role-based flows, I can create separate contexts for admin and normal user so
> cookies and permissions do not leak."

## 8. Common Mistakes / Weak Areas

- Saying page and context are the same.
- Sharing one page across tests.
- Forgetting to close manually created contexts.
- Using one context for two roles.
- Thinking storage state also tests login.
- Ignoring resource cost of many contexts.

## 9. 30-Second Answer

Browser is the process, context is an isolated browser session, and page is a tab. Each
test normally gets a fresh context/page, which improves isolation and parallel execution.
For multi-user testing, I create separate contexts with different storage states.

## 10. 2-Minute Answer

The browser/context/page model is Playwright's foundation. Browser is the engine process.
Context is an isolated session like an incognito profile: it owns cookies, storage,
permissions, viewport, locale, timezone, and auth state. Page is the tab where actions and
assertions happen.

This matters because parallel tests should not share session state. It also makes
multi-user flows cleaner: admin and user can run in different contexts. The trade-off is
that manually created contexts must be closed and not overused.

## 11. Interview Q&A

### Q: Why context if we already have page?

Page is only a tab. Context owns the browser session state.

### Q: How do you test two users?

Create two contexts, usually with different `storageState` files, and then create pages
from them.

### Q: What can leak if context is shared?

Cookies, local storage, auth state, permissions, and sometimes feature/session data.

### Q: Does Playwright create a new browser for every test?

Not necessarily. It can reuse browser processes while keeping tests isolated through
fresh contexts/pages.

## 12. Quick Revision Points

- Browser = process.
- Context = isolated session.
- Page = tab.
- Context is the isolation boundary.
- Use separate contexts for separate users.
