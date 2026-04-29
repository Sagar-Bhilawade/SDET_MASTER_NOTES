# RestAssured Assertions & Matchers

### 1. 🚀 Tutorial Overview
Assertions are the most critical component of any automated test. In RestAssured, assertions are the mechanism used to validate that the API response matches the expected state. This tutorial covers the transition from simple status code checks to complex collection validations and industrial schema matching.

**Prerequisites:**
- Basic understanding of HTTP Methods (GET, POST).
- RestAssured Setup (v5.x+ recommended).
- Java environment with Maven/Gradle.

---

### 2. 📌 Core Theoretical Pillars
RestAssured assertions are built on two foundational technologies working in tandem:
- **GPath Expressions (The Navigator):** A Groovy-based path language used **exclusively for traversing and querying** JSON/XML data. GPath's job is to find a specific node or collect a list of values. It does NOT assert; it only "reaches" the data.
- **Hamcrest Matchers (The Judge):** A framework for performing the actual validation logic on the value provided by GPath. It "judges" whether the reached data meets the criteria (e.g., is it equal, greater, or contains a string?).

**The Synergy:**
`body("users.find { it.id == 1 }.name", equalTo("Sagar"))`
- `users.find { it.id == 1 }.name` -> **GPath** (Navigates to the name of user with ID 1).
- `equalTo("Sagar")` -> **Hamcrest** (Asserts the extracted name is "Sagar").

---

### 3. 🛠️ Step-by-Step Chronological Implementation

#### Phase 1: The Foundations (Entry Level)
At the basic level, we focus on the high-level attributes of the HTTP Response: Status Codes and Basic Headers.

```java
given()
    .baseUri("https://api.example.com")
.when()
    .get("/users/1")
.then()
    .statusCode(200)                            // Assert Status Code
    .statusLine("HTTP/1.1 200 OK")              // Assert Full Status Line
    .header("Content-Type", "application/json") // Assert Specific Header
    .contentType(ContentType.JSON);             // Assert Content Type
```

#### Phase 2: Intermediate Configurations (Body Validation)
Once the envelope is validated, we dive into the message body. We start using simple Hamcrest matchers to verify field values.

```java
.then()
    .body("first_name", equalTo("John"))        // Single field match
    .body("last_name", is("Doe"))               // Using 'is' wrapper for readability
    .body("age", greaterThan(18))               // Numerical range check
    .body("active", is(true));                  // Boolean check
```

#### Phase 3: Professional Optimization (Lists & GPath)
In real-world APIs, we often deal with arrays of objects. We use GPath to filter and extract data before asserting.

```java
.then()
    // Logic: Find all orders where price > 100, then check if their IDs are not empty
    .body("orders.findAll { it.price > 100 }.id", hasSize(greaterThan(0)))
    
    // Logic: Verify that at least one order has the status 'SHIPPED'
    .body("orders.status", hasItem("SHIPPED"))
    
    // Performance: Verify the response time is within SLA
    .time(lessThan(2000L));
```

#### Phase 4: External Assertions (TestNG / JUnit)
Sometimes, inline assertions are insufficient. We use **TestNG `Assert`** when we need to extract the response into a variable or a POJO and then perform complex comparisons (e.g., comparing API data vs Database).

```java
// 1. Extract data to a variable
String email = given().when().get("/user/1").then().extract().path("email");

// 2. Use TestNG Assert (Hard Assertion)
Assert.assertEquals(email, "sagar@example.com", "Email mismatch found!");

// 3. POJO Comparison
User user = response.as(User.class);
Assert.assertNotNull(user.getId());
```

---

### 4. 📊 Technical Component Encyclopedia

| Source/Library | Category | Matcher / Method | Method Signature | Description & Example |
| :--- | :--- | :--- | :--- | :--- |
| `io.restassured` | **Status** | `statusCode()` | `.statusCode(int code)` | Validates the HTTP status. `statusCode(200)` |
| `io.restassured` | **Header** | `header()` | `.header(String k, Matcher v)` | Validates header values. `header("Server", containsString("nginx"))` |
| `org.hamcrest` | **Core** | `equalTo()` | `equalTo(T value)` | Standard equality check. `body("id", equalTo(1))` |
| `org.hamcrest` | **Core** | `is()` | `is(T value)` | Decorator for readability. `body("status", is("ACTIVE"))` |
| `org.testng` | **Runner** | `assertEquals()` | `Assert.assertEquals(act, exp)` | Hard assertion on extracted variables. |
| `org.testng` | **Runner** | `assertNotNull()`| `Assert.assertNotNull(object)` | Checks if extracted object is valid. |
| `org.hamcrest` | **String** | `containsString()` | `containsString(String s)` | Substring check. `body("email", containsString("@"))` |
| `org.hamcrest` | **String** | `matchesPattern()` | `matchesPattern(String regex)`| Regex validation. `body("zip", matchesPattern("\\d{5}"))` |
| `org.hamcrest` | **Number** | `greaterThan()` | `greaterThan(N value)` | Numeric comparison. `body("score", greaterThan(90))` |
| `org.hamcrest` | **Number** | `closeTo()` | `closeTo(double v, double err)`| Float/Double precision. `body("price", closeTo(10.5, 0.1))` |
| `org.hamcrest` | **Collection**| `hasItem()` | `hasItem(T item)` | Checks if item exists in list. `body("tags", hasItem("java"))` |
| `org.hamcrest` | **Collection**| `hasItems()` | `hasItems(T... items)` | Checks multiple items exist. `body("tags", hasItems("java", "api"))` |
| `org.hamcrest` | **Collection**| `contains()` | `contains(T... items)` | Exact items in exact order. `body("ids", contains(1, 2, 3))` |
| `org.hamcrest` | **Collection**| `containsInAnyOrder()`| `containsInAnyOrder(T... i)` | Exact items in any order. `body("ids", containsInAnyOrder(3, 1, 2))` |
| `org.hamcrest` | **Collection**| `everyItem()` | `everyItem(Matcher m)` | Validat every element. `body("prices", everyItem(greaterThan(0)))` |

---

### 5. 🔄 Visual Workflow (Mermaid)

```mermaid
graph TD
    subgraph "The Request-Response Loop"
        A[Client Request] --> B[Server Response]
    end

    subgraph "RestAssured Validation Engine"
        B --> C{then() Block}
        C --> D[GPath Parser]
        D -->|Navigation| E[Extract Data Node]
        E --> F{Hamcrest Matcher}
        F -->|Validation| G{Success / Fail}
    end

    subgraph "Failure Reporting"
        G -->|Fail| H[Throw AssertionError]
        H --> I[Detailed Mismatch Report]
    end
```

---

### 6. 🏭 Real-World Industrial Patterns

#### Pattern 1: The "Soft Assertion" Wrapper
In production frameworks, we avoid individual `body()` calls and group them to see multiple failures at once.

```java
public void validateUserResponse(Response response, String expectedName) {
    response.then()
        .assertThat()
        .body("user.name", equalTo(expectedName),
              "user.role", is("ADMIN"),
              "user.metadata.lastLogin", notNullValue());
}
```

#### Pattern 2: Global Response Specification
Centralizing common assertions (like 200 OK and JSON Content-Type) using `ResponseSpecBuilder`.

```java
ResponseSpecification commonSpec = new ResponseSpecBuilder()
    .expectStatusCode(200)
    .expectContentType(ContentType.JSON)
    .expectResponseTime(lessThan(5000L))
    .build();

// Usage in tests
given().spec(requestSpec).when().get("/endpoint").then().spec(commonSpec);
```
