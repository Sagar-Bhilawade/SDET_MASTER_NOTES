# RestAssured - Authentication & Response Validation

## What This Covers

Handling auth flows, validating responses properly, and JSON schema validation.

---

## Why It Matters (Interview Context)

Auth is where API test frameworks get messy — token acquisition per test, hardcoded credentials, expiry not handled. How you structure it shows framework thinking.

Validation is where the "do you assert meaningfully" question shows up, same as everywhere. Schema validation specifically is a capability plenty of people don't know RestAssured has.

---

## Your 30-Second Interview Answer

> "Token acquired once in setup and reused rather than authenticating per test — same principle as storage state in Playwright. For validation I assert status, the specific fields under test, and the response schema. Schema validation is the underused one: a JSON schema file catches type changes and missing fields across the whole contract in a single assertion, which field-by-field checks miss."

---

## Your 2-Minute Deep Answer

> "**Authentication approaches:**
>
> ```java
> given().auth().basic("user", "pass")
> given().auth().preemptive().basic("user", "pass")     // sends without waiting for 401
> given().auth().oauth2(accessToken)
> given().header("Authorization", "Bearer " + token)
> ```
>
> `preemptive()` matters — without it, RestAssured waits for a 401 challenge before sending credentials, which is an extra round trip and doesn't work with every server.
>
> **Token acquisition, structured properly:**
>
> ```java
> public class AuthManager {
>     private static String token;
>     private static Instant expiry;
>
>     public static synchronized String getToken() {
>         if (token == null || Instant.now().isAfter(expiry)) {
>             Response response = given()
>                 .contentType(ContentType.JSON)
>                 .body(new LoginRequest(
>                     ConfigReader.get("api.user"),
>                     ConfigReader.get("api.password")))
>             .when()
>                 .post(ConfigReader.get("api.base.url") + "/auth/login")
>             .then()
>                 .statusCode(200)
>                 .extract().response();
>
>             token = response.jsonPath().getString("accessToken");
>             expiry = Instant.now().plusSeconds(response.jsonPath().getInt("expiresIn") - 60);
>         }
>         return token;
>     }
> }
> ```
>
> Three deliberate things there:
>
> **Cached with expiry** — authenticating per test is wasted time, exactly like logging in per test in a UI suite. But caching forever means a long suite dies mid-run when the token expires. Tracking expiry with a safety margin handles both.
>
> **`synchronized`** — under parallel execution, several threads could hit this simultaneously and each trigger a login. Worth guarding.
>
> **Credentials from config**, never hardcoded. That's a security issue, not just hygiene — the repo is readable by everyone with access.
>
> ---
>
> **Testing auth itself:**
>
> Because most tests reuse a token, the auth flow needs its own explicit coverage — same gap as storage state creates in a UI suite:
>
> ```java
> // Valid credentials
> // Invalid password → 401
> // Expired token → 401
> // Missing token → 401
> // Valid token, insufficient role → 403
> ```
>
> **The 401 versus 403 distinction is worth testing deliberately.** 401 means not authenticated; 403 means authenticated but not permitted. APIs get this wrong regularly, and returning 401 for a permissions problem sends the client into a pointless re-authentication loop.
>
> ---
>
> **Response validation — the layers:**
>
> ```java
> Response response = given().spec(authSpec()).when().get("/users/1")
>     .then().extract().response();
>
> // Status
> assertThat(response.statusCode()).isEqualTo(200);
>
> // Headers
> assertThat(response.contentType()).contains("application/json");
>
> // Fields under test
> assertThat(response.jsonPath().getString("email")).isEqualTo("test@example.com");
>
> // Types, not values, for dynamic fields
> assertThat(response.jsonPath().getInt("id")).isPositive();
> assertThat(response.jsonPath().getString("createdAt")).matches("\\d{4}-\\d{2}-\\d{2}.*");
>
> // Response time
> assertThat(response.time()).isLessThan(2000L);
> ```
>
> Same principle as everywhere: assert what the test is about, plus the contract. Don't assert the entire body — an added field is backwards-compatible and shouldn't fail anything.
>
> ---
>
> **JSON Schema validation — the underused capability:**
>
> ```java
> given().spec(authSpec())
> .when().get("/users/1")
> .then().assertThat()
>     .body(matchesJsonSchemaInClasspath("schemas/user-schema.json"));
> ```
>
> ```json
> {
>   "type": "object",
>   "required": ["id", "email", "role"],
>   "properties": {
>     "id":    { "type": "integer" },
>     "email": { "type": "string", "format": "email" },
>     "role":  { "type": "string", "enum": ["admin", "user", "viewer"] }
>   }
> }
> ```
>
> One assertion validates the whole contract — required fields present, types correct, enums valid. Field-by-field assertions catch the fields you thought to check; a schema catches the ones you didn't.
>
> It's particularly valuable for contract regression: if the backend changes `id` from integer to string, every hand-written assertion on other fields still passes and only the schema catches it.
>
> I'd apply it to core contracts rather than everywhere — it's another artifact to maintain, so it should earn its place.
>
> ---
>
> **Error response validation:**
>
> ```java
> Response response = given().spec(authSpec())
>     .body(new User("not-an-email", "admin"))
> .when().post("/users")
> .then().extract().response();
>
> assertThat(response.statusCode()).isEqualTo(400);
> assertThat(response.jsonPath().getString("message")).isNotBlank();
> assertThat(response.jsonPath().getString("field")).isEqualTo("email");
> ```
>
> A 400 with an empty body is technically correct and useless — the UI has nothing to show the user. Asserting that errors are *actionable* catches a real class of problem, and it's the kind of thing only API-level testing finds."

---

## Auth Reference

| Method | Use |
|--------|-----|
| `.auth().basic()` | Basic auth (waits for 401 challenge) |
| `.auth().preemptive().basic()` | Basic auth, sent immediately |
| `.auth().oauth2(token)` | OAuth2 bearer |
| `.header("Authorization", ...)` | Manual — most common |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Token management was centralized in an auth manager — acquired once, cached with expiry tracking, and refreshed with a safety margin before it lapsed. Authenticating per test is wasted time, the same as logging in per test in a UI suite, but caching indefinitely means a long run dies partway through when the token expires. Tracking expiry handles both, and it needs to be synchronized under parallel execution or several threads each trigger their own login.
>
> Because nearly every test reuses that token, the auth flow itself needed dedicated coverage — that's the same blind-spot pattern storage state creates in Playwright. Invalid credentials, expired token, missing token, and specifically the 401 versus 403 distinction. APIs get that one wrong often, and returning 401 for a permissions problem sends clients into a pointless re-authentication loop.
>
> On validation, the capability I'd highlight is JSON schema. One assertion validates required fields, types, and enums across the whole contract. Hand-written assertions catch the fields you thought to check; a schema catches the ones you didn't. If the backend changed an ID from integer to string, every other assertion still passes and only the schema notices.
>
> And I'd assert error responses properly. A 400 with an empty body is technically correct and useless, because the UI has nothing to show. Checking that errors are actionable is something only API-level testing finds."

---

## Common Interview Questions & Answers

### Q1: "How do you handle authentication in API tests?"

**A:** "Acquire the token once and cache it with expiry tracking, refreshed with a safety margin. Per-test authentication is wasted time; caching forever means a long suite dies when the token lapses. It needs synchronizing under parallel execution, and credentials come from config, never hardcoded."

### Q2: "What's the difference between 401 and 403, and why test it?"

**A:** "401 is not authenticated; 403 is authenticated but not permitted. Worth testing deliberately because APIs get it wrong regularly, and returning 401 for a permissions problem makes the client retry authentication pointlessly — it can produce an infinite loop in a UI that auto-refreshes tokens."

### Q3: "What's JSON schema validation and when would you use it?"

**A:** "A schema file defining required fields, types, and allowed values, asserted in one line with `matchesJsonSchemaInClasspath`. It catches contract regressions that field-by-field assertions miss — if an ID changes from integer to string, your other assertions still pass and only the schema catches it. I'd apply it to core contracts rather than everywhere, since it's another artifact to maintain."

### Q4: "How do you validate a response without making tests brittle?"

**A:** "Assert the status, the fields the test is actually about, and the contract shape — not the entire body. An added field is backwards-compatible and shouldn't fail anything. For dynamic values like IDs and timestamps I assert type and format rather than the value."

### Q5: "If most tests reuse a cached token, what have you not tested?"

**A:** "The auth flow itself. That's the same gap storage state creates in a UI suite — you've optimized away coverage of the most-used path in the system. So login gets explicit dedicated tests: valid, invalid, expired, missing, and insufficient permissions."

---

## Weak Areas to Watch

❌ **Don't authenticate per test**
→ Wasted time, same as logging in per test.

❌ **Don't cache the token forever**
→ Long suites die mid-run on expiry.

❌ **Don't hardcode credentials**
→ Security issue; the repo is readable.

❌ **Don't skip testing the auth flow**
→ Token reuse creates a blind spot in the most-used path.

❌ **Don't accept an empty 400 body**
→ Technically correct, practically useless.

---

## Key Takeaway for Interview

Auth structure shows framework thinking; schema validation shows depth.

The token pattern:

> **"Acquired once, cached with expiry tracking and a safety margin, synchronized for parallel. Per-test auth is wasted time; caching forever means the suite dies mid-run."**

And the validation capability most people miss:

> **"Hand-written assertions catch the fields you thought to check. A JSON schema catches the ones you didn't — if an ID changes from integer to string, every other assertion still passes and only the schema notices."**

---

## Checklist

✅ Token acquired once, cached with expiry margin
✅ Token acquisition synchronized for parallel runs
✅ Credentials from config
✅ Auth flow covered by dedicated tests
✅ 401 vs 403 distinction tested
✅ Schema validation on core contracts
✅ Dynamic values asserted by type, not value
✅ Error responses asserted as actionable

---

**Remember:** Reusing a token everywhere means nothing is testing how tokens are issued. 🎯
