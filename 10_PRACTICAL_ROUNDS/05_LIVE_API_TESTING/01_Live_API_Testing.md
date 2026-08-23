# Live API Testing — Approach

## What it is

The round where they give you an API — an endpoint, maybe a Swagger/Postman collection — and ask you to test it live, or design the tests for it. They want to see that you understand HTTP semantics, know what to validate beyond the status code, and can write clean API tests in either your Playwright `APIRequestContext` or RestAssured, depending on the stack.

---

## Why it matters

API testing is where a lot of real quality lives — it's faster, more stable, and closer to the business logic than UI tests. A senior SDET pushes as much coverage as possible down to this layer. This round checks whether you *actually* understand REST (not just "I send requests in Postman") and whether you validate the things that matter: status, body, schema, headers, and the negative/error paths that juniors skip.

You've done this for real on both stacks — Playwright's request library and RestAssured. This should be a confident round.

---

## The approach

**Step 1 — Understand the contract.** Method, endpoint, required vs optional params, auth, expected response shape and status codes. Ask for the schema or an example response if you don't have one.

**Step 2 — Enumerate what to validate** (say this out loud — it's the depth signal):
- **Status code** — the right one for the right case (200 vs 201 vs 204 vs 4xx).
- **Response body** — the actual values, not just "it returned something."
- **Schema** — structure, types, required fields present.
- **Headers** — content-type, auth-related, pagination.
- **Negative paths** — missing auth (401), forbidden (403), bad input (400), not found (404).
- **Idempotency & side effects** — did the POST actually create the record? Did a repeated PUT stay safe?

**Step 3 — Test the CRUD lifecycle, not just one call.** Create → read it back → update → verify → delete → confirm gone. This shows you test *state*, not isolated requests.

**Step 4 — Validate at the DB level too, where it matters.** *"After the POST returns 201, I'd also query the DB to confirm it actually persisted correctly"* — ties to your SQL strength.

---

## HTTP semantics they probe

| Method | Semantics | Idempotent? | Success code |
|--------|-----------|:-----------:|--------------|
| GET | Read, no side effects | Yes | 200 |
| POST | Create | No (creates each time) | 201 (or 200) |
| PUT | Replace whole resource | Yes | 200 / 204 |
| PATCH | Partial update | Usually yes | 200 |
| DELETE | Remove | Yes (2nd = already gone) | 204 / 200 |

**Status codes worth knowing cold:** 200 OK, 201 Created, 204 No Content, 400 Bad Request, **401 vs 403** (not authenticated vs authenticated-but-forbidden — a classic question), 404 Not Found, 409 Conflict, 429 Too Many Requests, 500 Server Error. See [HTTP Methods](../../02_PLAYWRIGHT/03_API_TESTING/02_HTTP_Methods.md).

---

## Worked example — Playwright APIRequestContext (TS)

```typescript
test('POST /orders creates an order', async ({ request }) => {
  const res = await request.post('/api/orders', {
    data: { productId: 42, quantity: 2 },
    headers: { Authorization: `Bearer ${token}` },
  });

  expect(res.status()).toBe(201);
  const body = await res.json();
  expect(body).toMatchObject({ productId: 42, quantity: 2, status: 'CREATED' });
  expect(body.id).toBeDefined();

  // read it back — verify it actually persisted
  const get = await request.get(`/api/orders/${body.id}`, {
    headers: { Authorization: `Bearer ${token}` },
  });
  expect(get.status()).toBe(200);
});
```
**Talk track:** *"`toMatchObject`, not `toEqual` — I assert the fields I care about without breaking when the API adds a new field. I read the resource back to prove it persisted, not just that the POST returned 201."* See [Response Validation](../../02_PLAYWRIGHT/03_API_TESTING/03_Response_Validation.md).

**The negative:**
```typescript
test('POST /orders without a token is rejected', async ({ request }) => {
  const res = await request.post('/api/orders', { data: { productId: 42, quantity: 2 } });
  expect(res.status()).toBe(401);
});
```

---

## Worked example — RestAssured (Java)

```java
@Test
public void createOrderReturns201() {
    given()
        .header("Authorization", "Bearer " + token)
        .contentType(ContentType.JSON)
        .body(new OrderRequest(42, 2))
    .when()
        .post("/api/orders")
    .then()
        .statusCode(201)
        .body("productId", equalTo(42))
        .body("status", equalTo("CREATED"))
        .body("id", notNullValue());
}
```
**Talk track:** given/when/then structure, POJO serialization for the body via Jackson (`@JsonIgnoreProperties(ignoreUnknown = true)` so a new response field doesn't break deserialization), token cached across tests rather than re-fetched each time. See [POJO Serialization](../../03_SELENIUM_JAVA/04_RESTASSURED_API/02_POJO_Serialization.md) and [Auth & Validation](../../03_SELENIUM_JAVA/04_RESTASSURED_API/03_Auth_And_Validation.md).

---

## Schema validation (the senior touch)

Beyond field-by-field asserts, validate the whole *shape*:
- **Playwright/TS:** a JSON-schema validator (e.g. Ajv) or a type-guard against a defined interface.
- **RestAssured:** `matchesJsonSchemaInClasspath("order-schema.json")`.

*"Field asserts catch value bugs; schema validation catches structural drift — a field disappearing or changing type — which is exactly the contract break that breaks consumers."* That's the point most candidates miss.

---

## Common interview Q&A

**Q: "What do you validate beyond the status code?"**
Body values, schema/structure, headers, and the negative paths. Then the DB-persistence check. The status code alone is the junior answer.

**Q: "401 vs 403?"**
401 = not authenticated (who are you?). 403 = authenticated but not allowed (I know who you are, you can't do this). Common gotcha.

**Q: "How do you test a POST is idempotent-safe / doesn't double-create?"**
POST isn't idempotent by design — but I'd test that a retry with an idempotency key doesn't create two records, and that PUT/DELETE *are* idempotent (second DELETE returns 404 or 204, not an error).

**Q: "How do you handle auth in the suite?"**
Fetch the token once in setup, cache it, reuse across tests — don't re-authenticate per request. Refresh on 401.

**Q: "UI test or API test for this validation?"**
API level almost always — faster, stabler, closer to the logic. Keep UI tests for what only the UI can verify (rendering, wiring). The trade-off answer.

---

## Weak areas to watch

- **Don't stop at the status code.** Validating only `status == 200` is the single most common junior mistake. Assert the body and schema.
- **Read the resource back** after a create/update — proving persistence beats trusting the response.
- **`toMatchObject` over `toEqual`** (and partial body matchers in RestAssured) — exact-match assertions break every time the API adds a field.
- **Cover the negative paths** — 401/403/400/404. A suite that only tests happy paths misses where APIs actually break.
- **Know your idempotency and 401-vs-403 cold** — these are the two most-asked REST gotchas.
- **Mention DB validation** where it fits — it's your differentiator and shows you test the whole path.

---

## Key takeaway

> **Validate status *and* body *and* schema *and* the negative paths — then read the resource back to prove it persisted.** API testing is where a senior SDET moves coverage for speed and stability, so this round rewards understanding REST semantics (idempotency, 401 vs 403) and validating the contract, not just the status code. Push logic to this layer and say why.
