# Selenium + Java Cheat Sheet — Technical Round (Track 2)

> Dense recall for `03_SELENIUM_JAVA`. Your depth track — proves range and enterprise
> experience. Learn from the full notes; fire from this.

---

## WebDriver architecture (the 30-second answer)

Your test → **WebDriver client** → (W3C protocol over HTTP) → **browser driver** (chromedriver) → browser.

**Why stale elements happen:** a `WebElement` is a *reference* to a DOM node at a point in time. If the DOM re-renders (AJAX, navigation), the reference points at a node that no longer exists → `StaleElementReferenceException`. Fix: re-find the element, or use PageFactory's lazy proxies carefully.

**Why it's slower than Playwright:** each command is an HTTP round-trip to the driver; no persistent connection, no native event-waiting → you manage waits yourself.

---

## Locators & XPath

Priority: `id` → `name` → `cssSelector` → `xpath` (slowest, most brittle but most powerful).

**The table-row XPath (know this cold):**
```
//table[@id='orders']//tr[td[text()='John']]/td[3]
```
"The 3rd cell of the row whose first cell is John." Axes: `following-sibling`, `parent`, `ancestor`, `preceding`.

---

## Waits (the #1 Selenium gotcha)

- **Implicit:** global, applied to every findElement. Set once.
- **Explicit (`WebDriverWait` + `ExpectedConditions`):** wait for a specific condition on a specific element.
- **Fluent:** explicit + custom polling interval + ignored exceptions.

**THE mixing problem:** setting implicit AND explicit waits together makes timeouts **compound unpredictably** — a 10s implicit + 10s explicit can wait far longer or behave erratically. **Rule: pick explicit-only.** Replacing hard `Thread.sleep` with Fluent Waits is what drove your flakiness under 2%.

---

## Common exceptions → real fixes

| Exception | Cause | Fix |
|-----------|-------|-----|
| `StaleElementReference` | DOM re-rendered | Re-find element |
| `ElementClickIntercepted` | Another element on top (overlay/modal) | Wait for overlay gone, or scroll into view |
| `ElementNotInteractable` | Present but hidden/disabled | Explicit wait for clickability |
| `NoSuchElement` | Not in DOM yet | Explicit wait for presence |
| `TimeoutException` | Condition never met | Check locator / real app delay |

---

## TestNG

- **Annotation order:** `@BeforeSuite → @BeforeTest → @BeforeClass → @BeforeMethod → @Test → @After...` (reverse).
- **`@BeforeTest` trap:** runs once per `<test>` tag, not per method — people confuse it with `@BeforeMethod`.
- **DataProvider vs Parameters:** `@DataProvider` = complex/multiple data sets from code; `@Parameters` = simple values from `testng.xml`.
- **Groups:** tag tests (`smoke`, `regression`), run subsets.

---

## Parallel execution & ThreadLocal (the senior question)

Shared `WebDriver` across threads = race condition → tests stomp each other. Fix: **`ThreadLocal<WebDriver>`** — each thread gets its own instance.

**Why `remove()` matters:** thread pools **reuse threads**. Without `driver.remove()` in teardown, the next test on that thread inherits a stale/closed driver → leaks and flaky failures. `remove()` in `@AfterMethod` is mandatory, not optional.

---

## Java for SDET

- **OOP via framework:** Encapsulation (page objects hide locators), Inheritance (BaseTest/BasePage), Polymorphism (driver factory returns `WebDriver`), Abstraction (interfaces). Explain via *your* framework, not textbook definitions.
- **Collections:** `HashMap` vs `ConcurrentHashMap` (parallel-safe). `List` vs `Set` (dupes/order).
- **Java 8:** lambdas, streams (`map`/`filter`/`collect`/`groupingBy`). **Why `ExpectedCondition` takes one lambda** → it's a functional interface (single abstract method), so a lambda satisfies it. `Optional` to avoid NPEs.
- **Exceptions:** catching too broadly **hides failures** — a swallowed exception makes a broken test pass green. Type erasure: generics are compile-time only.

---

## RestAssured / API

- Structure: `given().header().body() .when().post() .then().statusCode().body()`.
- **POJO serialization** via Jackson; `@JsonIgnoreProperties(ignoreUnknown=true)` so a new response field doesn't break deserialization.
- **401 vs 403:** not authenticated vs authenticated-but-forbidden.
- Token: fetch once, cache, reuse. `matchesJsonSchemaInClasspath()` for schema validation.

---

## SQL (see `10/02` for the full bank)

Orphan detection (LEFT JOIN + IS NULL), `NOT IN` + NULL trap (returns nothing), `DENSE_RANK` for Nth-highest, `WHERE` vs `HAVING`, `COUNT(col)` skips NULLs.

---

## Framework & build

- **"Explain your framework":** decisions with costs — POM, ThreadLocal driver, explicit waits, data-driven via DataProvider/POI, ExtentReports, Maven surefire for parallelism, Jenkins.
- **Maven:** scopes (compile/test/provided), surefire runs tests, profiles for environments, `dependency:tree` for conflicts.
- **ExtentReports ThreadLocal trap:** `ExtentTest` must be ThreadLocal in parallel, or logs cross between tests. Always `flush()`.
- **Jenkins:** `post { always { } }` for reports/cleanup, headless viewport size matters.

---

## Key takeaway

> **WebDriver architecture (why staleness happens), the implicit/explicit mixing problem,
> and ThreadLocal `remove()` — these three are the Selenium seniority markers.** This track
> proves you handle enterprise legacy and gives your "I've built in both paradigms" bridge
> its weight. Explain everything through your framework, never as definitions.
