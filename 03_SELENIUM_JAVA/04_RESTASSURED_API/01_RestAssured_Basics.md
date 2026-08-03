# RestAssured - Fundamentals & BDD Syntax

## What This Covers

RestAssured's given/when/then structure, request specification, and response extraction — the foundation of the API testing you did on the legacy stack.

---

## Why It Matters (Interview Context)

You did **heavy backend API validation** on Track 2. That's a strength worth leading with, because plenty of UI automation engineers have barely touched API testing.

Interviewers check whether you used RestAssured properly — specifications, reusable configuration, proper extraction — or whether you wrote the same six lines in every test.

---

## Your 30-Second Interview Answer

> "RestAssured uses a given/when/then structure — given sets up the request with headers, auth, body; when performs it; then validates. The thing that separates a real framework from copy-pasted tests is request and response specifications: you define base URI, headers, and auth once and reuse them, so changing the base URL is one edit. I'd extract the response and assert in code rather than chaining everything into `then()`, because the failure messages are far better."

---

## Your 2-Minute Deep Answer

> "**The BDD syntax:**
>
> ```java
> given()
>     .baseUri("https://api.example.com")
>     .header("Authorization", "Bearer " + token)
>     .contentType(ContentType.JSON)
>     .body(requestPayload)
> .when()
>     .post("/users")
> .then()
>     .statusCode(201)
>     .body("email", equalTo("test@example.com"));
> ```
>
> `given` builds the request, `when` executes, `then` validates. It reads well, and it's the syntax everyone shows in tutorials.
>
> ---
>
> **Request specifications — what makes it a framework:**
>
> ```java
> public static RequestSpecification baseSpec() {
>     return new RequestSpecBuilder()
>         .setBaseUri(ConfigReader.get("api.base.url"))
>         .setContentType(ContentType.JSON)
>         .addHeader("Accept", "application/json")
>         .addFilter(new RequestLoggingFilter())
>         .addFilter(new ResponseLoggingFilter())
>         .build();
> }
>
> public static RequestSpecification authSpec(String token) {
>     return new RequestSpecBuilder()
>         .addRequestSpecification(baseSpec())
>         .addHeader("Authorization", "Bearer " + token)
>         .build();
> }
> ```
>
> ```java
> given().spec(authSpec(token))
>     .body(payload)
> .when().post("/users");
> ```
>
> This is the difference between a framework and a pile of tests. Base URI, common headers, and logging defined once. Changing environments is a config change, not fifty edits.
>
> **Response specifications** do the same for common assertions:
>
> ```java
> ResponseSpecification okJson = new ResponseSpecBuilder()
>     .expectStatusCode(200)
>     .expectContentType(ContentType.JSON)
>     .expectResponseTime(lessThan(2000L))
>     .build();
> ```
>
> ---
>
> **Extract-and-assert versus chained validation:**
>
> ```java
> // Chained
> .then().statusCode(200).body("role", equalTo("admin"));
>
> // Extract, then assert
> Response response = given().spec(spec).when().get("/users/1").then().extract().response();
>
> assertThat(response.statusCode()).isEqualTo(200);
> assertThat(response.jsonPath().getString("role")).isEqualTo("admin");
> ```
>
> I lean toward extraction for anything non-trivial. Three reasons: you can inspect the response for multiple assertions without re-requesting, the failure messages from AssertJ or TestNG are more informative than RestAssured's Hamcrest output, and it's easier to debug because you can log the whole response.
>
> The chained form is fine for a simple status check.
>
> ---
>
> **JsonPath — extracting from responses:**
>
> ```java
> JsonPath json = response.jsonPath();
>
> String email = json.getString("email");
> int id = json.getInt("id");
> List<String> names = json.getList("users.name");
> String first = json.getString("users[0].name");
>
> // Conditional extraction — genuinely useful
> String adminEmail = json.getString("users.find { it.role == 'admin' }.email");
> List<String> activeUsers = json.getList("users.findAll { it.active == true }.name");
> ```
>
> Those last two use Groovy GPath expressions. Being able to filter a collection inline is a strong RestAssured capability that most people don't know exists — worth mentioning because it demonstrates you went past the basics.
>
> ---
>
> **Authentication:**
>
> ```java
> given().auth().basic("user", "pass")
> given().auth().oauth2(token)
> given().header("Authorization", "Bearer " + token)
> ```
>
> In practice I'd acquire the token once in a setup method or a `@BeforeSuite` and reuse it, rather than authenticating per test. That's the same principle as storage state in Playwright — don't pay the auth cost hundreds of times.
>
> ---
>
> **Logging — practical detail:**
>
> ```java
> .log().ifValidationFails()          // only on failure — the right default
> .log().all()                         // everything — debugging only
> ```
>
> `log().all()` on every test produces unreadable CI output. `ifValidationFails()` gives you the request and response exactly when you need them and stays quiet otherwise. That's the same principle as trace-on-first-retry.
>
> ---
>
> **Versus Playwright's request context, having used both:**
>
> RestAssured is more expressive for pure API work — specifications, GPath filtering, and the fluent assertions are genuinely nice. Playwright's request context is simpler but means one framework for UI and API rather than two projects, two sets of data helpers, two reports.
>
> For a dedicated API suite in a Java shop, RestAssured. For a team already running Playwright for UI, the consolidation usually wins."

---

## Structure Reference

| Element | Purpose |
|---------|---------|
| `given()` | Request setup — headers, auth, body, params |
| `when()` | Execute — get, post, put, delete |
| `then()` | Validate |
| `RequestSpecBuilder` | Reusable request config |
| `ResponseSpecBuilder` | Reusable response expectations |
| `extract().response()` | Get the response for code assertions |
| `jsonPath()` | Navigate the response body |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "API validation was a substantial part of that role — the application had a heavy backend and a lot of the real logic lived there, so testing it directly was more valuable than driving everything through the UI.
>
> We built request specifications rather than repeating configuration. Base URI from config, common headers, logging filters — defined once. Switching environments was a properties change rather than editing every test, and that's what makes it a framework rather than a collection of scripts.
>
> I preferred extracting the response and asserting in code over chaining into `then()`. You can make several assertions on one response without re-requesting, the failure messages are more informative, and you can log the whole payload when debugging.
>
> The capability I'd point to as underused is GPath filtering — expressions like `users.find { it.role == 'admin' }.email` let you filter a collection inline rather than pulling the list and looping. That's real expressiveness you don't get in most HTTP clients.
>
> On logging, `ifValidationFails()` rather than `log().all()`. Full logging on every test makes CI output unreadable; conditional logging gives you the request and response exactly when something broke."

---

## Common Interview Questions & Answers

### Q1: "Explain RestAssured's syntax."

**A:** "given/when/then — `given` builds the request with headers, auth, and body, `when` executes the HTTP method, `then` validates. What matters beyond the syntax is request specifications: defining base URI, headers, and filters once via `RequestSpecBuilder` and reusing them, so environment changes are one edit rather than fifty."

### Q2: "How do you avoid repeating configuration across tests?"

**A:** "`RequestSpecBuilder` for requests and `ResponseSpecBuilder` for common expectations. Base URI from config, standard headers, logging filters — all defined once. Tests then just pass `spec(baseSpec())`. Without that you end up with the same six lines in every test and no single place to change the environment."

### Q3: "Chained validation or extract-and-assert?"

**A:** "Extract for anything non-trivial. You can assert multiple things on one response without re-requesting, the failure messages from AssertJ are more informative than Hamcrest's, and you can log the full payload while debugging. Chaining is fine for a simple status check."

### Q4: "How do you extract a specific item from a JSON array?"

**A:** "JsonPath with GPath — `json.getString(\"users.find { it.role == 'admin' }.email\")` filters inline. Or `findAll` for multiple matches. That's a strong RestAssured capability most people don't know about; the alternative is pulling the whole list and looping in Java."

### Q5: "How does this compare to Playwright's request context?"

**A:** "RestAssured is more expressive — specifications, GPath, fluent assertions. Playwright's is simpler and means one framework for UI and API instead of two projects with separate data helpers and reports. Having used both, I'd pick RestAssured for a dedicated API suite in a Java shop, and Playwright's for a team already running it for UI."

---

## Weak Areas to Watch

❌ **Don't repeat configuration in every test**
→ Specifications exist for exactly this.

❌ **Don't use `log().all()` by default**
→ Unreadable CI output. `ifValidationFails()`.

❌ **Don't authenticate per test**
→ Acquire once and reuse.

❌ **Don't chain complex validations**
→ Extraction gives better failure messages.

❌ **Don't overlook GPath**
→ It's the capability that shows you went beyond tutorials.

---

## Key Takeaway for Interview

You have real API depth from this stack — lead with it.

What separates framework from scripts:

> **"Request specifications. Base URI, headers, and filters defined once and reused, so switching environments is a config change rather than fifty edits."**

And the capability worth demonstrating:

> **"GPath filtering — `users.find { it.role == 'admin' }.email` filters a collection inline instead of pulling the list and looping. Most people don't know RestAssured can do that."**

---

## Checklist

✅ `RequestSpecBuilder` for shared config
✅ `ResponseSpecBuilder` for common expectations
✅ Base URI from config, never hardcoded
✅ Token acquired once, reused
✅ `log().ifValidationFails()` rather than `all()`
✅ Extraction preferred over chained validation
✅ Comfortable with GPath find/findAll

---

**Remember:** Six repeated lines in every test is a specification you haven't written yet. 🎯
