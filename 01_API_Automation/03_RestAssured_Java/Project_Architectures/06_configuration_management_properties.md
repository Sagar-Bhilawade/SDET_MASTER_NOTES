# 06: Configuration Management with Properties Files

### 1. 🚀 Tutorial Overview
As a framework matures, hardcoding URLs in a Java class (`Routes.java`) becomes a bottleneck. To make the framework truly portable and environment-agnostic, we externalize all configurations into **Properties Files**. This allows us to change environment URLs without recompiling the Java code.

- **Objective:** Move from static `Routes.java` to dynamic `.properties` loading.
- **Tooling:** `ResourceBundle` class (Native Java) and `.properties` files.
- **Refactoring:** Creating `UserEndpoints2.java` to demonstrate property-based endpoint management.

---

### 2. 📌 Core Theoretical Pillars

#### A. Externalization of Environment Data
Configurations like `base_url`, `port`, or `timeout` vary between Local, Dev, QA, and Prod environments. Storing them in a `.properties` file (Key-Value pairs) follows the **Twelve-Factor App** methodology for configuration.

#### B. ResourceBundle vs Properties Class
While `java.util.Properties` is common, `java.util.ResourceBundle` is a more modern and cleaner approach for reading configuration files located in `src/test/resources`. It automatically handles the file extension and pathing.

#### C. Single Point of Failure (SPOF) Prevention
By using property files, we prevent the risk of a developer accidentally changing a URL in the source code. The `.properties` file acts as the single source of truth for the environment state.

---

### 3. 🛠️ Step-by-Step Implementation

#### Phase 1: Creating the Properties File
Place this file in the `src/test/resources` folder.

```properties
# routes.properties
# User Module Endpoints
base_url=https://petstore.swagger.io/v2
post_url=https://petstore.swagger.io/v2/user
get_url=https://petstore.swagger.io/v2/user/{username}
update_url=https://petstore.swagger.io/v2/user/{username}
delete_url=https://petstore.swagger.io/v2/user/{username}
```

#### Phase 2: Implementing the Property Loader
Instead of a class with static strings, we create a utility to fetch values dynamically.

```java
// api.endpoints.Routes.java (Refactored or used within Endpoints)
public class Routes {
    // This utility method will load the properties file
    public static ResourceBundle getURL() {
        // "routes" is the name of the file (routes.properties)
        ResourceBundle routes = ResourceBundle.getBundle("routes");
        return routes;
    }
}
```

#### Phase 3: Creating UserEndpoints2 (Property-Based)
We create a new version of the endpoints class to use the dynamic properties.

```java
// api.endpoints.UserEndpoints2.java
public class UserEndpoints2 {

    public static Response createUser(User payload) {
        // Fetching URL from properties file
        String post_url = Routes.getURL().getString("post_url");

        return given()
                .contentType(ContentType.JSON)
                .accept(ContentType.JSON)
                .body(payload)
            .when()
                .post(post_url);
    }

    public static Response getUser(String userName) {
        String get_url = Routes.getURL().getString("get_url");

        return given()
                .pathParam("username", userName)
            .when()
                .get(get_url);
    }

    // ... similarly for update and delete using getString("update_url") and getString("delete_url")
}
```

#### Phase 4: Updating Test Classes
Simply point your `UserTests.java` to use `UserEndpoints2` instead of `UserEndpoints`.

```java
// api.tests.UserTests.java
public void testPostUser() {
    // Switching to the property-based implementation
    Response response = UserEndpoints2.createUser(userPayload);
    Assert.assertEquals(response.getStatusCode(), 200);
}
```

---

### 4. 📊 Technical Component Encyclopedia

| Component | Responsibility | Technical Implementation |
| :--- | :--- | :--- |
| **routes.properties** | Stores all endpoint mappings. | Key-Value Pairs |
| **ResourceBundle** | Reads the properties file from the classpath. | `getBundle("filename")` |
| **UserEndpoints2** | Orchestrates HTTP calls using dynamic URLs. | `getString("key")` |

---

### 5. 🏭 Real-World Industrial Patterns

#### A. Multiple Environment Support
You can create `routes_qa.properties` and `routes_dev.properties`. By passing a system variable (e.g., `-Denv=qa`), you can programmatically decide which bundle to load:
```java
String env = System.getProperty("env");
ResourceBundle.getBundle("routes_" + env);
```

#### B. Avoiding Hardcoded Keys
In very large frameworks, developers use an `Enum` to store property keys, preventing "Magic Strings" and reducing the risk of typos when calling `.getString("key_name")`.

---

### 🛡️ Content Preservation Note
- The `.properties` file MUST be in `src/test/resources` to be detected by `ResourceBundle`.
- Do not include the `.properties` extension when calling `getBundle()`.
