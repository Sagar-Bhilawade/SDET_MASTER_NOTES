# 02: Core DSL Syntax (Given / When / Then)

### 1. Summary of Why (Architectural Context)
RestAssured follows the **BDD (Behavior Driven Development)** pattern. This separation of concerns ensures that test code reads like a requirement document:
- **Given:** Pre-conditions (Auth, Headers, Payload).
- **When:** The Action (HTTP Method + Endpoint).
- **Then:** The Post-conditions (Assertions & Extraction).
- **Senior Tip:** This structure enforces **Statelessness**. Every `given()` block should ideally contain everything needed for that specific request to succeed in isolation.

### 2. Clean Code (Production-Ready Snippets)
A professional request combines multiple pre-conditions and strict post-condition validation.

```java
Response response = 
    given()
        .baseUri("https://api.example.com")
        .header("Content-Type", "application/json")
        .auth().oauth2("your_token")
        .cookie("session_id", "12345")
        .queryParam("status", "active")
        .pathParam("userId", 101)
        .body(payloadObject)
    .when()
        .post("/users/{userId}")
    .then()
        .log().ifValidationFails() // Industry Standard: Log only if something is wrong
        .statusCode(201)
        .header("Content-Encoding", "gzip")
        .extract().response(); // Extracting the full response object for further use
```

### 3. The "Industry Gap" (Missing in Documentation)
- **Param Nuances:**
    - `queryParam()`: Appended to the URL (`?key=val`). Used for filtering.
    - `pathParam()`: Replaces template variables in the URL (`/users/{id}`).
    - `formParam()`: Sent in the body as `application/x-www-form-urlencoded`.
- **Extraction Techniques:** 
    - `extract().path("id")`: Fast JSON path extraction.
    - `extract().as(User.class)`: Direct deserialization into a POJO.
- **Header Best Practices:** In real-world enterprise apps, headers like `X-Request-ID` or `X-Client-Version` are mandatory. Always parameterize these rather than hard-coding them in every `given()` block.

### 4. The Gotcha (Senior Insight)
- **The Extraction Order Trap:** You can extract *before* assertions, but a Senior SDET always asserts *before* extracting. If the status code isn't 200, extracting the body might throw a `NullPointerException` or a parsing error. **Always validate `statusCode()` before `extract()`.**
- **Cookie Persistence:** If your API uses session cookies, don't manually parse them. Use `.extract().cookies()` to get a `Map<String, String>` and pass it directly to the next `.cookies(map)` call.
- **Interview Trap (PATCH vs PUT):**
    - *"How do you code a partial update in your DSL?"*
    - **Answer:** Use `.when().patch()`. Passing a partial body to `.put()` is a common junior mistake that can lead to data loss (nullifying fields not present in the payload).
- **Log Bloating:** Never use `.log().all()` in a CI/CD pipeline. It generates megabytes of text which can crash Jenkins/GitHub Actions log viewers. Use `.log().ifValidationFails()` or `.log().ifError()`.
