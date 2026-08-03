# Playwright - Network Mocking: Intercepting & Mocking API Requests

## What is Network Mocking?

Network mocking is intercepting browser network requests and providing mock responses instead of hitting real servers.

Basic idea:
```
Test says: POST /api/users
Playwright intercepts: "I'll handle this"
Test gets: Mock response (no server needed)
Result: Faster, more deterministic tests
```

---

## Why It Matters (Interview Context)

Interviewers ask about network mocking because:
1. **Shows testing independence** (tests don't depend on backend being up)
2. **Demonstrates test speed** (mock responses are instant)
3. **Proves reliability** (mock data is deterministic, no flakiness)
4. **Explains isolation** (frontend tests, backend unchanged)

Network mocking is essential for fast, stable test suites.

---

## Your 30-Second Interview Answer

> "Network mocking intercepts HTTP requests and provides mock responses. At NielsenIQ, we mock external APIs and heavy backend calls. Tests don't wait for servers, don't depend on external state. Page.route() intercepts matching requests, route.fulfill() provides response. Result: 10x faster tests, zero external dependencies."

---

## Your 2-Minute Deep Answer

> "Network mocking lets you control what the app sees, not what's actually running.
>
> **The Problem Without Mocking:**
>
> ```typescript
> // ❌ Test depends on real server
> test('User creates payment', async ({ page }) => {
>   await page.goto('/payment');
>   await page.getByTestId('card').fill('4242 4242 4242 4242');
>   await page.getByRole('button', { name: 'Submit' }).click();
>
>   // Wait for real payment API (3-5 seconds!)
>   // If API is down, test fails
>   // If payment quota exceeded, test fails
>   // If network is slow, test is flaky
>
>   await expect(page.getByText('Payment successful')).toBeVisible();
> });
>
> // Problem: Test depends on external system
> // Slow, unreliable, not under your control
> ```
>
> **The Solution: Network Mocking**
>
> ```typescript
> // ✅ Test mocks the API
> test('User creates payment', async ({ page }) => {
>   // Intercept POST /api/payments
>   await page.route('**/api/payments', async (route) => {
>     // Mock response (instant, reliable)
>     await route.fulfill({
>       status: 200,
>       body: JSON.stringify({ success: true, orderId: '12345' }),
>     });
>   });
>
>   await page.goto('/payment');
>   await page.getByTestId('card').fill('4242 4242 4242 4242');
>   await page.getByRole('button', { name: 'Submit' }).click();
>
>   // Mock response returns instantly
>   // No network latency, no external dependencies
>
>   await expect(page.getByText('Payment successful')).toBeVisible();
> });
>
> // Result: Fast, reliable, under your control
> ```
>
> **Network Mocking Patterns:**
>
> **1. ROUTE & FULFILL (Provide Mock Response)**
>
> ```typescript
> await page.route('**/api/users', async (route) => {
>   await route.fulfill({
>     status: 200,
>     contentType: 'application/json',
>     body: JSON.stringify({
>       id: 1,
>       name: 'John Doe',
>       email: 'john@example.com'
>     }),
>   });
> });
>
> // GET /api/users returns mocked data
> await page.goto('/profile');
> await expect(page.getByText('John Doe')).toBeVisible();
> ```
>
> **2. ROUTE & ABORT (Simulate Network Error)**
>
> ```typescript
> await page.route('**/api/payment', (route) => {
>   // Simulate network failure
>   route.abort('failed');
> });
>
> // Test handles network error gracefully
> await page.getByRole('button', { name: 'Submit' }).click();
> await expect(page.getByText('Network error')).toBeVisible();
> ```
>
> **3. ROUTE & CONTINUE (Pass Through to Real Server)**
>
> ```typescript
> await page.route('**/api/users', async (route) => {
>   // Intercept but pass through (useful for logging)
>   console.log('Request:', route.request().method(), route.request().url());
>   await route.continue();
> });
>
> // Request goes to real server, logged before/after
> ```
>
> **4. ROUTE WITH CONDITIONS**
>
> ```typescript
> await page.route('**/api/payment', async (route) => {
>   if (route.request().method() === 'POST') {
>     // Mock POST requests
>     await route.fulfill({ status: 200, body: '{}' });
>   } else {
>     // Pass through GET requests
>     await route.continue();
>   }
> });
> ```
>
> **5. DATA-DRIVEN MOCKING (Different Responses)**
>
> ```typescript
> const mockResponses = {
>   'success': { status: 200, payment: { id: '123' } },
>   'error': { status: 400, message: 'Invalid card' },
>   'timeout': { status: 504 },
> };
>
> test('User payment succeeds', async ({ page }) => {
>   await mockPaymentAPI(page, 'success');
>   // ... test success case
> });
>
> test('User payment fails', async ({ page }) => {
>   await mockPaymentAPI(page, 'error');
>   // ... test error handling
> });
> ```
>
> **At NielsenIQ:**
>
> ```typescript
> // fixtures/api.fixture.ts
> export const withMockedAPI = base.extend({
>   page: async ({ page }, use) => {
>     // Mock external payment API
>     await page.route('**/external-payment-api/**', async (route) => {
>       await route.fulfill({
>         status: 200,
>         body: JSON.stringify({ success: true }),
>       });
>     });
>
>     // Mock analytics API
>     await page.route('**/analytics/**', async (route) => {
>       await route.abort('failed');  // Analytics failures don't break tests
>     });
>
>     // Real backend still works, external APIs mocked
>     await use(page);
>   },
> });
>
> // tests/payments/create-payment.spec.ts
> test('User creates payment with mocked external API', async ({ page }) => {
>   // External API is mocked, backend is real
>   // Payment processing is instant (no external API latency)
>   // Test doesn't depend on external service status
> });
>
> // Result:
> // - ✅ Fast (no network latency to external APIs)
> // - ✅ Deterministic (mocked responses always same)
> // - ✅ Independent (external services can be down)
> // - ✅ Realistic (backend still works)
> ```
>
> **When to Mock vs When to Hit Real API:**
>
> ❌ Don't mock: Your backend (test it for real)
> ✅ Mock: External APIs (payment processors, analytics)
> ✅ Mock: Network errors (test error handling)
> ✅ Mock: Slow endpoints (speed up tests)
> ✅ Mock: Third-party services (they might be down)"

---

## Network Mocking Patterns

| Pattern | Use Case | Example |
|---------|----------|---------|
| **fulfill** | Provide mock response | Mock payment API |
| **abort** | Simulate network error | Test error handling |
| **continue** | Pass through (with logging) | Monitor requests |
| **fallback** | Mock specific, pass through others | Mock external, real backend |

---

## Your Real Project Example (NielsenIQ)

**fixtures/api.fixture.ts:**
```typescript
export const withMockedExternalAPIs = base.extend({
  page: async ({ page }, use) => {
    // Mock payment processor (external)
    await page.route('**/stripe.com/**', async (route) => {
      await route.fulfill({
        status: 200,
        body: JSON.stringify({ success: true, transactionId: 'tx_123' }),
      });
    });

    // Mock analytics (external)
    await page.route('**/analytics.example.com/**', async (route) => {
      await route.abort('failed');  // Analytics failures are acceptable
    });

    // Real backend still works (not mocked)
    // This allows testing real backend + mocked external services

    await use(page);
  },
});
```

**tests/payments/create-payment.spec.ts:**
```typescript
test('User creates payment', async ({ page }) => {
  // External APIs mocked, no latency
  // Backend API runs for real, tests business logic
  
  await page.goto('/payment');
  await page.getByTestId('card').fill('4242 4242 4242 4242');
  await page.getByRole('button', { name: 'Submit' }).click();
  
  // Mock Stripe API responds instantly
  // Real backend processes payment
  // Test completes in <1 second
  
  await expect(page.getByText('Payment confirmed')).toBeVisible();
});

test('User sees error if payment fails', async ({ page }) => {
  // Mock payment API returns error
  await page.route('**/stripe.com/**', async (route) => {
    await route.fulfill({
      status: 400,
      body: JSON.stringify({ error: 'Card declined' }),
    });
  });

  await page.goto('/payment');
  // ... attempt payment
  await expect(page.getByText('Card declined')).toBeVisible();
});
```

**Result:**
- External APIs mocked (fast, reliable)
- Backend tested for real (realistic)
- No dependencies on external services
- Tests run 10x faster

---

## Common Interview Questions & Answers

### Q1: "Should you mock the backend or external APIs?"

**A:** "Mock external APIs (payment processors, analytics, third-party). Don't mock your backend - test it for real. This validates your backend works correctly. External APIs are out of your control, so mock them for reliability."

### Q2: "How do you mock different responses in the same test?"

**A:** "Define multiple routes or use conditional logic. For example: Mock payment success route, test payment succeeds. Then remove that route, add payment failure route, test failure handling."

### Q3: "What if you need to test backend error handling?"

**A:** "Mock your backend endpoints to return errors. For example: mock GET /api/users to return 500, test if UI handles server error gracefully."

### Q4: "Does mocking slow down tests?"

**A:** "No, it speeds them up. Real API calls take 1-5 seconds. Mock responses take milliseconds. Mocking removes network latency."

### Q5: "How do you test with mocked API and real database?"

**A:** "Mock the API boundary. Mocked API calls come back to your backend which hits the real database. Best of both worlds: API is predictable, database is real."

---

## What You Should Be Ready to Explain

✅ Why network mocking improves test speed  
✅ Route patterns (fulfill, abort, continue)  
✅ When to mock vs test for real  
✅ Mocking external vs internal APIs  
✅ Error scenario testing (simulate failures)  
✅ Your mocking strategy at NielsenIQ  

---

## Weak Areas to Watch

❌ **Don't mock:** Your own backend  
→ **Reality:** Test backend for real

❌ **Don't forget:** Clear routes between tests  
→ **Reality:** Routes persist, clear if test-specific

❌ **Don't mock:** Everything (turns tests into stubs)  
→ **Reality:** Mock external, test internal

❌ **Don't ignore:** Error scenarios  
→ **Reality:** Mock failures to test error handling

---

## Key Takeaway for Interview

Network mocking is how you make tests fast and reliable. Without it, tests are slow, flaky, dependent on external services.

Your mocking strategy shows if you understand the boundary between your code and external systems.

**At NielsenIQ:** External APIs mocked, backend tested for real. That's the right balance.

**Your competitive edge:** You know what to mock and what to test. That's architectural thinking. ✨

---

**Remember:** Mock external, test internal. 🎯
