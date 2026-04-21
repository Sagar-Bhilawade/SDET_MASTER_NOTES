# 00: API Foundations

### 1. Summary of Why (Architectural Context)
Testing an API is the verification of **System Integrity** and **Contract Adherence**. Beyond just status codes, we focus on REST constraints and the Transport Layer to ensure the system handles edge cases like race conditions and stateless scaling.
- **Statelessness:** The server stores no client context. This allows for horizontal scaling. In automation, every request MUST be self-contained (Auth headers, Correlation IDs).
- **Idempotency & Safety:** 
    - **Safe:** Methods that don't change resource state (`GET`, `HEAD`, `OPTIONS`).
    - **Idempotent:** Methods that can be called N times with the same result (`PUT`, `DELETE`). 
    - **Application:** A `DELETE` framework check should expect `200/204` on the first call, and while subsequent calls return `404`, the *server state* (the resource being gone) remains identical.

### 2. Clean Code (Production-Ready Basics)
Understanding the "Anatomy of a Request" is the baseline for robust framework parameterization.

- **URI vs. URL vs. URN:** 
    - **URI:** The generic identifier.
    - **URL:** The location (`https://api.dev.com`).
    - **URN:** The name (`urn:isbn:0451450523`).
    - **Automation Strategy:** Always parameterize the `BaseURI` separately from the `Resource Path`. This allows the framework to switch environments (QA/UAT/Prod) without modifying script logic.

- **Status Code Standard Mapping:**

| Range | Category | Key Focus |
| :--- | :--- | :--- |
| **201** | Created | Immediate verification of the resource in the Database. |
| **204** | No Content | Standard for `DELETE` or `PATCH` where no payload return is needed. |
| **401 vs 403** | Auth | `401` = Unauthenticated (Token missing/expired). `403` = Forbidden (No permissions). |
| **422** | Unprocessable Entity | Business logic failure (e.g., valid JSON but invalid data rules). |
| **502/504** | Infrastructure | Gateway issues or timeouts, usually out of scope for app code debugging. |

### 3. The "Industry Gap" (Missing in Documentation)
Standard tutorials omit the metadata that drives production observability.
- **X-Correlation-ID:** Essential in microservices. Every automation request should generate and send a unique UUID in this header to allow log tracing across Splunk/ELK.
- **Cache-Control:** Use `no-cache` in scripts to bypass CDNs/Proxies, ensuring the test validates the actual Origin Server and Database state.
- **Content-Type vs. Accept:** `Content-Type` is what you *send*; `Accept` is what you *expect* back. Mismatches here often trigger `415 Unsupported Media Type` or `406 Not Acceptable`.

### 4. The Gotcha (Senior Insight)
- **The PUT vs. PATCH Integrity Risk:** Using `PUT` requires the full object. Sending a partial payload via `PUT` can result in the server overwriting missing fields with `null`. Always perform a `GET` after update operations to verify field-level integrity.
- **Query Params in Logs:** Never put PII (Passwords, Emails) in Query Params. They are logged in plain text by Nginx/Apache. Use the Request Body or Headers.
- **GET Body Trap:** While the HTTP spec doesn't strictly forbid a body in a `GET`, many load balancers and servers ignore it. Never design automation relying on a `GET` payload.
- **Interview Trap (PUT Idempotency):** Most candidates think `PUT` is just for "Update." It can also be "Create if not exists." If you call `PUT` twice with the same ID and data, the result must be the same (Idempotent).