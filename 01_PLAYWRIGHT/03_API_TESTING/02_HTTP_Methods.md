# Playwright - HTTP Methods: GET, POST, PUT, PATCH, DELETE

## What This Covers

Making each type of HTTP request in Playwright, and — more importantly for interviews — knowing what each method *should* do, so you can tell when an API is behaving incorrectly.

---

## Why It Matters (Interview Context)

Two layers here:

**Layer 1 (basic):** Can you make the calls? Everyone can.

**Layer 2 (what they're actually testing):** Do you understand REST semantics well enough to spot when an API is wrong? An interviewer asking "what's the difference between PUT and PATCH" isn't checking syntax — they're checking whether you'd notice an API that returns 200 for a failed operation, or a DELETE that isn't idempotent.

That second layer is where the API testing depth question lives.

---

## Your 30-Second Interview Answer

> "The methods map to operations — GET reads, POST creates, PUT replaces, PATCH partially updates, DELETE removes. What I actually test against is the semantics: GET should be safe and have no side effects, PUT and DELETE should be idempotent so calling them twice gives the same end state, and POST shouldn't be. Those properties are where real API bugs hide, and they're the things a UI test would never catch."

---

## Your 2-Minute Deep Answer

> "The syntax is straightforward — it's the semantics that matter for testing.
>
> **GET — read, no side effects**
> ```typescript
> const response = await request.get('/api/users/1');
> const response = await request.get('/api/users', {
>   params: { role: 'admin', page: 2 }
> });
> ```
> GET must be **safe**: calling it shouldn't change anything. A GET that mutates state is a genuine bug, and it's one I'd deliberately test for — call it twice, confirm nothing changed.
>
> **POST — create**
> ```typescript
> const response = await request.post('/api/users', {
>   data: { email: 'test@example.com', role: 'admin' }
> });
> expect(response.status()).toBe(201);
> ```
> Should return **201 Created**, not 200. And POST is deliberately *not* idempotent — two calls create two resources. That's correct behaviour, and it's why double-submit protection is a real concern worth testing.
>
> **PUT — full replacement**
> ```typescript
> await request.put('/api/users/1', {
>   data: { email: 'new@example.com', role: 'admin', name: 'Test' }
> });
> ```
> PUT replaces the **entire** resource. Omit a field and it should be cleared or reset — not silently preserved. That's a good test: PUT with a partial body, confirm the omitted fields actually got wiped. Plenty of APIs implement PUT as a partial update, which is a spec violation worth flagging.
>
> PUT must be **idempotent** — same request twice, same end state.
>
> **PATCH — partial update**
> ```typescript
> await request.patch('/api/users/1', {
>   data: { role: 'viewer' }        // only this field changes
> });
> ```
> Only touches what you send. Everything else stays.
>
> **DELETE — remove**
> ```typescript
> const response = await request.delete('/api/users/1');
> expect(response.status()).toBe(204);
> ```
> Usually **204 No Content**. Should be idempotent: deleting an already-deleted resource shouldn't error — 204 or 404 is fine, a 500 is not.
>
> ---
>
> **Request options that matter:**
>
> ```typescript
> await request.post('/api/users', {
>   data: { email: 'x@y.com' },              // JSON body
>   headers: { 'Authorization': `Bearer ${token}` },
>   params: { include: 'profile' },          // query string
>   timeout: 10000,
>   failOnStatusCode: false,                 // don't throw on 4xx/5xx
> });
> ```
>
> `failOnStatusCode: false` is the one worth knowing. By default Playwright doesn't throw on error statuses — but when you're deliberately testing error handling, being explicit makes the intent obvious to whoever reads the test next.
>
> **File uploads** use multipart:
> ```typescript
> await request.post('/api/upload', {
>   multipart: {
>     file: { name: 'data.csv', mimeType: 'text/csv', buffer: Buffer.from('a,b,c') },
>   },
> });
> ```
>
> ---
>
> **What I actually test, beyond the happy path:**
>
> - **Idempotency** — call PUT/DELETE twice, confirm the end state matches
> - **Correct status codes** — 201 for create, not 200; 204 for delete
> - **Validation** — bad payload returns 400 with a useful message, not 500
> - **Authorization** — the same call as a different role returns 403
> - **Not found** — 404, not an empty 200
>
> That last one is a common real bug: an API returns `200 OK` with an empty body for a missing resource. The UI shows a blank page instead of an error, and nobody notices until a user reports it. A UI test would never catch it — the page renders fine."

---

## Method Semantics Reference

| Method | Purpose | Success | Safe? | Idempotent? |
|--------|---------|:---:|:---:|:---:|
| **GET** | Read | 200 | ✅ | ✅ |
| **POST** | Create | 201 | ❌ | ❌ |
| **PUT** | Full replace | 200 / 204 | ❌ | ✅ |
| **PATCH** | Partial update | 200 | ❌ | ⚠️ Usually |
| **DELETE** | Remove | 204 | ❌ | ✅ |

**Safe** = no side effects. **Idempotent** = same result if repeated.

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "On the legacy stack I did heavy API validation with RestAssured — that was a large part of that role, backend validation alongside database checks. On Playwright it's the same discipline through the request context.
>
> What I focus on beyond the happy path is the semantic properties, because that's where real bugs hide and where UI tests are blind. Idempotency in particular: calling DELETE twice should be safe, and PUT twice should land in the same state. Those break more often than people expect, usually when someone adds a side effect like an audit log or a notification without thinking about repeat calls.
>
> The one I've found most valuable to check is status code correctness — specifically an API returning 200 with an empty body when a resource doesn't exist. The UI renders a blank page rather than an error, so a UI test passes happily. It only surfaces as a confused user. That's the kind of thing API-level testing catches and browser testing structurally can't."

---

## Common Interview Questions & Answers

### Q1: "What's the difference between PUT and PATCH?"

**A:** "PUT replaces the whole resource — omit a field and it should be cleared. PATCH updates only what you send. Worth testing, because plenty of APIs implement PUT as a partial update, which is a spec violation. I'd send a PUT with a partial body and confirm the omitted fields actually got wiped."

### Q2: "What does idempotent mean and why does it matter?"

**A:** "Calling it repeatedly produces the same end state. PUT and DELETE should be idempotent, POST shouldn't. It matters practically because networks retry — if a DELETE isn't idempotent, a retried request can throw a 500 or corrupt state. It's also where side effects sneak in: someone adds a notification to a DELETE handler and now the second call sends a duplicate email."

### Q3: "How would you test error handling?"

**A:** "Send deliberately bad input and assert on the response. Malformed payload should give 400 with a useful message, not 500. Unauthorized access should give 403, not 404 or a silent empty result. And missing resources should give 404, not 200 with an empty body — that last one is a common bug that UI tests can't catch, because the page just renders empty and looks fine."

### Q4: "How do you test file uploads?"

**A:** "The `multipart` option on the request. Beyond the happy path I'd check the limits — oversized files, wrong MIME types, empty files. Those are the cases that break in production, and they're much cheaper to test at API level than by driving a file picker in the browser."

### Q5: "Should you test every endpoint?"

**A:** "No — same prioritization as anywhere else: risk and repetition. Endpoints behind critical flows get thorough coverage including error paths. Low-risk read endpoints might just get a smoke check. Every test is a maintenance liability, so blanket coverage of an API surface costs more than it returns."

---

## Weak Areas to Watch

❌ **Don't just test happy paths**
→ Error handling and status codes are where API bugs live.

❌ **Don't accept 200 for everything**
→ 201 for create, 204 for delete. Wrong codes are real bugs.

❌ **Don't skip idempotency**
→ It's the semantic property most likely to be broken.

❌ **Don't forget authorization**
→ Same call, different role. This is a security check, not just a functional one.

---

## Key Takeaway for Interview

The syntax is trivial; the semantics are the interview.

The insight that demonstrates real API testing experience:

> **"An API returning 200 with an empty body for a missing resource is a common bug that UI tests structurally cannot catch — the page renders blank and looks fine. That's the case for testing at the API level."**

And the property most often broken:

> **"Idempotency. PUT and DELETE should be safe to repeat, and they break the moment someone adds a side effect like an audit log or a notification without considering retries."**

---

## Checklist

✅ Correct status codes asserted (201 create, 204 delete)
✅ Idempotency verified for PUT and DELETE
✅ GET confirmed to have no side effects
✅ Validation errors return 400 with useful messages
✅ Authorization tested across roles
✅ 404 returned for missing resources, not empty 200
✅ File upload limits and MIME types covered
✅ Coverage prioritized by risk, not by endpoint count

---

**Remember:** Status codes are an API's error messages. Wrong ones mislead every consumer downstream. 🎯
