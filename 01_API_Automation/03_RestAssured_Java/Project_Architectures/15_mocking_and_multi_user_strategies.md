# 15: Mocking and Multi-User Strategies

### 1. 🚀 Tutorial Overview
The most advanced stage of API automation involves testing in isolation and handling complex user state. **Mocking** allows you to simulate API behavior when the real server is unavailable, while **Multi-User Strategies** ensure your framework can handle concurrency and role-based access control (RBAC).

- **Objective:** decouple tests from live environments and handle user sessions.
- **Mocking:** Introduction to WireMock.
- **Concurrency:** Handling multiple user roles in a single suite.

---

### 2. 📌 Core Theoretical Pillars

#### A. Service Virtualization (Mocking)
Why mock?
1. **Independence:** Test your framework logic even if the Dev environment is down.
2. **Deterministic Data:** Force the API to return a specific `500 Internal Server Error` or a rare `429 Too Many Requests` for edge-case testing.
3. **Speed:** Local mocks are faster than network-bound APIs.

#### B. User Session Management
In professional suites, you often need to simulate interactions between different roles:
- **Admin:** Can create, edit, and delete.
- **Moderator:** Can only edit and delete.
- **User:** Can only read.

---

### 3. 🛠️ Implementation Deep Dive

#### Phase 1: Setting up WireMock
Add the dependency to `pom.xml`.

```xml
<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock-jre8</artifactId>
    <version>2.35.0</version>
    <scope>test</scope>
</dependency>
```

#### Phase 2: Creating a Mock Server
```java
// api.utils.MockServer.java
public class MockServer {
    public static WireMockServer server;

    public static void start() {
        server = new WireMockServer(8089);
        server.start();
        
        // Define a "Stub" (Fake response)
        server.stubFor(get(urlEqualTo("/mock/user"))
                .willReturn(aResponse()
                        .withStatus(200)
                        .withHeader("Content-Type", "application/json")
                        .withBody("{\"status\": \"mocked_success\"}")));
    }

    public static void stop() {
        server.stop();
    }
}
```

#### Phase 3: Handling Multi-User Contexts
Use a `Map` or `ThreadLocal` to store session tokens for different users to prevent state leakage during parallel execution.

```java
// api.utils.SessionManager.java
public class SessionManager {
    private static Map<String, String> userTokens = new HashMap<>();

    public static void login(String role, String username, String password) {
        Response resp = given()
                .formParam("user", username)
                .formParam("pass", password)
                .post("/login");
        
        userTokens.put(role, resp.jsonPath().getString("token"));
    }

    public static String getToken(String role) {
        return userTokens.get(role);
    }
}
```

#### Phase 4: Role-Based Test Case
```java
@Test
public void testRBAC() {
    // Standard User tries to delete (Should fail)
    given()
        .header("Authorization", "Bearer " + SessionManager.getToken("standard_user"))
    .when()
        .delete("/admin/resource/1")
    .then()
        .statusCode(403);
}
```

---

### 4. 📊 Technical Component Encyclopedia

| Strategy | Responsibility | Tool |
| :--- | :--- | :--- |
| **Stubbing** | Returning a fixed response for a fixed request. | WireMock |
| **RBAC Testing** | Verifying permissions across different user roles. | SessionManager |
| **Parallel Execution** | Running multiple users at the same time. | TestNG `parallel="methods"` |
| **Stateful Testing** | Passing IDs/Tokens from one user's action to another. | Context Objects |

---

### 5. 🏭 Real-World Industrial Patterns

#### A. The "Golden Payload" Mock
Instead of hardcoding the body in Java, WireMock can read "Mapping files" (JSON). This allows you to store entire "Golden" response payloads in `src/test/resources/mappings`.

#### B. Dynamic Port Allocation
In CI/CD, port `8089` might be busy. Use `WireMockConfiguration.options().dynamicPort()` to let WireMock find an available port and then pass that port to your `BaseTest`.

---

### 🛡️ Final Architectural Milestone
By completing this module, you have transitioned from a **Script Developer** to a **Senior SDET Architect**. You now possess the knowledge to:
1. Build scalable, layered frameworks.
2. Handle complex security and data flows.
3. Integrate reporting, logging, and CI/CD.
4. Test in isolation using Mocking.

---

### 🛡️ Content Preservation Note
- Always call `server.stop()` in `@AfterSuite` to avoid port binding errors in subsequent runs.
- For parallel execution, ensure your `UserContext` or `SessionManager` uses `ThreadLocal` to keep tokens separate for each thread.
