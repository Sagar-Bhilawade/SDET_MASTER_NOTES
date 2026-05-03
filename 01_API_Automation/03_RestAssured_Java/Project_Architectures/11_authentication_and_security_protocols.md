# 11: Authentication and Security Protocols

### 1. 🚀 Tutorial Overview
Real-world APIs are secured to protect data. As an architect, you must implement a mechanism that handles security tokens (JWT, OAuth2, API Keys) seamlessly across all requests without duplicating code in every test.

- **Objective:** Implement a global authentication strategy.
- **Security Types:** Bearer Tokens (JWT), OAuth 2.0, and API Keys.
- **Repository Pattern:** Handling tokens in the `Endpoints` layer and `BaseTest`.

---

### 2. 📌 Core Theoretical Pillars

#### A. Bearer Token (JWT) Lifecycle
A JSON Web Token (JWT) is typically obtained via a "Login" or "Token" endpoint. Once received, it must be included in the `Authorization` header as `Bearer <token>` for all subsequent calls.

#### B. OAuth 2.0 Flows
- **Client Credentials:** Used for machine-to-machine auth (App-to-App). You exchange a `Client ID` and `Client Secret` for an access token.
- **Authorization Code:** Used for user-to-app auth. It involves a redirection step, but for automation, we often bypass this using a "Refresh Token" strategy.

#### C. API Key Management
API Keys are simpler strings passed in headers (e.g., `x-api-key`) or query parameters. Unlike tokens, they rarely expire.

---

### 3. 🛠️ Implementation in the Repository

#### Phase 1: Storing Credentials in `config.properties`
Never hardcode keys. Use the property management system we built in Part 06.

```properties
# config.properties
app_id=my_client_id
app_secret=my_client_secret
api_key=sk_test_51Mz...
auth_url=https://auth.api.com/v1/token
```

#### Phase 2: Dynamic Token Generation in `BaseTest`
We generate the token once before any tests run and store it in a static variable.

```java
// api.base.BaseTest.java
public class BaseTest {
    public static String bearerToken;

    @BeforeSuite
    public void getAccessToken() {
        // Fetching credentials from properties
        String clientID = Routes.getURL().getString("app_id");
        String secret = Routes.getURL().getString("app_secret");

        // OAuth2 Client Credentials Flow
        Response response = given()
                .auth().preemptive().basic(clientID, secret)
                .formParam("grant_type", "client_credentials")
            .when()
                .post(Routes.getURL().getString("auth_url"));

        bearerToken = response.jsonPath().getString("access_token");
    }
}
```

#### Phase 3: Passing the Token in Endpoints
Update your Endpoints class to accept or automatically inject the token.

```java
// api.endpoints.UserEndpoints.java
public class UserEndpoints {

    public static Response createUser(User payload) {
        return given()
                .header("Authorization", "Bearer " + BaseTest.bearerToken) // JWT/OAuth2
                .header("x-api-key", Routes.getURL().getString("api_key")) // API Key
                .contentType(ContentType.JSON)
                .body(payload)
            .when()
                .post(Routes.post_url);
    }
}
```

---

### 4. 📊 Handling Multiple Users & Roles

In advanced testing, you might need to test scenarios between an **Admin** and a **Standard User**.

#### Strategy: The "User Store" Pattern
Create a class to hold different user contexts.

```java
public class UserContext {
    public String name;
    public String token;
    public String role;

    public UserContext(String name, String token, String role) {
        this.name = name;
        this.token = token;
        this.role = role;
    }
}
```

#### Implementation in Tests:
```java
@Test
public void testAdminAccessToUser() {
    UserContext admin = AuthManager.getAdminUser();
    UserContext guest = AuthManager.getGuestUser();

    // 1. Admin deletes guest (Should pass 200)
    Response resp1 = UserEndpoints.deleteUser(guest.name, admin.token);
    Assert.assertEquals(resp1.getStatusCode(), 200);

    // 2. Guest tries to delete Admin (Should fail 403)
    Response resp2 = UserEndpoints.deleteUser(admin.name, guest.token);
    Assert.assertEquals(resp2.getStatusCode(), 403);
}
```

---

### 5. 🏗️ Industrial Best Practices

#### A. Token Expiry Handling
For long-running suites, tokens might expire. Implement a "Retry" filter that checks for a `401 Unauthorized` and automatically refreshes the token before retrying the original request.

#### B. Environment Variables for CI/CD
While local dev uses `.properties`, CI/CD (Jenkins/GitHub Actions) should inject keys via **Environment Variables**.
```java
// Priority: System Env > Property File
String apiKey = System.getenv("API_KEY") != null ? 
               System.getenv("API_KEY") : 
               Routes.getURL().getString("api_key");
```

---

### 🛡️ Content Preservation Note
- Use `auth().oauth2(token)` for a cleaner RestAssured syntax compared to manual headers.
- For API Keys in query params, use `.queryParam("api_key", key)`.
