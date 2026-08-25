# Playwright - Fixtures, Hooks, And Auth Setup: Practical Guide

## What it is

Fixtures and hooks are Playwright's setup and teardown mechanisms.

**Fixtures** provide ready-made objects to a test. Playwright already gives built-in
fixtures like `page`, `context`, `browser`, `request`, and `browserName`. We can also
create our own fixtures, like `customUser1`, `adminPage`, `apiClient`, or
`testProject`.

**Hooks** run at fixed points in the test file:

- `beforeAll` runs once before all tests in that file.
- `afterAll` runs once after all tests in that file.
- `beforeEach` runs before every test.
- `afterEach` runs after every test.

The simple rule:

> Fixtures are for reusable test setup. Hooks are for file-level lifecycle work.

---

## Why it matters

Interviewers ask fixtures and hooks because they reveal framework maturity.

Anyone can write:

```typescript
test('login test', async ({ page }) => {
  await page.goto('/login');
});
```

But framework-level thinking is:

- How do all tests get authenticated state without logging in every time?
- How do tests create and clean data safely?
- How do we avoid duplicated `beforeEach` login code?
- How do we keep tests isolated when running in parallel?
- How do we separate UI storage state from API bearer token handling?

Fixtures and hooks are where these answers live.

---

## 30-second answer

> "In Playwright, I use fixtures for reusable setup like logged-in users, API clients,
> page objects, and test data. Fixtures wrap setup before `await use()` and cleanup after
> it, so every test gets clean dependencies. Hooks are still useful, but mostly for
> file-level lifecycle like `beforeAll` setup or `afterEach` diagnostics. For auth, I
> prefer storage state for UI tests so we do not log in before every test, and separate
> bearer-token fixtures for API tests because UI storage state and API auth are not the
> same thing."

---

## 2-minute answer

> "I separate setup into three layers.
>
> First, Playwright built-in fixtures: `page`, `context`, `browser`, `request`. Every
> test that asks for `page` gets a fresh page inside an isolated browser context, so it
> does not reuse state from another test.
>
> Second, custom fixtures using `base.extend`. For example, if many tests need a logged
> in user or a created project, I create a fixture. Code before `await use(value)` is
> setup, code after it is teardown. That gives me a clean place for API seeding and
> cleanup.
>
> Third, hooks. I avoid putting reusable login or data setup in `beforeEach` because it
> gets duplicated across files and runs even for tests that do not need it. Hooks are
> still useful for file-level work, like attaching screenshots after failure, setting a
> temporary route for a describe block, or starting/stopping something once for the file.
>
> For authentication, UI tests usually load storage state from config or `test.use()`.
> That means login happens once in a setup project and tests start already authenticated.
> For API tests, I do not depend on UI storage state; I create a request context with an
> Authorization header or expose a token fixture. If token expiry is possible, the
> fixture validates or refreshes the token before giving it to tests.
>
> The trade-off is that storage state makes tests faster, but it skips the login journey.
> So I still keep separate login/logout tests that do not use storage state."

---

## Real project example

In the Playwright track, the pattern to explain is not "one big setup file". The clean
answer is small fixtures with clear ownership.

### Built-in fixture usage

```typescript
import { test, expect } from '@playwright/test';

test('dashboard loads for authenticated user', async ({ page }) => {
  await page.goto('/dashboard');
  await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
});
```

Here `page` is not reused from another test. Playwright creates it for this test inside
an isolated context.

### Custom user fixture with setup and teardown

```typescript
import { test as base, expect } from '@playwright/test';

type TestUser = {
  id: string;
  email: string;
};

type Fixtures = {
  customUser1: TestUser;
};

export const test = base.extend<Fixtures>({
  customUser1: async ({ request }, use) => {
    const createResponse = await request.post('/api/test-users', {
      data: {
        name: `user-${Date.now()}`,
        role: 'standard',
      },
    });

    expect(createResponse.status()).toBe(201);
    const user = await createResponse.json();

    await use(user);

    await request.delete(`/api/test-users/${user.id}`);
  },
});
```

How to explain `use()`:

```text
Before await use(user)  = setup
During await use(user)  = test runs with that user
After await use(user)   = teardown
```

Test using the fixture:

```typescript
import { test, expect } from '../fixtures/users.fixture';

test('user can see own profile', async ({ page, customUser1 }) => {
  await page.goto(`/users/${customUser1.id}`);
  await expect(page.getByText(customUser1.email)).toBeVisible();
});
```

### Storage state for logged-in UI user

```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  projects: [
    {
      name: 'setup auth',
      testMatch: /auth\.setup\.ts/,
    },
    {
      name: 'chromium',
      use: {
        baseURL: process.env.BASE_URL,
        storageState: '.auth/customUser1.json',
      },
      dependencies: ['setup auth'],
    },
  ],
});
```

```typescript
// tests/auth.setup.ts
import { test as setup, expect } from '@playwright/test';

setup('login as customUser1', async ({ page, context }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill(process.env.TEST_USER_EMAIL!);
  await page.getByLabel('Password').fill(process.env.TEST_USER_PASSWORD!);
  await page.getByRole('button', { name: 'Login' }).click();

  await expect(page).toHaveURL(/dashboard/);
  await context.storageState({ path: '.auth/customUser1.json' });
});
```

All normal UI tests now start with cookies/localStorage already loaded.

### Different logged-in roles with `test.use`

```typescript
import { test, expect } from '@playwright/test';

test.describe('admin flows', () => {
  test.use({ storageState: '.auth/admin.json' });

  test('admin can open user management', async ({ page }) => {
    await page.goto('/admin/users');
    await expect(page.getByRole('heading', { name: 'User Management' })).toBeVisible();
  });
});
```

This is a clean answer when they ask: "How do you handle multiple users?"

### API auth token fixture

```typescript
import { test as base, request, expect, APIRequestContext } from '@playwright/test';

type Fixtures = {
  apiClient: APIRequestContext;
};

export const test = base.extend<Fixtures>({
  apiClient: async ({}, use) => {
    const loginContext = await request.newContext();

    const loginResponse = await loginContext.post('/api/auth/login', {
      data: {
        username: process.env.API_USER!,
        password: process.env.API_PASSWORD!,
      },
    });

    expect(loginResponse.status()).toBe(200);
    const { token } = await loginResponse.json();

    const apiClient = await request.newContext({
      extraHTTPHeaders: {
        Authorization: `Bearer ${token}`,
      },
    });

    await use(apiClient);

    await apiClient.dispose();
    await loginContext.dispose();
  },
});
```

Important interview line:

> "UI storage state helps browser tests. API tests should manage API auth through headers
> or request context. I do not assume the browser's storage state automatically solves
> API authentication."

---

## Common interview Q&A

### Q1: What is a fixture in Playwright?

**Answer:** A fixture is a dependency Playwright prepares for a test. Built-in fixtures
include `page`, `context`, `browser`, and `request`. Custom fixtures are created with
`base.extend()` for things like logged-in users, page objects, API clients, or test data.

### Q2: What does `await use(user)` mean?

**Answer:** It is the handoff point. Setup happens before it, the test runs while it is
active, and teardown happens after it.

```typescript
user: async ({ request }, use) => {
  const user = await createUser(request);  // setup
  await use(user);                         // test gets user
  await deleteUser(request, user.id);      // teardown
}
```

### Q3: What is the difference between test-scoped and worker-scoped fixtures?

**Answer:** Test-scoped fixtures run for every test. Worker-scoped fixtures run once per
worker process. I use test scope for mutable data like users/orders because each test
needs isolation. I use worker scope only for expensive read-only setup.

```typescript
export const test = base.extend<{
  testUser: User;
  referenceData: ReferenceData;
}>({
  testUser: async ({ request }, use) => {
    const user = await createUser(request);
    await use(user);
    await deleteUser(request, user.id);
  },

  referenceData: [async ({ request }, use) => {
    const data = await loadReferenceData(request);
    await use(data);
  }, { scope: 'worker' }],
});
```

### Q4: When do you use hooks instead of fixtures?

**Answer:** I use hooks when the behavior belongs to the whole file or describe block,
not to one reusable dependency. Examples: attach logs after each failure, configure a
temporary route for a group, or run one setup for the file. If the same setup is needed
across multiple files, it should usually become a fixture.

### Q5: Should login be in `beforeEach`?

**Answer:** Usually no. If login is repeated in `beforeEach` across many files, it is
duplicated and slow. Prefer storage state for already-authenticated UI tests, and keep
separate login tests to cover the login journey itself.

### Q6: What happens if `beforeAll` fails?

**Answer:** The tests depending on that `beforeAll` will fail or be skipped because the
file-level setup failed. That is why `beforeAll` should be used carefully and should not
hide too much business setup.

### Q7: What happens if fixture setup fails?

**Answer:** The test fails before reaching the test body. That is good because the test
dependency could not be prepared. The failure points to setup, not product behavior, so
clear error messages and small fixtures matter.

### Q8: How do you handle expired storage state?

**Answer:** The setup project regenerates it before the test run. If the app has short
token expiry, I validate the logged-in state after loading storage state. If validation
fails, regenerate the state. For API tests, I refresh bearer tokens in the API fixture.

### Q9: What should not go inside fixtures?

**Answer:** Assertions for the actual feature under test. Fixtures can validate setup
enough to fail fast, but they should not hide business assertions. The test body should
still clearly show what behavior is being verified.

---

## Weak areas to watch

- Do not say Playwright has a "project-scoped fixture" like test/worker fixtures.
  Projects are configured in `playwright.config.ts`; fixtures are mainly test-scoped or
  worker-scoped.
- Do not put every setup into `beforeEach`. It becomes duplicated, always runs, and hides
  test intent.
- Do not use one shared mutable user across parallel tests. That creates data collisions.
- Do not say UI storage state automatically handles API token validation. UI auth and API
  auth are related, but not the same mechanism.
- Do not overuse worker scope because it is faster. Scope should follow mutability, not
  only cost.
- Do not hide real test assertions inside fixtures. Fixtures prepare; tests verify.

---

## Key takeaway

> Fixtures make setup reusable and isolated. Hooks manage lifecycle. Storage state makes
> UI auth fast, but it creates a login coverage gap and does not replace API token
> management.

The interview line that lands:

> "I prefer fixtures for reusable dependencies, hooks for lifecycle, storage state for
> fast UI auth, and separate API token fixtures for API tests. That keeps tests fast,
> isolated, and clear."
