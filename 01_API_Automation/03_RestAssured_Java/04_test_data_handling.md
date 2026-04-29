# RestAssured: Test Data Handling & Payload Management

### 1. 🚀 Tutorial Overview
In enterprise API automation, the "Payload" (Request Body) is the most volatile part of your test suite. A poorly managed payload strategy leads to "Hardcoded Hell," where tests break with every minor schema change. This tutorial provides a deep-dive into the four primary ways to handle test data in RestAssured: **HashMap**, **org.json**, **POJO Classes**, and **External JSON Files**.

**Context within Automation Lifecycle:**
- **Creation:** Generating the request body before the execution phase.
- **Serialization:** Converting Java structures into JSON strings.
- **Maintenance:** Ensuring data updates don't require rewriting dozens of tests.

**Prerequisites:**
- Understanding of HTTP POST/PUT/PATCH methods.
- Maven dependencies for `jackson-databind` or `gson`.
- Basic Java Object-Oriented programming concepts.

---

### 2. 📌 Core Theoretical Pillars

#### A. Serialization (The "Why")
Serialization is the process of converting a stateful Java Object (like a POJO or a Map) into a byte stream (typically a JSON String) that can be sent over the network. RestAssured simplifies this by automatically detecting serializers (Jackson/Gson) in the classpath.

#### B. Dynamic vs. Static Payloads
- **Dynamic Payloads:** Required when data changes per test execution (e.g., creating a user with a unique email). Handled best via **POJOs** or **org.json**.
- **Static Payloads:** Used for payloads that rarely change or are massive (100+ lines). Handled best via **External JSON Files**.

#### C. Type Safety and Refactoring
Using POJOs provides compile-time safety. If an API field name changes, you update the POJO once, and the IDE helps you find every broken reference. Maps and Strings lack this safety, leading to runtime failures.

---

### 3. 🛠️ Step-by-Step Chronological Implementation

#### Phase 1: Java Collections (HashMap & List)
The entry-level approach. No external libraries needed beyond standard Java. It is ideal for small, flat payloads where you don't want the overhead of creating classes.

**Implementation Logic:**
1. Create a `Map<String, Object>`.
2. For nested objects, put another `Map` inside the first one.
3. For arrays, use `ArrayList` or `Arrays.asList()`.

```java
// Logic: Creating { "username": "sagar", "roles": ["admin", "editor"] }
Map<String, Object> body = new HashMap<>();
body.put("username", "sagar");
body.put("roles", Arrays.asList("admin", "editor"));

given()
    .header("Content-Type", "application/json")
    .body(body) // RestAssured converts Map to JSON automatically
.when()
    .post("/api/users")
.then()
    .statusCode(201);
```

#### Phase 2: org.json Library (JSONObject & JSONArray)
A step up from Maps. It provides a dedicated API for JSON manipulation. It is highly flexible for building complex JSON structures on the fly without the rigidity of POJOs.

**Implementation Logic:**
1. Import `org.json` dependency in `pom.xml`.
2. Use `JSONObject` for curly braces `{}` and `JSONArray` for square brackets `[]`.

```java
// Logic: Creating { "id": 101, "info": { "loc": "India" } }
JSONObject innerBody = new JSONObject();
innerBody.put("loc", "India");

JSONObject mainBody = new JSONObject();
mainBody.put("id", 101);
mainBody.put("info", innerBody);

given()
    .contentType(ContentType.JSON)
    .body(mainBody.toString()) // Must use .toString() for org.json
.when()
    .post("/api/v1/resource");
```

#### Phase 3: POJO Classes (The Gold Standard)
The most professional approach. Plain Old Java Objects (POJOs) represent the JSON schema as a class. This provides the highest level of maintainability and integration with tools like Lombok.

**Implementation Logic:**
1. Create a class with private fields matching JSON keys.
2. Generate Getters and Setters (or use Lombok `@Data`).
3. Pass the object directly to `.body()`.

```java
// UserPayload.java
public class UserPayload {
    private String name;
    private String job;
    // Getters and Setters...
}

// Execution logic
UserPayload user = new UserPayload();
user.setName("Sagar");
user.setJob("Architect");

given()
    .body(user) // Automatic Serialization via Jackson
.when()
    .post("/api/users");
```

#### Phase 4: External JSON Files
Best for massive, complex payloads that are mostly static. Instead of cluttering Java code with 50 lines of `put()` calls, you keep the JSON in a separate file and read it as a String or File object.

**Implementation Logic:**
1. Save JSON in `src/test/resources/payloads/user.json`.
2. Use `new File()` or `Files.readAllBytes()` to pass it to RestAssured.

```java
// Logic: Sending data from a physical file
File jsonFile = new File(System.getProperty("user.dir") + "/src/test/resources/data.json");

given()
    .contentType(ContentType.JSON)
    .body(jsonFile)
.when()
    .post("/api/upload")
.then()
    .log().all();
```

#### Phase 5: Builder Pattern (Fluent Construction)
In modern frameworks, we avoid long lists of `set()` methods. The **Builder Pattern** (typically via **Lombok**) allows for a fluent, readable API to construct complex payloads. This is the industry standard for "Readability + Type Safety."

**Implementation Logic:**
1. Annotate the POJO with `@Builder`.
2. Chain the methods to set only the required fields.

```java
// Logic: Creating a user with only name and age, skipping email
UserPayload user = UserPayload.builder()
    .name("Sagar")
    .age(30)
    .build();

given().body(user).post("/api/users");
```

#### Phase 6: Data-Driven Integration (TestNG/CSV)
For large-scale regression, data is moved out of Java entirely into CSV or Excel files. We use TestNG's `@DataProvider` to feed these values into the test method, which then constructs the payload.

**Implementation Logic:**
1. Read data from CSV/Excel into a 2D Object array.
2. The Test method receives these parameters and builds the POJO/Map.

```java
@Test(dataProvider = "userCSVData")
public void testBulkUserCreation(String name, String role) {
    UserPayload payload = UserPayload.builder()
        .name(name)
        .role(role)
        .build();
        
    given().body(payload).post("/users").then().statusCode(201);
}
```

#### Phase 7: Advanced Serialization (Jackson MixIns)
Used when you need to add JSON annotations to classes from a 3rd-party library that you cannot modify. This is a high-level architectural pattern in enterprise frameworks.

**Implementation Logic:**
1. Create an abstract "MixIn" class with the required annotations.
2. Register the MixIn with the `ObjectMapper`.

```java
// ExternalUser.class (Cannot be modified)
// UserMixIn.class (Our annotation wrapper)
abstract class UserMixIn {
    @JsonProperty("user_name") abstract String getName();
}

ObjectMapper mapper = new ObjectMapper();
mapper.addMixIn(ExternalUser.class, UserMixIn.class);
```

---

### 4. 📊 Technical Component Encyclopedia

| Component | Source/Library | Class/Method | Responsibility | Industrial Usage Example |
| :--- | :--- | :--- | :--- | :--- |
| **HashMap** | `java.util` | `HashMap<K,V>` | Quick, non-reusable payloads. | `Map<String, String> data = new HashMap<>();` |
| **JSONObject** | `org.json` | `JSONObject` | Dynamic JSON construction without classes. | `new JSONObject().put("key", "val");` |
| **JSONArray** | `org.json` | `JSONArray` | Building JSON arrays dynamically. | `new JSONArray().put("item1").put("item2");` |
| **POJO** | `Custom Class` | `UserObj.class` | Reusable, type-safe data structures. | `given().body(new UserObj());` |
| **Jackson Databind**| `com.fasterxml.jackson` | `ObjectMapper` | The engine that converts Java to JSON. | `mapper.writeValueAsString(obj);` |
| **File Object** | `java.io` | `File` | Passing static files directly to body. | `given().body(new File("path.json"));` |
| **Lombok** | `org.projectlombok` | `@Builder` | Fluent API for creating POJO instances. | `User.builder().name("A").build();` |
| **JavaFaker** | `com.github.javafaker`| `Faker` | Random data generation for dynamic fields. | `faker.name().firstName();` |
| **Jackson MixIn** | `com.fasterxml.jackson` | `addMixIn()` | Adding annotations to classes you don't own. | `mapper.addMixIn(Target.class, MixIn.class);` |

---

---

### 5. 🔄 Visual Workflow (Mermaid)

```mermaid
graph TD
    A[Test Execution Starts] --> B{Choose Payload Strategy}
    
    B -->|Small/Flat| C[HashMap]
    B -->|Dynamic/Complex| D[org.json]
    B -->|Reusable/Type-Safe| E[POJO Class]
    B -->|Massive/Static| F[External File]
    
    C --> G[RestAssured Internal Serializer]
    D --> H[Manual .toString()]
    E --> G
    F --> I[Direct File Streaming]
    
    G --> J[HTTP Request Body]
    H --> J
    I --> J
    
    J --> K[API Response]
    K --> L[Deserialization to POJO/Map]
```

---

### 6. 🏭 Real-World Industrial Patterns

#### Pattern 1: The "Token Replacement" Strategy
Use an External JSON file as a template and replace specific values at runtime. This combines the "External File" and "Dynamic" approaches.

```java
// user_template.json: { "name": "{{NAME}}", "city": "Pune" }
String content = FileUtils.readFileToString(new File("user_template.json"), "UTF-8");
String dynamicPayload = content.replace("{{NAME}}", "Sagar Bhilawade");

given().body(dynamicPayload).post("/users");
```

#### Pattern 2: Nested POJO Construction
For enterprise schemas with deep nesting, use a parent POJO containing child POJOs.

```java
public class Address {
    private String street;
    private String zip;
}

public class Employee {
    private String name;
    private Address address; // Nested POJO
}

// Usage
Employee emp = new Employee();
emp.setAddress(new Address("MG Road", "411001"));
```

#### Pattern 3: Utility for External Data Reading
Create a dedicated utility to fetch JSON from resources, keeping tests clean.

```java
public class JsonUtils {
    public static String getPayload(String fileName) {
        try {
            return new String(Files.readAllBytes(Paths.get("src/test/resources/json/" + fileName)));
        } catch (IOException e) {
            throw new RuntimeException("Failed to read JSON file: " + fileName);
        }
    }
}
```

#### Pattern 4: The "Faker" Utility Factory
Never use hardcoded strings like "TestUser". Use a central Factory that leverages JavaFaker to return randomized but valid data.

```java
public class UserFactory {
    private static Faker faker = new Faker();

    public static UserPayload getRandomUser() {
        return UserPayload.builder()
            .name(faker.name().fullName())
            .email(faker.internet().safeEmailAddress())
            .build();
    }
}
```

#### Pattern 5: Serialization Config for Nulls
In industry, we often need to prevent `null` fields from being sent in the JSON.

```java
// Strategy: Global config to ignore nulls during serialization
ObjectMapper mapper = new ObjectMapper();
mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);

RestAssured.config = RestAssuredConfig.config()
    .jackson2ObjectMapperFactory((cls, charset) -> mapper);
```

