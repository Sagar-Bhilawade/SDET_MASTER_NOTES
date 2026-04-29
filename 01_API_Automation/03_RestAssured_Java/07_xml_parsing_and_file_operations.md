# 07: XML Parsing and File Operations (Upload/Download)

### 1. 🚀 Tutorial Overview
While JSON dominates modern RESTful services, enterprise automation often requires interacting with legacy SOAP systems or handling binary data through file uploads and downloads. This tutorial deep-dives into **XML Parsing** using `XmlPath` and **File Operations** using Multi-part requests.

- **Purpose:** To validate XML-based responses and automate file transfer scenarios.
- **Context:** `XmlPath` is used in the `Then` phase, while `.multiPart()` is used in the `Given` phase.
- **Prerequisites:** Understanding of XML structure (nodes, attributes) and multipart form-data.

---

### 2. 📌 Core Theoretical Pillars

#### A. XmlPath & GPath
Like `JsonPath`, `XmlPath` uses Groovy's GPath for navigation. However, it introduces specific syntax for:
- **Node Navigation:** Using dots (`.`) to move down the tree.
- **Attributes:** Using the `@` symbol (e.g., `node.@id`).
- **Namespaces:** Handling prefix-based nodes common in SOAP.

#### B. Multi-part Form Data
The `multipart/form-data` content type allows sending multiple types of data (text fields + binary files) in a single HTTP request. Each part is separated by a boundary string.

#### C. Static vs. Dynamic Validation
- **Static XML:** Validating nodes by specific index (`node[0]`) when the order is guaranteed.
- **Dynamic XML:** Searching for nodes using criteria (`find`, `findAll`) when the order or number of nodes changes.

---

### 3. 🛠️ Step-by-Step Chronological Implementation

#### Phase 1: XML Parsing Foundations
Traversing nodes and capturing attributes from an XML response.

```java
// Sample XML: <travel><user id="101"><name>Sagar</name></user></travel>
Response response = get("/travel/info");
XmlPath xp = response.xmlPath();

// 1. Basic Traversal
String name = xp.get("travel.user.name");

// 2. Attribute Extraction
String userId = xp.get("travel.user.@id");

// 3. Root Array Logic (Multiple Users)
int userCount = xp.get("travel.user.size()");
```

#### Phase 2: Advanced XML Validation (Dynamic vs Static)
Handling scenarios where node positions might shift.

```java
// Static Validation: When index is known
given().when().get("/users").then()
    .body("users.user[0].name", equalTo("Sagar"));

// Dynamic Validation: Searching for a specific value (GPath)
String targetEmail = xp.get("users.user.find { it.name == 'Sagar' }.email");
```

#### Phase 3: File Upload (Multi-part)
Setting the `Content-Type` to `multipart/form-data` and attaching files.

```java
File testFile = new File("src/test/resources/sample.pdf");

given()
    .multiPart("file", testFile) // "file" is the control name expected by API
    .multiPart("description", "User Passport Copy") // Sending text part
    .contentType("multipart/form-data")
.when()
    .post("/upload")
.then()
    .statusCode(200)
    .body("message", containsString("uploaded successfully"));
```

#### Phase 4: File Download & Validation
Capturing the response as a byte array or file and validating its integrity.

```java
byte[] downloadedFile = 
    given()
    .when()
        .get("/download/report.pdf")
    .then()
        .statusCode(200)
        .header("Content-Type", "application/pdf")
        .extract().asByteArray();

// Validate file size is greater than 0
Assert.assertTrue(downloadedFile.length > 0);

// Logic: Save to local disk for manual verification if needed
Files.write(Paths.get("target/downloaded.pdf"), downloadedFile);
```

---

### 4. 📊 Technical Component Encyclopedia

| Component | Source/Library | Method Signature | Definition & Example |
| :--- | :--- | :--- | :--- |
| `xmlPath()` | `io.restassured.response.Response` | `xmlPath()` | Returns an `XmlPath` object to query XML. |
| `.multiPart()` | `io.restassured.specification.RequestSpecification` | `multiPart(String name, File file)` | Attaches a file to the request. |
| `.multiPart()` | `io.restassured.specification.RequestSpecification` | `multiPart(String name, String value)`| Attaches a text part to a multipart request. |
| `@` | XmlPath Syntax | `@attributeName` | Accesses the value of an XML attribute. <br> `user.@id` |
| `**` | XmlPath Syntax | `**` | Deep scan / Recursive search (Depth-first). |
| `asByteArray()`| `io.restassured.response.ExtractableResponse` | `asByteArray()` | Extracts response body as binary data. |

---

### 5. 🔄 Visual Workflow (Mermaid)

```mermaid
graph TD
    subgraph Upload_Flow
    A[Start Upload] --> B[Create File Object]
    B --> C[Set multiPart File + Fields]
    C --> D[Set Content-Type: multipart/form-data]
    D --> E[Execute POST]
    end

    subgraph XML_Parsing_Flow
    F[Receive XML] --> G[Parse to XmlPath Tree]
    G --> H{Validation Type}
    H -->|Static| I[Index Based: user[0]]
    H -->|Dynamic| J[GPath Search: find]
    I --> K[Assert Value]
    J --> K
    end
```

---

### 6. 🏭 Real-World Industrial Patterns

#### A. Multiple File Upload Utility
Professional frameworks often wrap upload logic to handle multiple files in a single call.

```java
public void uploadDocumentBatch(List<File> files) {
    RequestSpecification spec = given();
    
    for (File f : files) {
        spec.multiPart("documents", f);
    }
    
    spec.when().post("/batch-upload").then().statusCode(207); // 207 Multi-Status
}
```

#### B. SOAP Envelope Validation
Validating specific business logic inside a heavy SOAP response.

```java
// Logic: Validating TransactionStatus inside a SOAP Body
given()
.when()
    .post("/soap-service")
.then()
    .body("Envelope.Body.TransactionResponse.Status", equalTo("COMPLETED"))
    .body("Envelope.Body.TransactionResponse.@id", notNullValue());
```

---

### 🛡️ Content Preservation Note
- This tutorial extends the concepts from **06: JSON Parsing**, applying GPath logic to the XML domain.
- For large file uploads, ensure the timeout configurations in your HTTP client are adjusted to prevent premature connection closure.
