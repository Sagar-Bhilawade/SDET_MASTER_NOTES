# 14: Advanced Filters and File Handling

### 1. 🚀 Tutorial Overview
To reach the "Architect" level, you must understand how to intercept and manipulate requests globally. **Filters** allow you to implement cross-cutting concerns like global logging or automatic token injection. Additionally, we cover **File Handling**, a common requirement for APIs that process attachments or images.

- **Objective:** Intercept requests and automate multi-part form data.
- **Filters:** Custom `RequestFilter` and `ResponseFilter`.
- **Files:** Multi-part uploads and binary downloads.

---

### 2. 📌 Core Theoretical Pillars

#### A. The Filter Chain
A Filter is a piece of code that executes *before* the request reaches the server or *after* the response is received. It can modify the headers, body, or URI on the fly.

#### B. Multi-part/form-data
Standard JSON requests use `application/json`. File uploads require `multipart/form-data`. RestAssured handles this by wrapping the file into a "Part" of the HTTP request.

#### C. Binary Responses
Some APIs return images, PDFs, or Excel files. We must capture these as `byte[]` or `File` objects rather than strings.

---

### 3. 🛠️ Step-by-Step Implementation

#### Phase 1: Custom Logging Filter
Instead of calling `.log().all()` everywhere, create a global logger that writes to your Log4j2 files automatically.

```java
// api.utils.CustomLogFilter.java
public class CustomLogFilter implements Filter {
    @Override
    public Response filter(FilterableRequestSpecification requestSpec, 
                           FilterableResponseSpecification responseSpec, 
                           FilterContext ctx) {
        
        System.out.println("Executing: " + requestSpec.getMethod() + " " + requestSpec.getURI());
        Response response = ctx.next(requestSpec, responseSpec); // Execute request
        
        System.out.println("Response Status: " + response.getStatusCode());
        return response;
    }
}
```

#### Phase 2: Applying Global Filters
Apply the filter to your `RequestSpecification`.

```java
RequestSpecification spec = new RequestSpecBuilder()
    .addFilter(new CustomLogFilter())
    .addFilter(new ErrorLoggingFilter()) // Log only if error occurs
    .build();
```

#### Phase 3: Implementing File Upload
Testing the "Upload Pet Image" endpoint.

```java
// api.endpoints.PetEndpoints.java
public static Response uploadImage(int petId, File imageFile) {
    return given()
            .pathParam("petId", petId)
            .multiPart("file", imageFile) // KEY: multiPart
            .multiPart("additionalMetadata", "Optional description")
            .contentType("multipart/form-data")
        .when()
            .post(Routes.pet_upload_image_url);
}
```

#### Phase 4: Implementing File Download
Downloading a report or image from an API.

```java
@Test
public void testFileDownload() throws IOException {
    Response response = given().get("/download/report.pdf");
    
    // 1. Get as byte array
    byte[] bytes = response.asByteArray();
    
    // 2. Save to disk
    Files.write(Paths.get("target/report.pdf"), bytes);
}
```

---

### 4. 📊 Technical Component Encyclopedia

| Component | Responsibility | Method |
| :--- | :--- | :--- |
| **Filter** | Intercepting and modifying the HTTP cycle. | `ctx.next()` |
| **.multiPart()** | Sending files or form-data fields. | `multiPart("key", file)` |
| **.asByteArray()**| Capturing binary response data. | `response.asByteArray()` |

---

### 5. 🏭 Real-World Industrial Patterns

#### A. The "Auto-Auth" Filter
Instead of passing tokens in every endpoint, create a filter that checks if a token is present in the `BaseTest`. If yes, it adds the `Authorization` header to every request automatically.

#### B. Request/Response Capturing for Reporting
You can create a filter that captures the full cURL command of every request and attaches it to your **Extent Report** on failure. This makes debugging incredibly fast for developers.

---

### 🛡️ Content Preservation Note
- For file uploads, ensure the "key" name (`"file"`) matches exactly what the API documentation expects.
- `multiPart` can take `File`, `InputStream`, `byte[]`, or even `String`.
