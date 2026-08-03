# RestAssured - POJOs, Serialization & Deserialization

## What This Covers

Converting Java objects to JSON and back — POJOs, Jackson, and `ObjectMapper`.

**This is the most-asked RestAssured depth question.** Interviewers use it specifically because it separates people who built an API framework from people who sent requests with string bodies.

---

## Why It Matters (Interview Context)

Expect these almost verbatim:

> "What's serialization and deserialization in RestAssured?"
> "What's a POJO and why use one?"
> "What does `ObjectMapper` do?"

The reason it's such a reliable filter: sending a JSON string body works fine for simple cases, so plenty of people never move past it. Using POJOs means you designed for maintainability.

---

## Your 30-Second Interview Answer

> "A POJO is a plain Java class representing the request or response shape. Serialization is converting that object into JSON for the request body; deserialization is converting the JSON response back into a typed object. RestAssured does it automatically via Jackson if it's on the classpath. The reason it matters is that string-based JSON puts your field names in string literals — a typo compiles fine and fails at runtime. With POJOs the IDE autocompletes and the compiler catches it."

---

## Your 2-Minute Deep Answer

> "**The problem POJOs solve:**
>
> ```java
> // ❌ String-based — brittle
> String body = "{ \"email\": \"test@example.com\", \"role\": \"admin\" }";
>
> given().body(body).when().post("/users");
>
> String role = response.jsonPath().getString("role");   // string literal again
> ```
>
> Every field name is a string literal. A typo — `"rol"` instead of `"role"` — compiles perfectly and fails at runtime with a null. Escaping quotes makes it unreadable. And when the API contract changes, nothing tells you which tests are affected; you find out when they fail.
>
> **With a POJO:**
>
> ```java
> public class User {
>     private String email;
>     private String role;
>     private String name;
>
>     public User() {}                         // no-arg constructor — Jackson needs it
>
>     public String getEmail() { return email; }
>     public void setEmail(String email) { this.email = email; }
>     // ... remaining getters/setters
> }
> ```
>
> ```java
> // Serialization — object → JSON
> User user = new User();
> user.setEmail("test@example.com");
> user.setRole("admin");
>
> given().spec(spec)
>     .body(user)                              // RestAssured serializes automatically
> .when().post("/users");
>
> // Deserialization — JSON → object
> User created = given().spec(spec)
>     .when().get("/users/1")
>     .then().extract().as(User.class);
>
> assertThat(created.getEmail()).isEqualTo("test@example.com");   // compiler-checked
> ```
>
> `extract().as(User.class)` is the one-liner worth remembering.
>
> ---
>
> **How it actually works:**
>
> RestAssured delegates to **Jackson** (or GSON) if it's on the classpath. When you pass an object to `.body()` with content type JSON, it calls Jackson's `ObjectMapper` to serialize. On extraction it does the reverse.
>
> `ObjectMapper` is the Jackson class doing the conversion:
>
> ```java
> ObjectMapper mapper = new ObjectMapper();
> String json = mapper.writeValueAsString(user);       // serialize
> User user = mapper.readValue(json, User.class);      // deserialize
> ```
>
> You mostly don't call it directly with RestAssured, but knowing it's Jackson underneath is exactly what the interview question is checking.
>
> ---
>
> **Jackson annotations that matter:**
>
> ```java
> public class User {
>     @JsonProperty("user_email")              // JSON name differs from field name
>     private String email;
>
>     @JsonIgnore                               // exclude from serialization
>     private String internalNote;
>
>     @JsonInclude(JsonInclude.Include.NON_NULL)   // omit nulls
>     private String middleName;
>
>     @JsonFormat(pattern = "yyyy-MM-dd")
>     private Date createdAt;
> }
> ```
>
> **And the one that saves the most pain:**
>
> ```java
> @JsonIgnoreProperties(ignoreUnknown = true)
> public class User { }
> ```
>
> Without this, Jackson throws `UnrecognizedPropertyException` if the API returns a field your POJO doesn't declare. So the day the API adds a field — a backwards-compatible change that should break nothing — your entire test suite fails.
>
> That's a real incident pattern, and being able to name it demonstrates you've run this in production rather than read about it. It's the same principle as `toMatchObject` over `toEqual` in the Playwright notes: tolerate additions, catch removals.
>
> ---
>
> **Nested objects and collections:**
>
> ```java
> public class Order {
>     private String orderId;
>     private User customer;                    // nested POJO
>     private List<LineItem> items;             // collection of POJOs
> }
> ```
>
> Jackson handles nesting automatically. For a list at the top level of a response:
>
> ```java
> List<User> users = response.jsonPath().getList(".", User.class);
> ```
>
> ---
>
> **Builder pattern — worth mentioning:**
>
> ```java
> User user = User.builder()
>     .email("test@example.com")
>     .role("admin")
>     .build();
> ```
>
> With Lombok's `@Builder`, or hand-written. This is the Java equivalent of the factory pattern from the Playwright notes — the test states only the fields it cares about, and defaults fill the rest. Lombok's `@Data` also removes the getter/setter boilerplate entirely, which is most of what makes POJOs tedious to write.
>
> ---
>
> **The honest trade-off:**
>
> POJOs are more upfront work. For a handful of simple endpoints, a JSON string or a `Map<String, Object>` is genuinely fine, and I wouldn't over-engineer a small suite.
>
> They pay off when the API surface is large, when contracts change, or when the same shapes are reused across many tests. At that point the compile-time checking and IDE support are worth substantially more than the boilerplate — especially with Lombok removing most of it."

---

## Serialization Reference

| Direction | Term | RestAssured |
|-----------|------|-------------|
| Object → JSON | Serialization | `.body(pojo)` |
| JSON → Object | Deserialization | `.extract().as(Pojo.class)` |
| Engine | — | Jackson `ObjectMapper` |

| Annotation | Purpose |
|-----------|---------|
| `@JsonProperty` | Map differing field name |
| `@JsonIgnore` | Exclude field |
| `@JsonInclude(NON_NULL)` | Omit nulls |
| `@JsonIgnoreProperties(ignoreUnknown=true)` | **Survive new API fields** |
| `@JsonFormat` | Date formatting |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "We used POJOs for request and response bodies rather than string JSON. The practical difference is compile-time safety — with string-based JSON your field names are string literals, so a typo compiles fine and fails at runtime with a null that takes a while to trace. With a POJO the IDE autocompletes and the compiler catches it.
>
> Serialization is the object going out as the request body; deserialization is the response coming back as a typed object via `extract().as(User.class)`. RestAssured delegates to Jackson's `ObjectMapper` underneath.
>
> The annotation I'd single out is `@JsonIgnoreProperties(ignoreUnknown = true)`. Without it, Jackson throws when the API returns a field the POJO doesn't declare — so the day a backend team adds a field, which is a backwards-compatible change that should break nothing, your entire suite fails. That's a real incident pattern and it's a one-line fix.
>
> It's the same principle I apply in Playwright with `toMatchObject` over exact matching: tolerate additions, catch removals. Different stack, identical reasoning.
>
> I'd also say POJOs aren't automatic. For a handful of simple endpoints a Map or a JSON string is fine. They pay off when the API surface is large or contracts change — and Lombok removes most of the boilerplate that makes them tedious."

---

## Common Interview Questions & Answers

### Q1: "What are serialization and deserialization in RestAssured?"

**A:** "Serialization is converting a Java object into JSON for the request body — you pass the POJO to `.body()` and RestAssured handles it. Deserialization is the reverse, converting the JSON response into a typed object via `extract().as(User.class)`. Jackson's `ObjectMapper` does the actual conversion if Jackson is on the classpath."

### Q2: "What's a POJO and why use one instead of a JSON string?"

**A:** "A plain Java class representing the request or response shape — fields, a no-arg constructor, getters and setters. The reason over strings is compile-time safety: with string JSON your field names are literals, so a typo compiles and fails at runtime. With a POJO the IDE autocompletes and the compiler catches it. It also makes contract changes visible at build time rather than at run time."

### Q3: "What does `ObjectMapper` do?"

**A:** "It's Jackson's conversion class — `writeValueAsString` to serialize, `readValue` to deserialize. RestAssured calls it under the hood, so you rarely use it directly, but it's what's actually doing the work."

### Q4: "Your tests suddenly all fail after a backend deploy that only added a field. Why?"

**A:** "Missing `@JsonIgnoreProperties(ignoreUnknown = true)`. Jackson throws `UnrecognizedPropertyException` when the response contains a field the POJO doesn't declare. Adding a field is backwards-compatible and shouldn't break anything, so that annotation should be on every response POJO by default."

### Q5: "Are POJOs always worth it?"

**A:** "No. For a few simple endpoints a `Map<String, Object>` or a JSON string is fine and I wouldn't over-engineer. They pay off with a large API surface, changing contracts, or shapes reused across many tests. Lombok's `@Data` and `@Builder` remove most of the boilerplate, which shifts the calculation."

---

## Weak Areas to Watch

❌ **Don't use string JSON bodies in a real framework**
→ Field names as literals; typos fail at runtime.

❌ **Don't omit `@JsonIgnoreProperties(ignoreUnknown = true)`**
→ A new API field breaks the whole suite.

❌ **Don't forget the no-arg constructor**
→ Jackson needs it to instantiate during deserialization.

❌ **Don't claim POJOs are always right**
→ Over-engineering a small suite is a real cost.

---

## Key Takeaway for Interview

This is the RestAssured question most likely to separate you.

The clean definition:

> **"Serialization is object to JSON for the request; deserialization is JSON to a typed object on the response. Jackson's `ObjectMapper` does it, and RestAssured calls it automatically."**

And the production detail that proves real usage:

> **"`@JsonIgnoreProperties(ignoreUnknown = true)` on every response POJO. Without it, the day the backend adds a field — a backwards-compatible change — Jackson throws and your entire suite fails."**

---

## Checklist

✅ POJOs for request and response bodies
✅ No-arg constructor present
✅ `@JsonIgnoreProperties(ignoreUnknown = true)` on response POJOs
✅ `@JsonProperty` where JSON names differ
✅ `extract().as(Pojo.class)` for deserialization
✅ Lombok `@Data` / `@Builder` to cut boilerplate
✅ Can name Jackson as the engine

---

**Remember:** Tolerate added fields, catch removed ones. Same rule in every stack. 🎯
