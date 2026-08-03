# Playwright - APIRequestContext: Built-in API Testing

## What is APIRequestContext?

Playwright's built-in HTTP client. It lets you make API calls directly from your tests — no Axios, no Supertest, no RestAssured.

```typescript
test('API returns user', async ({ request }) => {
  const response = await request.get('/api/users/1');
  expect(response.status()).toBe(200);
});
```

That `request` fixture is APIRequestContext. One framework covers UI and API instead of two.

---

## Why It Matters (Interview Context)

This is a strong differentiator for you specifically, because you've done API testing **both ways** — RestAssured in Java on the legacy track, Playwright's request context on the modern one. That comparison is credible in a way it isn't for someone who's only used one.

Interviewers are checking:
1. **Do you know Playwright can test APIs at all?** (many candidates don't)
2. **Do you understand the two request modes and when each applies?** (the real depth question)
3. **Do you use API tests strategically**, or automate everything through the browser?

---

## Your 30-Second Interview Answer

> "Playwright has a built-in request context, so I don't need a separate HTTP client — one framework covers UI and API. The distinction that matters is between the isolated `request` fixture and `page.request`, which shares cookies and auth state with the browser. I use the isolated one for pure API tests and `page.request` when I need calls made as the logged-in user. Coming from RestAssured, the big practical win is not maintaining two frameworks and two sets of test data helpers."

---

## Your 2-Minute Deep Answer

> "There are two ways to make requests, and the difference is the thing worth understanding.
>
> **1. The `request` fixture — isolated**
>
> ```typescript
> test('Create user via API', async ({ request }) => {
>   const response = await request.post('/api/users', {
>     data: { email: 'test@example.com', role: 'admin' }
>   });
>   expect(response.ok()).toBeTruthy();
> });
> ```
>
> This has its own cookie jar. It knows nothing about any browser session. Use it for pure API tests and for seeding data before a UI test starts.
>
> **2. `page.request` — shares browser state**
>
> ```typescript
> test('Fetch as logged-in user', async ({ page }) => {
>   await page.goto('/dashboard');       // user is now authenticated
>   const response = await page.request.get('/api/me');
>   // this call carries the browser's cookies automatically
> });
> ```
>
> This shares cookies, storage state, and auth with the page. Use it when the request must be made *as the logged-in user* — checking what the UI would receive, or verifying a backend state change after a UI action.
>
> **Getting this wrong is a real source of confusion.** A test hits `/api/me` with the isolated `request` fixture, gets a 401, and the developer spends an hour investigating auth — when the actual issue is that the isolated context was never logged in. Knowing which one you're using explains most 'why is this 401' incidents.
>
> ---
>
> **Setting up a shared context:**
>
> For consistent base URL and headers across tests:
>
> ```typescript
> // playwright.config.ts
> use: {
>   baseURL: process.env.API_URL,
>   extraHTTPHeaders: {
>     'Accept': 'application/json',
>   },
> }
> ```
>
> Or build a context explicitly when you need different auth:
>
> ```typescript
> const apiContext = await playwright.request.newContext({
>   baseURL: process.env.API_URL,
>   extraHTTPHeaders: { 'Authorization': `Bearer ${token}` },
> });
> ```
>
> ---
>
> **The three ways I actually use it:**
>
> **Seeding test data.** This is the highest-value use, and it connects to everything else in the framework. Instead of clicking through three screens to reach a state, create it in one call:
> ```typescript
> const project = await request.post('/api/projects', { data: buildProject() });
> await page.goto(`/projects/${project.id}`);   // straight to the thing under test
> ```
> Fifteen seconds becomes under one, and a failure now points at the feature under test rather than at some unrelated setup screen.
>
> **Pure API tests.** Validation rules, permissions, calculations — anything with no UI dependency. Runs in ~200ms versus ~10s through a browser, and it's far more stable.
>
> **Verifying backend state after a UI action.** The user clicks Save; did the data actually persist?
> ```typescript
> await page.getByRole('button', { name: 'Save' }).click();
> const response = await page.request.get(`/api/projects/${id}`);
> expect((await response.json()).name).toBe('Updated Name');
> ```
> This catches a real class of bug — the UI showing success while the write silently failed.
>
> ---
>
> **Versus RestAssured, from having used both:**
>
> RestAssured is a mature, expressive library and its given/when/then fluent syntax is genuinely nice to read. But it's a separate framework: separate project, separate test data helpers, separate CI job, separate reporting.
>
> Playwright's request context is less expressive for pure API work, and in exchange you get one framework, one set of fixtures, one report, and the ability to mix UI and API in a single test. For a team already running Playwright for UI, that consolidation usually wins. For a dedicated API-only suite in a Java shop, RestAssured is still a reasonable choice."

---

## The Two Request Modes

| | `request` fixture | `page.request` |
|---|---|---|
| **Cookies** | Own isolated jar | Shares the page's |
| **Auth state** | None unless you set it | Inherits browser session |
| **Use for** | Pure API tests, data seeding | Calls as the logged-in user |
| **Common mistake** | Expecting it to be authenticated | — |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "On the legacy stack I used RestAssured for API validation — separate framework, separate project from the Selenium UI tests. On the Playwright side, the request context means it's all one framework, and that consolidation matters more than it sounds. One set of fixtures, one report, one CI job.
>
> The way I use it most is data seeding. Rather than building preconditions by clicking through the UI, tests create what they need via API and navigate straight to the thing under test. That's the single biggest lever on suite speed, and it also means a UI test failure points at the UI rather than at some upstream creation screen that happened to break.
>
> Second use is pure API tests for anything without a UI dependency — validation rules, permissions. Those run in a fraction of the time and they're more stable, so pushing that work down out of the browser is the highest-leverage decision available.
>
> The gotcha worth knowing is the two request modes. The isolated `request` fixture doesn't share the browser's session, so if you use it expecting to be logged in, you get a 401 that looks like an auth bug and isn't."

---

## Common Interview Questions & Answers

### Q1: "Can Playwright test APIs?"

**A:** "Yes — built-in request context, no separate HTTP client needed. That's one of the reasons we chose it, because it means one framework for UI and API rather than maintaining Playwright plus something like RestAssured or Supertest alongside it."

### Q2: "What's the difference between `request` and `page.request`?"

**A:** "Isolation. The `request` fixture has its own cookie jar and knows nothing about any browser session. `page.request` shares cookies and auth with the page. I use the isolated one for pure API tests and seeding, and `page.request` when the call needs to be made as the logged-in user. Mixing them up is a common source of unexplained 401s."

### Q3: "How does it compare to RestAssured?"

**A:** "RestAssured is more expressive for pure API work — the fluent syntax is genuinely nicer to read. But it's a separate framework with separate data helpers, CI, and reporting. Playwright's request context trades some expressiveness for consolidation: one framework, one report, and you can mix UI and API in a single test. Having used both, I'd pick Playwright's for a team already running it for UI, and RestAssured for a dedicated API suite in a Java shop."

### Q4: "Why not test everything through the UI?"

**A:** "Cost and stability. A UI test costs roughly fifty times an API test on every run, forever, and it's more fragile. Validation rules, permission logic, calculations — none of that needs a browser. I reserve UI tests for what genuinely requires one: rendering, interaction, and the critical end-to-end journeys."

### Q5: "How do you handle authentication for API tests?"

**A:** "Depends on what I'm testing. For pure API tests, get a token once and set it as an `Authorization` header on the request context — usually in a fixture so it's reused. For anything that needs to behave as the browser user, `page.request` inherits the session automatically. And auth itself gets its own explicit tests, since storage state means most tests skip the login flow entirely."

---

## Weak Areas to Watch

❌ **Don't say Playwright needs a separate HTTP library**
→ It's built in. This is a basic knowledge check.

❌ **Don't confuse the two request modes**
→ It's the depth question, and the 401 confusion is the practical consequence.

❌ **Don't dismiss RestAssured**
→ You've used it. A balanced comparison is more credible than dismissal.

❌ **Don't treat API testing as only for API validation**
→ Its highest-value use in a UI suite is data seeding.

---

## Key Takeaway for Interview

The built-in request context means one framework instead of two — and its biggest practical value in a UI suite isn't API validation, it's **fast data seeding**.

The detail that shows real usage:

> **"The isolated `request` fixture doesn't share the browser's session. Use it expecting to be logged in and you get a 401 that looks like an auth bug and isn't."**

And the strategic one:

> **"A UI test costs about fifty times an API test on every run. Choosing the right level does more for suite speed and stability than any optimization afterwards."**

---

## Checklist

✅ Know both request modes and when each applies
✅ `baseURL` and shared headers configured centrally
✅ Auth token acquired once in a fixture, not per test
✅ API used for data seeding, not just validation
✅ Backend state verified after UI writes
✅ Non-UI logic pushed to API-level tests

---

**Remember:** The best use of API testing in a UI suite is getting to the thing under test faster. 🎯
