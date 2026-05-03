# 13: JSON Schema Validation and Complex Payloads

### 1. 🚀 Tutorial Overview
Simple field-level assertions are not enough to guarantee that an API won't break client applications. **JSON Schema Validation** ensures the entire response structure (data types, mandatory fields, patterns) is intact. Additionally, we explore handling **Complex Payloads** that go beyond basic POJOs.

- **Objective:** Enforce strict structural integrity and manage nested JSON.
- **Tools:** `rest-assured-json-schema-validator`, Jackson Annotations.
- **Payload Patterns:** Dynamic `Map` construction and Nested Objects.

---

### 2. 📌 Core Theoretical Pillars

#### A. JSON Schema Validation
A JSON Schema is a contract that defines how a response *should* look. Instead of checking 50 individual fields, you compare the response against a `.json` schema file in one line of code.

#### B. Serialization Control (Jackson)
Sometimes, your Java Object has fields that the API doesn't want (or vice versa). We use Jackson annotations to control how Java objects turn into JSON.

#### C. Dynamic vs. Static Payloads
- **Static (POJO):** Best for stable, fixed endpoints.
- **Dynamic (Map/JSONObject):** Best for endpoints where the payload structure changes based on user input or random conditions.

---

### 3. 🛠️ Step-by-Step Implementation

#### Phase 1: Dependency Configuration
Update `pom.xml` for schema validation.

```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>json-schema-validator</artifactId>
    <version>5.3.0</version>
</dependency>
```

#### Phase 2: Implementing Schema Validation
1. Generate a schema from a sample response (use [jsonschema.net](https://jsonschema.net)).
2. Save it as `user-schema.json` in `src/test/resources`.

```java
// api.tests.UserTests.java
@Test
public void testUserSchema() {
    UserEndpoints.getUser("john_doe")
        .then()
            .assertThat()
            .body(JsonSchemaValidator.matchesJsonSchemaInClasspath("user-schema.json"));
}
```

#### Phase 3: Handling Complex/Nested Payloads
For an endpoint like `Create Order` with nested items:

```java
// api.payloads.Order.java
public class Order {
    public int id;
    public List<Item> items; // Nested list of objects
    
    public static class Item {
        public String prodName;
        public int quantity;
    }
}
```

#### Phase 4: Dynamic Payload construction (using Maps)
When you don't want to create a POJO class for a one-off test.

```java
@Test
public void testDynamicPayload() {
    Map<String, Object> payload = new HashMap<>();
    payload.put("username", "test_user");
    payload.put("id", 101);
    
    Map<String, String> profile = new HashMap<>();
    profile.put("bio", "SDET Architect");
    payload.put("profile", profile); // Nested Map

    Response response = given()
            .body(payload)
            .post(Routes.post_url);
}
```

---

### 4. 📊 Technical Component Encyclopedia

| Strategy | When to Use | Tool |
| :--- | :--- | :--- |
| **POJO** | Standard CRUD operations. | `Lombok` / `Jackson` |
| **Map<String, Object>** | Rapid prototyping / Dynamic fields. | Native Java |
| **Schema Validation** | Integration/Contract testing. | `json-schema-validator` |
| **@JsonIgnore** | Hide fields during serialization. | Jackson Annotation |

---

### 5. 🏭 Real-World Industrial Patterns

#### A. The "Object Mapper" Configuration
You can configure a global `ObjectMapper` to handle specific date formats or ignore unknown properties globally.

```java
RestAssured.config = RestAssuredConfig.config().objectMapperConfig(
    new ObjectMapperConfig().jackson2ObjectMapperFactory(
        (cls, charset) -> new ObjectMapper().configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
    )
);
```

#### B. Data-Driven Schema Validation
In enterprise frameworks, you might have different schemas for different user roles. You can pass the schema filename as a parameter in a `@DataProvider`.

---

### 🛡️ Content Preservation Note
- JSON Schema files MUST be in the classpath (`src/test/resources`).
- If the API returns an array at the root, the schema must define the type as `array`.
