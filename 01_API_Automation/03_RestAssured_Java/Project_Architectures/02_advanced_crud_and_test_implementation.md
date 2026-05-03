# 02: Advanced CRUD and Test Implementation

### 1. 🚀 Tutorial Overview
Building on the **Layered Architecture**, this tutorial focuses on the full CRUD (Create, Read, Update, Delete) lifecycle implementation for the `User` module. We transition from simple POST requests to a robust test suite that verifies state changes across the API.

- **Objective:** Master path parameter handling, payload updates, and status code verification.
- **API Target:** Swagger Petstore User Endpoints (`/user/{username}`).
- **Focus:** `UserTests.java` and `UserEndpoints.java` refinements.

---

### 2. 📌 Core Theoretical Pillars

#### A. Path Parameter Abstraction
In RestAssured, dynamic URLs like `/user/{username}` should never be hardcoded. We use `.pathParam()` to map Java variables to URL template placeholders, ensuring the endpoint layer remains clean and reusable.

#### B. The "Update" Strategy (PUT vs PATCH)
For the Petstore API, we use **PUT** to replace/update the user. The strategy involves:
1. Creating a user.
2. Modifying the existing POJO payload.
3. Sending the updated POJO to the same `username` resource.

#### C. Verification of Deletion
A DELETE request is only considered successful if:
1. The DELETE call returns `200 OK`.
2. A subsequent GET call returns `404 Not Found`.

---

### 3. 🛠️ Step-by-Step Implementation

#### Phase 1: Refining the Endpoints Layer
We expand `UserEndpoints.java` to include the remaining CRUD operations.

```java
// api.endpoints.UserEndpoints.java
public class UserEndpoints {

    // 1. Read User (GET)
    public static Response readUser(String userName) {
        Response response = given()
                .pathParam("username", userName)
        .when()
                .get(Routes.get_url);
        
        return response;
    }

    // 2. Update User (PUT)
    public static Response updateUser(String userName, User payload) {
        Response response = given()
                .contentType(ContentType.JSON)
                .accept(ContentType.JSON)
                .pathParam("username", userName)
                .body(payload)
        .when()
                .put(Routes.update_url);
        
        return response;
    }

    // 3. Delete User (DELETE)
    public static Response deleteUser(String userName) {
        Response response = given()
                .pathParam("username", userName)
        .when()
                .delete(Routes.delete_url);
        
        return response;
    }
}
```

#### Phase 2: Implementing the Test Layer
The `UserTests.java` class orchestrates the calls and performs assertions.

```java
// api.tests.UserTests.java
public class UserTests {
    User userPayload;

    @BeforeClass
    public void setupData() {
        // Using Faker to generate unique data for each run
        Faker faker = new Faker();
        userPayload = new User();
        userPayload.setId(faker.idNumber().hashCode());
        userPayload.setUsername(faker.name().username());
        userPayload.setFirstName(faker.name().firstName());
        // ... set other fields
    }

    @Test(priority = 1)
    public void testPostUser() {
        Response response = UserEndpoints.createUser(userPayload);
        Assert.assertEquals(response.getStatusCode(), 200);
    }

    @Test(priority = 2)
    public void testGetUserByName() {
        Response response = UserEndpoints.readUser(this.userPayload.getUsername());
        response.then().log().all();
        Assert.assertEquals(response.getStatusCode(), 200);
    }

    @Test(priority = 3)
    public void testUpdateUserByName() {
        // Update data using payload
        userPayload.setFirstName(faker.name().firstName());
        userPayload.setLastName(faker.name().lastName());

        Response response = UserEndpoints.updateUser(this.userPayload.getUsername(), userPayload);
        Assert.assertEquals(response.getStatusCode(), 200);

        // Checking data after update
        Response responseAfterUpdate = UserEndpoints.readUser(this.userPayload.getUsername());
        Assert.assertEquals(responseAfterUpdate.getStatusCode(), 200);
    }

    @Test(priority = 4)
    public void testDeleteUserByName() {
        Response response = UserEndpoints.deleteUser(this.userPayload.getUsername());
        Assert.assertEquals(response.getStatusCode(), 200);
    }
}
```

---

### 4. 📊 Technical Component Encyclopedia

| Component | Responsibility | Pattern Used |
| :--- | :--- | :--- |
| **PathParams** | Mapping Java strings to `{username}` tokens. | Tokenization |
| **Priority** | Controlling execution order (Create -> Read -> Update -> Delete). | TestNG @Test(priority=n) |
| **State Sharing** | Using class-level `userPayload` to pass data between methods. | Context Sharing |

---

### 5. 🏭 Real-World Industrial Patterns

#### A. The "Chain Validation" Pattern
Instead of just checking the status code of the Update request, always follow up with a GET request to verify that the database actually reflects the changes. This is "Deep Validation."

#### B. Dynamic Payload Modification
Instead of creating a new POJO for updates, modify the existing one. This ensures that only the intended fields change while maintaining the integrity of the object.

```java
// Modification logic within test method
userPayload.setEmail(faker.internet().emailAddress());
Response response = UserEndpoints.updateUser(userName, userPayload);
```

---

### 🛡️ Content Preservation Note
- Ensure `Routes.java` contains all four URLs (POST, GET, PUT, DELETE).
- Path parameters in `given()` must match the curly brace placeholders in `Routes.java`.
