# 10: Authentication and Authorization in RestAssured

### 1. 🚀 Tutorial Overview
Securing APIs is the most critical aspect of modern web services. Before a test can interact with a resource, it must prove its identity and permissions. This tutorial covers the fundamental difference between **Authentication** and **Authorization** and provides a comprehensive guide to implementing every major security scheme supported by RestAssured.

- **Purpose:** To secure API requests and simulate different user privilege levels.
- **Context:** Handled in the `Given` phase using the `.auth()` method or via custom headers.
- **Prerequisites:** Understanding of HTTP headers and query parameters.

---

### 2. 📌 Core Theoretical Pillars

#### A. Authentication vs. Authorization
- **Authentication (AuthN):** *"Who are you?"* (Identity).
- **Authorization (AuthZ):** *"What are you allowed to do?"* (Permissions).

#### B. The "Big Three" Legacy Auth Types
1. **Basic Auth:**
   - **Mechanism:** Combines `username:password`, encodes it in **Base64**, and sends it in the `Authorization` header.
   - **Security:** Low. Base64 is NOT encryption; it is easily reversible. Must always be used with HTTPS.
   - **Header:** `Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=`

2. **Preemptive vs. Challenged:**
   - **Challenged (Default):** The client sends a request without credentials $\rightarrow$ Server responds with `401 Unauthorized` and a `WWW-Authenticate` header $\rightarrow$ Client sends request again with credentials.
   - **Preemptive:** The client "pre-empts" the challenge and sends credentials in the very first request. This is faster (fewer network hops).

3. **Digest Auth:**
   - **Mechanism:** More secure than Basic. The server sends a "nonce" (a random string). The client hashes the password, the nonce, and other details together.
   - **Security:** Higher. The actual password is never sent over the network, only the hash.

#### C. Modern Auth (Bearer & OAuth)
1. **Bearer Token:**
   - **Analogy:** Like a movie ticket. The usher (server) doesn't care who you are; they only care if you "bear" a valid ticket.
   - **Implementation:** Usually a **JWT (JSON Web Token)**.
   - **Header:** `Authorization: Bearer <token_string>`

2. **OAuth 2.0:**
   - **Mechanism:** A delegation framework. Instead of sharing your password with a 3rd party app, you give it an "Access Token" with limited scope (e.g., "read-only").

---

### 3. 🛠️ Step-by-Step Chronological Implementation

#### Phase 1: Basic and Digest Authentication
Used primarily for legacy systems or simple internal tools.

```java
// 1. Basic Auth (Base64 Encoded)
given()
    .auth().basic("username", "password")
.when()
    .get("/basic-auth");

// 2. Preemptive Basic Auth (Saves a request)
given()
    .auth().preemptive().basic("admin", "admin")
.when()
    .get("/admin-panel");

// 3. Digest Auth (Uses hashing for better security than Basic)
given()
    .auth().digest("user", "pass")
.when()
    .get("/digest-auth");
```

#### Phase 2: Token-Based Authentication (Bearer & API Key)
The standard for modern REST APIs and cloud services.

```java
// 1. Bearer Token (JWT)
// Often passed as a header: Authorization: Bearer <token>
given()
    .header("Authorization", "Bearer my_jwt_token_123")
.when()
    .get("/user/profile");

// 2. API Key (Query Parameter)
// Common in Public APIs (e.g., OpenWeatherMap)
given()
    .queryParam("appid", "my_secret_api_key")
.when()
    .get("/weather");
```

#### Phase 3: OAuth 1.0 and OAuth 2.0
Used for high-security integrations (Google, GitHub, Twitter).

```java
// OAuth 2.0 (The modern standard)
// Usually requires just the access token
given()
    .auth().oauth2("access_token_here")
.when()
    .get("/github/repos");

// OAuth 1.0 (Complex, requires 4 parameters)
given()
    .auth().oauth("consumerKey", "consumerSecret", "accessToken", "secretToken")
.when()
    .get("/twitter/feed");
```

---

### 4. 📊 Technical Component Encyclopedia

| Auth Type | DSL Method | Description | Implementation Detail |
| :--- | :--- | :--- | :--- |
| **Basic** | `.auth().basic()` | Standard credentials. | Base64 encoded in header. |
| **Preemptive** | `.auth().preemptive()` | Forces credentials. | Sends `Authorization` header immediately. |
| **Digest** | `.auth().digest()` | Hashed credentials. | Prevents replay attacks. |
| **Bearer** | `.header()` | JWT/OAuth Token. | Sent as `Authorization: Bearer <token>`. |
| **API Key** | `.queryParam()` | Fixed string. | Passed in URL `?api_key=xyz`. |
| **OAuth 2.0** | `.auth().oauth2()` | Token-based AuthZ. | Most widely used modern standard. |

---

### 5. 🔄 Visual Workflow (Mermaid)

```mermaid
graph TD
    A[Start Request] --> B{Choose Auth Type}
    B -->|Basic/Digest| C[Set User/Pass]
    B -->|Bearer| D[Add Authorization Header]
    B -->|API Key| E[Add Query Param]
    B -->|OAuth 2.0| F[Set Access Token]
    
    C --> G[Request Sent]
    D --> G
    E --> G
    F --> G
    
    G --> H{Server Response}
    H -->|401| I[Authentication Failed]
    H -->|403| J[Authorization Failed]
    H -->|200| K[Success]
```

---

### 6. 🏭 Real-World Industrial Patterns

#### Pattern: Centralized Auth Provider
In frameworks, never hardcode tokens. Use a utility to fetch/rotate tokens.

```java
public class AuthUtils {
    public static String getValidToken() {
        // Logic to fetch from Auth Server or Config
        return "dynamic_token_from_vault";
    }
}

// In Test
given()
    .auth().oauth2(AuthUtils.getValidToken())
.when()
    .get("/secure-resource");
```

---

### 🛡️ Content Preservation Note
- Authentication methods are usually configured in a `RequestSpecification` (see **05: Core DSL Syntax**) to avoid repetition across tests.
- Always use **Preemptive Auth** when you know the server requires Basic Auth to minimize latency.
