# Playwright API Testing - Auth Token Management

## What it is

API auth token management means how the test framework gets, stores, reuses, refreshes, and
protects tokens used for API requests.

For API tests, the common pattern is:

1. get token from login/auth endpoint
2. keep token in memory or fixture for the run
3. attach token as `Authorization: Bearer <token>`
4. refresh token if expired
5. keep credentials in env/secret store, not in code

## Why it matters

This came up in the FULL Creative interview because it separates UI auth knowledge from API
auth knowledge.

Storage state is a good UI strategy because it stores browser cookies/localStorage. API token
management is usually different: you often put the bearer token in request headers.

The mature answer is:

> "For UI tests I use storage state. For pure API tests I use a request context with bearer
> token headers, usually created through a fixture. If I need an API call as the current
> browser user, I use `page.request` because it shares the page session."

## 30-second answer

> "For API tests, I do not hardcode tokens or authenticate in every test. I get the token in a
> fixture or setup helper, store it in memory for the run, pass it through the Authorization
> header on the request context, and refresh it when expired or when a 401 indicates expiry.
> Credentials come from environment variables or CI secrets. For browser-authenticated API
> calls, I use `page.request` because it shares the page's session."

## 2-minute answer

> "I separate UI auth and API auth.
>
> For UI tests, storage state is useful because it saves cookies and localStorage after login,
> and later browser contexts can start already logged in.
>
> For pure API tests, I prefer a token-based request context. A fixture logs in through the
> auth endpoint, extracts the access token and expiry if available, then creates an
> APIRequestContext with `Authorization: Bearer <token>`. Tests reuse that fixture instead of
> logging in again and again.
>
> If the token can expire during a long run, I either track expiry with a safety margin or
> refresh when a 401 indicates token expiry. But I also keep explicit auth tests for missing
> token, expired token, invalid token, and insufficient permission, because reusing a token
> means most tests are not testing the auth flow itself."

## Playwright fixture example

```typescript
import { test as base, request as playwrightRequest, APIRequestContext } from '@playwright/test';

type Fixtures = {
  authApi: APIRequestContext;
};

export const test = base.extend<Fixtures>({
  authApi: async ({}, use) => {
    const loginContext = await playwrightRequest.newContext({
      baseURL: process.env.API_URL
    });

    const loginResponse = await loginContext.post('/auth/login', {
      data: {
        username: process.env.API_USERNAME,
        password: process.env.API_PASSWORD
      }
    });

    const loginBody = await loginResponse.json();
    const token = loginBody.accessToken;

    const authApi = await playwrightRequest.newContext({
      baseURL: process.env.API_URL,
      extraHTTPHeaders: {
        Authorization: `Bearer ${token}`
      }
    });

    await use(authApi);

    await authApi.dispose();
    await loginContext.dispose();
  }
});
```

Test usage:

```typescript
test('gets current user details', async ({ authApi }) => {
  const response = await authApi.get('/api/me');
  expect(response.status()).toBe(200);
});
```

## Expiry handling

If the token response gives expiry:

```typescript
const expiresAt = Date.now() + loginBody.expiresIn * 1000;
```

Refresh before it expires:

```typescript
const shouldRefresh = Date.now() > expiresAt - 60_000;
```

Interview line:

> "I avoid caching forever. Either track expiry with a safety margin, or refresh when the API
> returns 401 and the failure is clearly token expiry."

## `request` fixture vs `page.request`

Use `request` / custom API context:

- pure API tests
- API data setup
- bearer token header auth
- no browser session needed

Use `page.request`:

- API call should behave as currently logged-in browser user
- shares browser cookies/session
- useful after UI login or storage state

Key line:

> "Storage state is browser session management. Bearer token header is API request
> authentication. `page.request` is the bridge when the API call must share the browser user's
> session."

## What to test explicitly

Because most API tests reuse a valid token, add dedicated auth coverage:

- no token -> 401
- expired token -> 401
- invalid token -> 401
- valid token but wrong role -> 403
- valid token accessing another user's data -> 403 or 404 depending on API design

## Common interview Q&A

### Q1: "Where do you keep API token?"

**A:** "For tests, I keep the runtime token in memory inside a fixture/request context. I keep
credentials in environment variables or CI secrets. I do not commit token values to Git."

### Q2: "How do you attach token to requests?"

**A:** "Through the `Authorization` header, usually `Bearer <token>`, configured in
`extraHTTPHeaders` for an API request context."

### Q3: "How do you know token expired?"

**A:** "Best is to track expiry from the auth response with a safety margin. If that is not
available, a 401 from an otherwise valid request can trigger refresh logic, but the framework
should distinguish expiry from wrong credentials or permission problems."

### Q4: "Is storage state enough for API tests?"

**A:** "For pure API tests, no. Storage state is mainly browser cookies/localStorage. Pure API
tests usually use bearer token headers. But `page.request` can share the browser session when
the API call must be made as the logged-in page user."

### Q5: "What gap does token reuse create?"

**A:** "If every test reuses a valid token, then most tests are not testing login/auth itself.
So auth needs dedicated tests for valid, invalid, missing, expired, and wrong-role cases."

## Weak areas to watch

- Do not mix UI storage state and API bearer token as if they are the same thing.
- Do not hardcode tokens.
- Do not authenticate in every API test unless required.
- Do not cache token forever without expiry handling.
- Do not treat 401 and 403 as the same.

## Key takeaway

> "For UI, storage state gives a logged-in browser session. For API, bearer token in request
> headers is the usual clean approach. Use fixtures to acquire, reuse, refresh, and cleanly
> provide authenticated API contexts."
