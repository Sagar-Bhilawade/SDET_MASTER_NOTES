# Playwright - Response Validation: Asserting on API Responses

## What This Covers

What to check when an API responds — and how to write assertions that catch real problems without breaking every time the API adds a field.

---

## Why It Matters (Interview Context)

The question underneath this topic is: **do you assert meaningfully, or just check for 200?**

A test that only asserts `status === 200` passes while the API returns garbage. It's counted in your coverage number and catches nothing — which is worse than not having it, because it buys false confidence.

The senior signal is knowing what to assert *and* what deliberately not to, so tests stay stable as the API evolves.

---

## Your 30-Second Interview Answer

> "I assert on three layers: status code, response shape, and the specific values I care about. What I deliberately avoid is asserting on the entire response body — that breaks every time someone adds a field, which is a backwards-compatible change that shouldn't fail a test. So I check the contract and the fields under test, not everything the endpoint happens to return."

---

## Your 2-Minute Deep Answer

> "Response validation has layers, and choosing the right layer is what keeps tests both meaningful and stable.
>
> **Layer 1: Status and basic checks**
> ```typescript
> const response = await request.get('/api/users/1');
>
> expect(response.status()).toBe(200);
> expect(response.ok()).toBeTruthy();          // any 2xx
> ```
>
> Necessary but nowhere near sufficient. A test that stops here passes while the body is empty or wrong.
>
> **Layer 2: Headers**
> ```typescript
> expect(response.headers()['content-type']).toContain('application/json');
> ```
>
> Worth checking on things like content type, caching headers, and security headers. An API that silently switches from JSON to an HTML error page is a real failure mode — status might still be 200 from a proxy.
>
> **Layer 3: Body values**
> ```typescript
> const body = await response.json();
>
> expect(body.email).toBe('test@example.com');
> expect(body.role).toBe('admin');
> expect(body.id).toBeDefined();
> ```
>
> Assert the fields your test is actually about.
>
> **Layer 4: Shape and types**
>
> This is the one people skip, and it's where contract regressions get caught:
>
> ```typescript
> expect(body).toMatchObject({
>   id: expect.any(Number),
>   email: expect.any(String),
>   createdAt: expect.any(String),
> });
> ```
>
> `toMatchObject` is the key choice — it checks the fields you name and ignores everything else. So the test survives the API adding a field, which is a backwards-compatible change, but fails if a field disappears or changes type. That's exactly the sensitivity you want.
>
> Compare with `toEqual`, which requires an exact match. Use that and every additive API change breaks your suite, and people learn to update assertions reflexively without reading them — which is how assertions get weakened into meaninglessness.
>
> ---
>
> **Handling dynamic values:**
>
> IDs, timestamps, and tokens change every run. Don't assert on the value; assert on the shape.
>
> ```typescript
> expect(body.id).toEqual(expect.any(Number));
> expect(body.createdAt).toMatch(/^\d{4}-\d{2}-\d{2}/);
> expect(new Date(body.createdAt).getTime()).toBeLessThanOrEqual(Date.now());
> ```
>
> That last one is a nice check — the timestamp should exist, parse, and not be in the future.
>
> ---
>
> **Arrays and collections:**
>
> ```typescript
> const body = await response.json();
>
> expect(Array.isArray(body.items)).toBeTruthy();
> expect(body.items.length).toBeGreaterThan(0);
> expect(body.items[0]).toMatchObject({ id: expect.any(Number), name: expect.any(String) });
> ```
>
> **Careful with ordering.** Asserting `body.items[0].name === 'Alpha'` assumes the API returns a stable order. Many don't guarantee it, and then you have a flaky test that looks like an API bug. Either assert the API's documented ordering deliberately, or assert on containment:
>
> ```typescript
> expect(body.items.map(i => i.name)).toContain('Alpha');
> ```
>
> ---
>
> **Error responses deserve real assertions too:**
>
> ```typescript
> const response = await request.post('/api/users', { data: { email: 'not-an-email' } });
>
> expect(response.status()).toBe(400);
> const error = await response.json();
> expect(error.message).toBeTruthy();          // there IS a message
> expect(error.field).toBe('email');           // it identifies the problem
> ```
>
> An error that returns 400 with an empty body is technically correct and practically useless — the UI has nothing to show the user. Asserting that error responses are *actionable* catches a real category of problem.
>
> ---
>
> **Schema validation for critical contracts:**
>
> For endpoints where the contract really matters, a schema library like Zod gives stronger guarantees than hand-written field checks:
>
> ```typescript
> const UserSchema = z.object({
>   id: z.number(),
>   email: z.string().email(),
>   role: z.enum(['admin', 'user', 'viewer']),
> });
>
> expect(() => UserSchema.parse(body)).not.toThrow();
> ```
>
> This catches type drift and invalid enum values that field-by-field assertions miss. I'd use it on core contracts rather than everywhere — it's more setup, so it should earn its place."

---

## What to Assert

| Layer | Check | Always? |
|-------|-------|:---:|
| Status | Correct code, not just 2xx | ✅ |
| Headers | Content-type, security headers | ⚠️ Key ones |
| Values | The fields under test | ✅ |
| Shape | `toMatchObject` with types | ✅ |
| Full body | `toEqual` exact match | ❌ Too brittle |
| Schema | Zod / JSON Schema | ⚠️ Critical contracts |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "The principle I hold to is asserting the contract, not the payload. Using `toMatchObject` rather than `toEqual` means a test checks the fields it cares about and ignores the rest — so the API adding a field doesn't break anything, because that's a backwards-compatible change, but a field disappearing or changing type does fail.
>
> That distinction matters more than it sounds. With exact-match assertions, every additive API change breaks a batch of tests, and people start updating assertions reflexively without reading them. That's how assertions get quietly weakened until they're checking nothing — and the suite stays green while catching less and less.
>
> The other thing I'm deliberate about is asserting error responses properly. A 400 with an empty body is technically correct and useless, because the UI has nothing to show the user. Checking that errors are actionable catches a real category of problem that a status-code-only assertion misses entirely.
>
> Coming from RestAssured, the discipline transfers directly — the syntax differs, the question of what's worth asserting doesn't."

---

## Common Interview Questions & Answers

### Q1: "What do you assert on an API response?"

**A:** "Status code, the specific fields my test is about, and the shape with types. What I avoid is asserting the full body exactly — that breaks whenever someone adds a field, which is backwards-compatible and shouldn't fail a test. `toMatchObject` gives the right sensitivity: catches removals and type changes, ignores additions."

### Q2: "How do you handle dynamic values like IDs and timestamps?"

**A:** "Assert the shape rather than the value. `expect.any(Number)` for IDs, a regex or date-parse check for timestamps. For timestamps I'll often also check it's not in the future, which catches timezone and clock bugs that a pure format check misses."

### Q3: "A test asserts 200 and passes, but the feature is broken. What went wrong?"

**A:** "The assertion isn't doing any real work — it's checking the request completed, not that the response is correct. That's the most common form of test debt I look for in review. Green with a weak assertion is more dangerous than red, because red gets fixed and green gets trusted."

### Q4: "When would you use schema validation?"

**A:** "For core contracts where drift would be expensive — the endpoints other teams or the frontend depend on. A schema catches type changes and invalid enum values that field-by-field assertions miss. I wouldn't apply it everywhere; it's more setup, so it should earn its place on the endpoints that matter."

### Q5: "How do you test a list endpoint?"

**A:** "Type and non-emptiness first, then shape on the elements. The thing to be careful about is ordering — asserting on `items[0]` assumes a stable sort, and many APIs don't guarantee one. That produces a flaky test that looks like an API bug. Unless ordering is part of the documented contract, I'd assert on containment instead."

---

## Weak Areas to Watch

❌ **Don't stop at `status === 200`**
→ Passes while the body is wrong. This is the classic weak assertion.

❌ **Don't use `toEqual` on full bodies**
→ Breaks on every additive change and trains people to weaken assertions.

❌ **Don't assert exact dynamic values**
→ IDs and timestamps change. Assert shape.

❌ **Don't assume array ordering**
→ Unless it's a documented contract, that's a flake waiting to happen.

❌ **Don't skip error response assertions**
→ An unhelpful error is a real bug the UI can't work around.

---

## Key Takeaway for Interview

Assert the **contract**, not the payload.

The distinction that shows judgement:

> **"`toMatchObject` over `toEqual`. Adding a field is backwards-compatible and shouldn't fail a test — removing one or changing its type should. Exact-match assertions break on every additive change, and that trains people to update assertions without reading them."**

And the one that connects to test quality generally:

> **"A test that only asserts 200 is counted in your coverage and catches nothing. Green with a weak assertion is worse than red."**

---

## Checklist

✅ Specific status code asserted, not just `ok()`
✅ `toMatchObject` for shape, not `toEqual`
✅ Types asserted with `expect.any()`
✅ Dynamic values checked by shape, not value
✅ Array ordering not assumed
✅ Error responses asserted as actionable
✅ Schema validation on critical contracts
✅ Every assertion able to actually fail

---

**Remember:** An assertion that can never fail is a comment with extra steps. 🎯
