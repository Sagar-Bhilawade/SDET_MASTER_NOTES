# Java 8 Features for SDETs

## What This Covers

Lambdas, functional interfaces, `Optional`, method references, and default methods — the Java 8 feature set that gets asked directly and that you actually use in framework code.

---

## Why It Matters (Interview Context)

**"What Java 8 features do you use?"** is a standard question in any Java interview, and it's a fair one — Java 8 changed how the language is written, and someone still writing pre-8 Java is a signal.

For an SDET specifically, the connection is that **Selenium's own API depends on these**. `ExpectedCondition` is a functional interface, which is why you can pass a lambda to `wait.until()`. Knowing that isn't trivia — it's why custom waits work.

Streams are covered separately in `02_Collections_And_Streams`. This note is the rest.

---

## Your 30-Second Interview Answer

> "Lambdas, functional interfaces, Optional, method references, and default methods. The one most relevant to Selenium is functional interfaces — `ExpectedCondition` is one, which is exactly why you can pass a lambda to `wait.until()` for a custom wait condition. `Optional` is the one I'd highlight for framework code, because it makes 'this might not be there' explicit in the type rather than a null waiting to happen."

---

## Your 2-Minute Deep Answer

> "**Lambdas and functional interfaces — and the Selenium connection:**
>
> A **functional interface** has exactly one abstract method. That's what makes a lambda possible — the compiler knows which method you're implementing.
>
> ```java
> @FunctionalInterface
> public interface ExpectedCondition<T> extends Function<WebDriver, T> {
>     T apply(WebDriver driver);
> }
> ```
>
> `ExpectedCondition` is a functional interface. That's precisely why this works:
>
> ```java
> // Pre-Java 8 — anonymous inner class
> wait.until(new ExpectedCondition<Boolean>() {
>     public Boolean apply(WebDriver d) {
>         return d.findElements(By.cssSelector(".row")).size() > 5;
>     }
> });
>
> // Java 8 — lambda
> wait.until(d -> d.findElements(By.cssSelector(".row")).size() > 5);
> ```
>
> Same thing, six lines shorter. **Custom wait conditions are the everyday use of lambdas in a Selenium framework**, and being able to explain *why* it works — because `ExpectedCondition` has a single abstract method — is better than just knowing the syntax.
>
> **The built-in functional interfaces worth knowing:**
>
> ```java
> Predicate<T>       boolean test(T t)        // filtering
> Function<T,R>      R apply(T t)             // transforming
> Consumer<T>        void accept(T t)         // side effect, no return
> Supplier<T>        T get()                  // produces, takes nothing
> BiFunction<T,U,R>  R apply(T t, U u)        // two arguments
> ```
>
> `Supplier` is the one I use most in framework code — a retry helper takes a `Supplier<T>` so it can re-invoke the action:
>
> ```java
> public static <T> T retry(Supplier<T> action, int attempts) { ... }
>
> WebElement el = retry(() -> driver.findElement(locator), 3);
> ```
>
> ---
>
> **Method references — shorthand for a lambda that just calls something:**
>
> ```java
> .map(element -> element.getText())     // lambda
> .map(WebElement::getText)              // method reference
>
> .forEach(item -> System.out.println(item))
> .forEach(System.out::println)
> ```
>
> Four kinds: static (`Integer::parseInt`), instance-of-a-particular-object (`logger::info`), instance-of-an-arbitrary-object (`WebElement::getText`), and constructor (`ArrayList::new`).
>
> The third is the one people find confusing — `WebElement::getText` means *"call `getText()` on whichever element comes through"*, which is why it works in a stream.
>
> ---
>
> **`Optional` — the one worth caring about in framework code:**
>
> `Optional<T>` makes "this might be absent" explicit in the type signature rather than an implicit null.
>
> ```java
> Optional<WebElement> row = rows.stream()
>     .filter(r -> r.getText().contains("Sagar"))
>     .findFirst();
>
> row.ifPresent(WebElement::click);
>
> WebElement r = row.orElseThrow(
>     () -> new AssertionError("Row for 'Sagar' not found in table"));
> ```
>
> **`orElseThrow` with a meaningful message is the pattern I'd use in tests** — it turns a missing element into a clear assertion failure instead of a `NoSuchElementException` from the Optional itself, which tells the reader nothing about what was expected.
>
> ```java
> optional.isPresent()          // check
> optional.get()                // ❌ throws if empty — avoid
> optional.orElse(default)      // fallback value
> optional.orElseThrow(...)     // throw with context
> optional.map(fn)              // transform if present
> optional.ifPresent(fn)        // act if present
> ```
>
> **What to avoid:** `.get()` without checking. It defeats the entire purpose — you've reintroduced the null pointer with extra steps. And `Optional` is designed for return types, not for fields or parameters.
>
> ---
>
> **Default methods:**
>
> Interfaces can now have implementations:
>
> ```java
> public interface BasePage {
>     WebDriver getDriver();
>
>     default void waitForLoad() {
>         new WebDriverWait(getDriver(), Duration.ofSeconds(10))
>             .until(d -> ((JavascriptExecutor) d)
>                 .executeScript("return document.readyState").equals("complete"));
>     }
> }
> ```
>
> The reason they were added is **backwards compatibility** — it let Java add `stream()` to `Collection` without breaking every existing implementation. That's the answer to "why do default methods exist," and it's a better answer than "so interfaces can have code."
>
> Practically, it narrows the gap with abstract classes — but the distinction still holds: **interfaces can't have instance state.** That's why `BasePage` in a Selenium framework is an abstract class, not an interface: it needs to hold the driver and the wait.
>
> ---
>
> **`java.time` — worth a mention:**
>
> Selenium 4 uses `Duration` throughout:
>
> ```java
> new WebDriverWait(driver, Duration.ofSeconds(10));     // Selenium 4
> new WebDriverWait(driver, 10);                          // Selenium 3, deprecated
> ```
>
> If someone writes the second form in an interview, it dates their experience. The old `Date` and `Calendar` API was mutable and not thread-safe; `LocalDate`, `LocalDateTime`, and `Duration` replaced it.
>
> ---
>
> **The honest summary:**
>
> The features I genuinely use daily in framework code are lambdas for custom waits, `Supplier` for retry helpers, method references in streams, and `Optional` for lookups that might not find anything. That's a more credible answer than listing every Java 8 feature."

---

## Feature Reference

| Feature | Use in a test framework |
|---------|------------------------|
| **Lambdas** | Custom `ExpectedCondition` waits |
| **Functional interfaces** | Why `wait.until(d -> ...)` compiles |
| `Supplier<T>` | Retry helpers that re-invoke an action |
| `Predicate<T>` | Filtering collections |
| **Method references** | `WebElement::getText` in streams |
| **`Optional<T>`** | Lookups that may find nothing |
| **Default methods** | Shared behaviour without state |
| **`Duration`** | Selenium 4 waits |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "The everyday use is custom wait conditions. `ExpectedCondition` is a functional interface — a single abstract method — which is exactly why you can pass a lambda to `wait.until()`. Pre-Java 8 that was a six-line anonymous inner class; now it's one line. Being able to explain *why* it works rather than just using the syntax is the difference between copying a pattern and understanding it.
>
> `Supplier` is what I'd reach for in a generic retry helper — it takes a supplier so it can re-invoke the action rather than a value that's already been computed. That's a small thing that makes the difference between a retry wrapper that works and one that just re-returns the same failed result.
>
> `Optional` I'd use for lookups that might not find anything, with `orElseThrow` carrying a meaningful message. That turns a missing table row into 'Row for Sagar not found' rather than a bare `NoSuchElementException` that tells the next person nothing.
>
> And `Duration` — Selenium 4 uses it throughout, so `new WebDriverWait(driver, 10)` dates you to Selenium 3."

---

## Common Interview Questions & Answers

### Q1: "What Java 8 features do you use?"

**A:** "Lambdas, functional interfaces, method references, `Optional` and `Duration`. The most relevant to Selenium is functional interfaces — `ExpectedCondition` is one, which is why you can pass a lambda to `wait.until()` for a custom wait. Day to day it's lambdas for waits, `Supplier` for retry helpers, method references in streams, and `Optional` for lookups that might find nothing."

### Q2: "What's a functional interface?"

**A:** "An interface with exactly one abstract method — which is what makes a lambda possible, since the compiler knows which method you're implementing. `@FunctionalInterface` enforces it at compile time. `ExpectedCondition`, `Predicate`, `Function`, `Supplier` and `Consumer` are all examples."

### Q3: "Why does `Optional` exist and how do you use it properly?"

**A:** "To make absence explicit in the type rather than an implicit null. Used properly means `orElseThrow` with a meaningful message, `orElse` for a fallback, or `ifPresent`. Used improperly means calling `.get()` without checking — that reintroduces the null pointer with extra steps and defeats the point."

### Q4: "Why were default methods added to interfaces?"

**A:** "Backwards compatibility. It let Java add `stream()` to `Collection` without breaking every existing implementation. It narrows the gap with abstract classes, but the key distinction holds — interfaces still can't have instance state, which is why `BasePage` in a Selenium framework is an abstract class: it has to hold the driver and the wait."

### Q5: "What's a method reference?"

**A:** "Shorthand for a lambda that does nothing but call an existing method — `WebElement::getText` instead of `e -> e.getText()`. Four kinds: static, bound to a specific instance, unbound on an arbitrary instance, and constructor. The unbound one is the one people find confusing — it means call this method on whatever comes through."

---

## Weak Areas to Watch

❌ **Don't just list features**
→ Say which you actually use and where.

❌ **Don't call `.get()` on an Optional unchecked**
→ Reintroduces the NPE. Common review catch.

❌ **Don't say default methods make interfaces equal to abstract classes**
→ No instance state. That's why `BasePage` is a class.

❌ **Don't write `new WebDriverWait(driver, 10)`**
→ Selenium 4 wants `Duration`. The int form dates you.

---

## Key Takeaway for Interview

The connection that shows you understand rather than copy:

> **"`ExpectedCondition` is a functional interface — one abstract method — which is exactly why `wait.until(d -> ...)` compiles. Custom waits are the everyday use of lambdas in a Selenium framework."**

And the `Optional` habit worth naming:

> **"`orElseThrow` with a meaningful message. That turns a missing row into 'Row for Sagar not found' instead of a bare exception that tells the next person nothing."**

---

## Checklist

✅ Can define a functional interface and name several
✅ Can explain why `ExpectedCondition` accepts a lambda
✅ `Supplier` used for retry helpers
✅ Method references in streams
✅ `Optional` with `orElseThrow`, never bare `.get()`
✅ Know why default methods exist (backwards compatibility)
✅ `Duration` used for Selenium 4 waits

---

**Remember:** Knowing *why* the lambda compiles beats knowing that it does. 🎯
