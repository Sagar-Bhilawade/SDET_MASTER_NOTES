# Playwright API Testing - Non-Functional API Checks

## What it is

Non-functional API testing checks **how well** the API behaves, not only whether the happy
path works.

Functional API test:

> "Does `POST /users` create a user?"

Non-functional API test:

> "Is the API secure, fast enough, stable under invalid input, and does it return useful
> errors without leaking sensitive information?"

## Why it matters

Interviewers ask this because many candidates only say status code and response body. A
strong SDET also thinks about security, performance, reliability, compatibility, and error
quality.

For APIs, non-functional defects can be expensive. An endpoint can return the correct data
but still be slow, insecure, unstable, or leak internal details.

## 30-second answer

> "For non-functional API testing, I look beyond happy-path status and body. I check response
> time, authentication and authorization behavior, error quality, schema stability, backward
> compatibility, rate limiting where applicable, sensitive-data leakage, and behavior with
> invalid or malformed input. I would not call functional API testing complete unless these
> risks are also considered."

## 2-minute answer

> "My API checks usually start with functional validation: method, status, response fields,
> business rule, and side effect. Then I add non-functional checks based on risk.
>
> Security and auth are first: missing token should be 401, valid token with wrong permission
> should be 403, expired token should not work, and changing a resource ID should not expose
> another user's data.
>
> Then error handling: invalid input should return a clear 4xx response, not a 500, and the
> error body should be useful for client/UI handling without exposing stack traces or internal
> details.
>
> Then performance and reliability: response time should stay within an agreed threshold for
> important endpoints, and the endpoint should behave consistently for duplicate requests,
> boundary input, and malformed payloads.
>
> Finally schema and compatibility: fields expected by UI should not silently disappear or
> change type without contract awareness."

## Non-functional API checklist

| Area | What to test | Example |
|---|---|---|
| Performance | Response time within threshold | p95/response-time budget if defined |
| Security | No token, expired token, wrong role | 401 vs 403 |
| Authorization | User cannot access another user's resource | IDOR check |
| Error quality | Clear 4xx response for bad input | no stack trace, useful message |
| Schema stability | Expected fields and types remain stable | string stays string, number stays number |
| Sensitive data | No password/token/internal info leaked | response body/header check |
| Rate limiting | Too many requests handled safely | 429 if supported |
| Idempotency | Repeated safe requests behave correctly | GET no side effect, DELETE repeat behavior |
| Compatibility | Old clients not broken unexpectedly | optional/new fields handled |

## String passed for numeric field

Question:

> "If an API expects a number and we send a string, will it hit the server?"

Best answer:

> "Usually yes, the request reaches the server or API gateway. Then validation should reject
> it with a 400-level response, normally `400 Bad Request`, if the contract expects a number.
> But the exact behavior depends on where validation is implemented: gateway/schema validation,
> controller validation, or service-level validation."

Add depth:

> "What I would test is not only the status. I would check that it does not become a 500, does
> not silently coerce a bad value incorrectly, and returns a useful error body."

Example:

```typescript
test('rejects string for numeric age field', async ({ request }) => {
  const response = await request.post('/api/users', {
    data: {
      name: 'Test User',
      age: 'twenty'
    }
  });

  expect(response.status()).toBe(400);
  const body = await response.json();
  expect(body.message).toContain('age');
});
```

## Real project example

> "In API validation, I don't stop at whether the endpoint returns 200. I check auth behavior,
> invalid input, error response quality, and whether important fields are stable for the UI.
> For a numeric field receiving a string, my expectation is a controlled 400-level validation
> error, not a 500 and not silent wrong conversion."

## Common interview Q&A

### Q1: "What non-functional checks do you do for APIs?"

**A:** "Security/auth, response time, error quality, schema stability, sensitive data leakage,
rate limiting if applicable, and compatibility. I choose based on endpoint risk."

### Q2: "If numeric field receives string, will it hit server?"

**A:** "Usually yes, it reaches server or gateway, and validation should reject it with 400.
The important checks are: no 500, no silent wrong conversion, useful error message, and no
bad state written."

### Q3: "Is response time a functional or non-functional check?"

**A:** "Non-functional. The response can be functionally correct but still unacceptable if it
takes too long."

### Q4: "What security checks belong in API automation?"

**A:** "Missing token, expired token, wrong role, IDOR/resource access, and 401 vs 403. Deep
penetration testing is specialist work, but authorization checks absolutely belong in API
automation."

## Weak areas to watch

- Do not say only "status code and body".
- Do not guarantee 400 for every system; say "should return controlled 4xx, typically 400".
- Do not ignore error body quality.
- Do not call storage state an API token strategy; UI storage state and API bearer tokens are
  related to auth, but handled differently.

## Key takeaway

> "API testing is not complete with 200 and JSON body. Good API testing also checks security,
> validation, performance, error quality, and contract stability."
