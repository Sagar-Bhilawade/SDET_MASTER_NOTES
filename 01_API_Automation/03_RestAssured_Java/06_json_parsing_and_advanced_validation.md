# 06: JSON Parsing and Advanced Validation

### 1. 🚀 Tutorial Overview
JSON is the lingua franca of modern APIs. While basic status code checks are necessary, they are insufficient for verifying data integrity. This tutorial focuses on **JSON Parsing**—the art of navigating complex response payloads to validate specific fields, iterate through arrays, and perform mathematical operations on response data.

- **Purpose:** To perform surgical assertions on specific data points and aggregate data from collections.
- **Context:** Primarily occurs in the `Then` phase of the DSL or via standalone `JsonPath` extraction.
- **Prerequisites:** Familiarity with the Given/When/Then syntax and basic Hamcrest matchers.

---

### 2. 📌 Core Theoretical Pillars

#### A. JsonPath & GPath
RestAssured’s JSON navigation is powered by **GPath** (Groovy Path), which is significantly more powerful than standard JSONPath. It allows for:
- **Direct Navigation:** Dot notation for objects and index notation for arrays.
- **Collection Manipulation:** Built-in methods to filter (`find`, `findAll`), transform (`collect`), and aggregate (`sum`, `max`, `min`) data.

#### B. The "In-Memory" Advantage
Unlike simple string matching, `JsonPath` parses the entire response into a traversable tree. This allows you to perform "Dynamic Assertions" where the expected value depends on other fields within the same response.

---

### 3. 🛠️ Step-by-Step Chronological Implementation

#### Phase 1: Basic Parsing & Field Validation
The most common use case is verifying a single value in a flat or nested JSON structure.

```java
// Sample Response: { "status": "success", "user": { "id": 101, "name": "Sagar" } }
given()
.when()
    .get("/user/101")
.then()
    .body("status", equalTo("success"))
    .body("user.id", equalTo(101))
    .body("user.name", equalTo("Sagar"));
```

#### Phase 1.5: Handling Root JSON Arrays
Sometimes an API returns an array as the root element (e.g., `[{"id":1}, {"id":2}]`). In RestAssured, use the `$` symbol or index notation directly to reference the root.

```java
// Sample Response: [{"id":1, "name":"A"}, {"id":2, "name":"B"}]
given()
.when()
    .get("/users")
.then()
    .body("$.size()", equalTo(2)) // Validating root array size
    .body("[0].id", equalTo(1))    // Accessing first item directly
    .body("id", hasItems(1, 2));   // Implicitly collects all 'id' fields from the root array
```

#### Phase 2: Navigating Complex Hierarchies (JsonPath Usage)
For deeper trees or when you need to extract data for debugging, use the `JsonPath` object.

```java
Response response = get("/store/inventory");
JsonPath jp = response.jsonPath();

// Extracting nested values
String firstBookTitle = jp.get("store.book[0].title");
int totalBooks = jp.get("store.book.size()");

// Deep Scan (Find any author in the document)
List<String> allAuthors = jp.get("store.book.author");
```

#### Phase 3: Advanced Validation & Data Accumulation (Senior Level)
Using Groovy closures to find specific items in an array and performing logic on them.

```java
// 1. Searching for a specific value in an array (The Loop Pattern)
// Find the price of the book where title is 'Moby Dick'
float price = jp.get("store.book.find { it.title == 'Moby Dick' }.price");

// 2. Finding all items matching a criteria
List<String> expensiveBooks = jp.get("store.book.findAll { it.price > 15 }.title");

// 3. Data Accumulation: Calculating Totals
// Calculate the sum of all book prices in the response
float totalInventoryValue = jp.get("store.book.collect { it.price }.sum()");

// 4. Advanced Logic: Max/Min
float maxPrice = jp.get("store.book.price.max()");
```

#### Phase 4: Manual Parsing with org.json (JSONObject & JSONArray)
While `JsonPath` is the standard for RestAssured, some legacy or complex frameworks prefer the `org.json` library for manual traversal. This is useful when you need to perform deep Java-based logic outside the GPath syntax.

```java
Response response = get("/users/all");
String responseBody = response.asString();

// 1. If response is a JSONObject
JSONObject jo = new JSONObject(responseBody);
String status = jo.getString("status");
JSONObject userDetails = jo.getJSONObject("user");

// 2. If response is a JSONArray
JSONArray ja = new JSONArray(responseBody);
for(int i=0; i < ja.length(); i++) {
    JSONObject user = ja.getJSONObject(i);
    System.out.println("User ID: " + user.getInt("id"));
    
    // Conditional Logic in Java
    if(user.getString("role").equals("admin")) {
        Assert.assertTrue(user.has("permissions"));
    }
}
```

---

### 4. 📊 Technical Component Encyclopedia

| Keyword/Method | Source | Method Signature | Definition & Example |
| :--- | :--- | :--- | :--- |
| `$` | JsonPath Syntax | N/A | Represents the **Root** element. Essential for root arrays. <br> `$.size()` |
| `[n]` | JsonPath Syntax | N/A | Index accessor for arrays. <br> `[0]` (First item), `[-1]` (Last item). |
| `it` | GPath / Groovy | Implicit Variable | Represents the current item in a loop/closure. <br> `findAll { it.id > 5 }` |
| `find` | GPath / Groovy | `find { closure }` | Returns the **first** item matching the criteria. <br> `book.find { it.author == 'Homer' }` |
| `findAll` | GPath / Groovy | `findAll { closure }` | Returns **all** items matching the criteria as a List. |
| `collect` | GPath / Groovy | `collect { closure }` | Transforms a collection into another (e.g., Object list to Price list). |
| `sum()` | GPath / Groovy | `sum()` | Calculates the total of a numeric collection. <br> `items.price.sum()` |
| `jsonPath()` | `io.restassured.response.Response` | `jsonPath()` | Returns a `JsonPath` object to query the response. |
| `getList()` | `io.restassured.path.json.JsonPath` | `getList(String path)` | Specifically extracts a list of objects. |
| `JSONObject` | `org.json` | `new JSONObject(String)` | Converts a JSON string into a searchable Java object. |
| `JSONArray` | `org.json` | `new JSONArray(String)` | Converts a JSON string into a searchable Java array. |
| `.getJSONObject()`| `org.json.JSONObject` | `getJSONObject(String key)`| Extracts a nested object from a `JSONObject`. |
| `.getJSONArray()` | `org.json.JSONObject` | `getJSONArray(String key)` | Extracts a nested array from a `JSONObject`. |

---

### 5. 🔄 Visual Workflow (Mermaid)

```mermaid
graph TD
    A[API Response Received] --> B[Parse String to JsonPath Tree]
    B --> C{Navigation Style}
    C -->|Static| D[Direct Path: store.book[0]]
    C -->|Dynamic| E[GPath Closure: find/findAll]
    
    D --> F[Assertion/Extraction]
    E --> G[Filter/Transform Data]
    G --> H[Aggregate: sum/max/min]
    H --> F
    
    F --> I[Final Validation Success/Failure]
```

---

### 6. 🏭 Real-World Industrial Patterns

#### A. Total Validation Utility
In e-commerce testing, validating that the "Grand Total" matches the sum of "Line Items" is a mandatory check.

```java
public void validateOrderTotal(Response response) {
    JsonPath jp = response.jsonPath();
    
    // Extract individual prices
    List<Float> itemPrices = jp.get("order.items.price");
    float expectedTotal = jp.get("order.totalAmount");
    
    // Logic: Calculate sum and compare
    float actualSum = 0;
    for(float p : itemPrices) {
        actualSum += p;
    }
    
    // Senior Tip: Use GPath for cleaner code
    float gPathSum = jp.get("order.items.price.sum()");
    
    Assert.assertEquals(gPathSum, expectedTotal, "Order total mismatch!");
}
```

#### B. Dynamic Existence Check
Validating that an item exists in a list based on a dynamic ID generated in a previous step.

```java
String targetBook = "Java Design Patterns";

given()
.when()
    .get("/books")
.then()
    .body("books.title", hasItem(targetBook)) // Hamcrest collection check
    .body("books.find { it.title == '" + targetBook + "' }.status", equalTo("available")); // Deep check
```

---

### 🛡️ Content Preservation Note
- This tutorial complements **03: Assertions and Matchers** by providing the logic required to feed data into those matchers.
- Use `JsonPath` sparingly for simple responses; prefer POJOs (documented in **04: Test Data Handling**) for complex, reused models.
