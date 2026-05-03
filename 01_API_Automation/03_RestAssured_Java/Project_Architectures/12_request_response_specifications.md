# 12: Request and Response Specifications

### 1. 🚀 Tutorial Overview
As your framework grows to hundreds of endpoints, you'll notice a lot of repetitive code (Headers, Content-Types, Base URIs). **Specifications** allow you to pre-define these common attributes into reusable "templates," significantly reducing boilerplate code and making the framework easier to maintain.

- **Objective:** Consolidate common request/response logic.
- **Key Tools:** `RequestSpecBuilder` and `ResponseSpecBuilder`.
- **Architectural Shift:** Moving common configuration from the method level to a global specification.

---

### 2. 📌 Core Theoretical Pillars

#### A. RequestSpecBuilder
This builder allows you to group common request parameters:
- `Base URI` and `Base Path`.
- `Authentication` (Tokens/Keys).
- `Headers` (Accept, Content-Type, Custom headers).
- `Logging` settings.

#### B. ResponseSpecBuilder
Used to define expected response patterns across multiple tests:
- Expected `Status Code`.
- Expected `Content Type`.
- Common `Response Headers` (e.g., Server name, Cache control).
- `Response Time` thresholds.

---

### 3. 🛠️ Step-by-Step Implementation

#### Phase 1: Creating a Specification Manager
We create a central class to hold our specifications.

```java
// api.base.SpecManager.java
public class SpecManager {
    public static RequestSpecification requestSpec;
    public static ResponseSpecification responseSpec;

    public static void initSpecs() {
        // 1. Build Request Specification
        requestSpec = new RequestSpecBuilder()
                .setBaseUri(Routes.base_url)
                .setContentType(ContentType.JSON)
                .addHeader("Authorization", "Bearer " + BaseTest.bearerToken)
                .addFilter(new RequestLoggingFilter()) // Automatic Logging
                .addFilter(new ResponseLoggingFilter())
                .build();

        // 2. Build Response Specification (Common for success cases)
        responseSpec = new ResponseSpecBuilder()
                .expectStatusCode(200)
                .expectContentType(ContentType.JSON)
                .expectHeader("Server", "cloudflare")
                .build();
    }
}
```

#### Phase 2: Refactoring Endpoints
Now, your `UserEndpoints.java` becomes incredibly lean.

```java
// api.endpoints.UserEndpoints.java
public class UserEndpoints {

    public static Response createUser(User payload) {
        return given()
                .spec(SpecManager.requestSpec) // Injecting all headers and auth
                .body(payload)
            .when()
                .post(Routes.post_url);
    }

    public static Response getUser(String name) {
        return given()
                .spec(SpecManager.requestSpec)
                .pathParam("username", name)
            .when()
                .get(Routes.get_url)
            .then()
                .spec(SpecManager.responseSpec) // Automatically validates 200 and ContentType
                .extract().response();
    }
}
```

---

### 4. 📊 Technical Component Encyclopedia

| Component | Responsibility | Pattern |
| :--- | :--- | :--- |
| **RequestSpecBuilder** | Constructing a `RequestSpecification` object. | Builder |
| **ResponseSpecBuilder**| Constructing a `ResponseSpecification` object. | Builder |
| **.spec()** | Injecting the template into the request chain. | Injection |
| **Logging Filters** | Global logging configuration within the spec. | Strategy |

---

### 5. 🏭 Real-World Industrial Patterns

#### A. Multiple Spec Strategy
Professional frameworks often use multiple specifications based on the use case:
- `jsonRequestSpec`: Content-Type JSON.
- `multiPartRequestSpec`: Content-Type Multi-part (for file uploads).
- `unauthorizedRequestSpec`: No tokens (to test negative scenarios).

#### B. Specification Inheritance
You can create a base specification and then "derive" specialized versions from it using `.addRequestSpecification()`.

```java
RequestSpecification specializedSpec = new RequestSpecBuilder()
    .addRequestSpecification(baseSpec)
    .addHeader("X-Special-Header", "Value")
    .build();
```

---

### 🛡️ Content Preservation Note
- Always call `.build()` at the end of the builder chain.
- If you override a header in the `.given()` section that is already in the `.spec()`, the manual header usually takes precedence.
