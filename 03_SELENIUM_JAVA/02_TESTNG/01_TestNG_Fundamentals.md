# TestNG - Fundamentals: Annotations, Lifecycle, Configuration

## What This Covers

TestNG's execution model — annotations, the order they run in, grouping, dependencies, and the XML suite file.

---

## Why It Matters (Interview Context)

TestNG is the test runner in most Java automation stacks, and interviewers use it to check whether you understand test *lifecycle* or just wrote test methods.

The specific question that separates people:

> "What's the execution order of TestNG annotations?"

And the follow-up that catches more:

> "What happens to `@AfterMethod` if `@BeforeMethod` throws?"

---

## Your 30-Second Interview Answer

> "TestNG runs `@BeforeSuite`, then `@BeforeTest`, `@BeforeClass`, then `@BeforeMethod` before each test method, with the corresponding After annotations unwinding in reverse. The features I actually relied on were groups for tiered execution — smoke versus regression — `dependsOnMethods` for genuine prerequisites, and the XML suite file to control which tests run in which configuration without touching code."

---

## Your 2-Minute Deep Answer

> "**Execution order:**
>
> ```
> @BeforeSuite
>   @BeforeTest
>     @BeforeClass
>       @BeforeMethod  →  @Test  →  @AfterMethod     (repeats per test method)
>     @AfterClass
>   @AfterTest
> @AfterSuite
> ```
>
> The naming is confusing and worth being precise about: `@BeforeTest` does **not** run before each `@Test` method. It runs before each `<test>` tag in the XML suite file — which is a grouping of classes, not a single method. `@BeforeMethod` is the one that runs before each test method.
>
> That distinction trips people up regularly and it's a fair thing to be asked.
>
> **The failure behaviour matters too:** if `@BeforeMethod` throws, the test method is **skipped**, not failed — and `@AfterMethod` still runs. Skipped rather than failed is significant, because a suite reporting lots of skips often means a setup problem rather than a product problem, and people misread that as "tests didn't run" instead of "setup broke."
>
> ---
>
> **Core `@Test` attributes:**
>
> ```java
> @Test(priority = 1)                          // lower runs first; default 0
> @Test(groups = {"smoke", "regression"})
> @Test(dependsOnMethods = {"loginTest"})      // skipped if dependency fails
> @Test(enabled = false)                        // excluded
> @Test(invocationCount = 3)                    // run 3 times
> @Test(timeOut = 5000)                         // fail if over 5s
> @Test(expectedExceptions = NoSuchElementException.class)
> @Test(description = "Verifies login with valid credentials")
> ```
>
> **`priority` is a weak ordering mechanism** and I'd use it sparingly. It creates implicit coupling — someone inserts a test at priority 2 and shifts the meaning of everything after it. If tests genuinely need an order, that's usually a sign they should be creating their own preconditions instead.
>
> **`dependsOnMethods`** is more honest when there is a real dependency, because a failed dependency produces a *skip* rather than a cascade of confusing failures. But the same caveat applies: dependencies mean tests aren't independent, which breaks under parallel execution and makes the suite harder to shard.
>
> ---
>
> **Groups — the feature with the most practical value:**
>
> ```java
> @Test(groups = {"smoke"})
> public void loginWithValidCredentials() { }
>
> @Test(groups = {"regression", "payments"})
> public void refundProcessing() { }
> ```
>
> ```xml
> <groups>
>   <run>
>     <include name="smoke"/>
>     <exclude name="slow"/>
>   </run>
> </groups>
> ```
>
> This is what enables tiered CI — smoke on every commit, full regression nightly. Same strategy as tagging in Playwright, and the same reasoning: running less is a legitimate speed lever and it's free, unlike more machines.
>
> ---
>
> **The XML suite file:**
>
> ```xml
> <suite name="Regression" parallel="classes" thread-count="4">
>   <parameter name="browser" value="chrome"/>
>
>   <listeners>
>     <listener class-name="com.framework.listeners.TestListener"/>
>   </listeners>
>
>   <test name="Auth Tests">
>     <classes>
>       <class name="com.tests.LoginTest"/>
>       <class name="com.tests.LogoutTest"/>
>     </classes>
>   </test>
>
>   <test name="Payment Tests">
>     <packages>
>       <package name="com.tests.payments"/>
>     </packages>
>   </test>
> </suite>
> ```
>
> The value is controlling execution without touching code — which tests run, in what browser, with what parallelism. You can keep several XML files for different purposes: `smoke.xml`, `regression.xml`, `cross-browser.xml`, and point Maven or Jenkins at whichever you need.
>
> ---
>
> **Soft assertions:**
>
> ```java
> SoftAssert softAssert = new SoftAssert();
> softAssert.assertEquals(actualTitle, "Dashboard");
> softAssert.assertTrue(isUserDisplayed);
> softAssert.assertAll();      // ← without this, nothing fails
> ```
>
> Collects failures and reports them together. **`assertAll()` is mandatory** — forget it and the assertions never report, so the test passes regardless. That's a silent false pass and a good code review catch.
>
> Same guidance as elsewhere: soft assertions for independent checks, hard assertions where a later step depends on an earlier one succeeding.
>
> ---
>
> **TestNG versus JUnit, if asked:**
>
> TestNG has built-in data providers, groups, dependencies, parallel execution configuration, and suite XML. JUnit 5 has closed much of that gap with `@ParameterizedTest` and tags, but TestNG's suite-level configuration and dependency handling are still more mature for large automation suites. For a UI automation framework I'd pick TestNG; for unit tests in the application itself, JUnit is the convention."

---

## Annotation Reference

| Annotation | Runs |
|-----------|------|
| `@BeforeSuite` / `@AfterSuite` | Once per suite |
| `@BeforeTest` / `@AfterTest` | Per `<test>` tag in XML — **not** per method |
| `@BeforeClass` / `@AfterClass` | Once per class |
| `@BeforeMethod` / `@AfterMethod` | Before/after **each** test method |
| `@Test` | The test |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Our suite was organized with groups driving tiered execution — smoke on every build, full regression on a schedule. That's the same reasoning I apply now with Playwright tags: running fewer tests more often is a free speed lever, whereas more machines costs money.
>
> `@BeforeMethod` handled driver setup and `@AfterMethod` teardown, so each test got a clean session. The behaviour worth knowing is that if `@BeforeMethod` throws, the test is *skipped* rather than failed, and `@AfterMethod` still runs. That matters for reading reports — a run with many skips usually means setup broke rather than tests failing, and people misdiagnose that.
>
> The XML suite files let us control execution without code changes — separate files for smoke, regression, and cross-browser, with Jenkins pointing at whichever the job needed.
>
> I used `priority` sparingly. It creates implicit coupling: someone inserts a test at priority 2 and silently shifts everything after it. If tests genuinely need ordering, that's usually a signal they should be building their own preconditions instead — and order dependence breaks the moment you go parallel."

---

## Common Interview Questions & Answers

### Q1: "What's the TestNG annotation execution order?"

**A:** "BeforeSuite, BeforeTest, BeforeClass, then BeforeMethod before each test method, with the After annotations unwinding in reverse. The naming trap is `@BeforeTest` — it runs before each `<test>` tag in the XML suite, not before each `@Test` method. `@BeforeMethod` is the per-method one."

### Q2: "If `@BeforeMethod` fails, what happens to the test and `@AfterMethod`?"

**A:** "The test is skipped rather than failed, and `@AfterMethod` still runs. The skip-versus-fail distinction matters for reading reports — a run showing many skips usually means setup broke, not that tests genuinely failed, and that gets misread."

### Q3: "How do you control which tests run?"

**A:** "Groups plus the XML suite file. Tests are tagged into groups — smoke, regression, and by feature area — and separate XML files include or exclude them. Jenkins points at whichever suite the job needs, so changing scope doesn't require a code change."

### Q4: "Would you use `priority` to order tests?"

**A:** "Sparingly. It creates implicit coupling — inserting a test at priority 2 shifts the meaning of everything after. And ordered tests aren't independent, which breaks under parallel execution. If there's a genuine prerequisite I'd use `dependsOnMethods` so a failure produces a clean skip, but mostly it means the tests should be creating their own setup."

### Q5: "What's the catch with `SoftAssert`?"

**A:** "`assertAll()` is mandatory. Without it the collected failures are never reported and the test passes regardless of what failed. That's a silent false pass, and it's a standard code review catch."

---

## Weak Areas to Watch

❌ **Don't say `@BeforeTest` runs before each test method**
→ It's per `<test>` tag. Very common error.

❌ **Don't say a failed `@BeforeMethod` fails the test**
→ It skips it. The distinction matters for reading reports.

❌ **Don't lean on `priority` for ordering**
→ Implicit coupling; breaks parallel execution.

❌ **Don't forget `assertAll()`**
→ Silent false pass.

---

## Key Takeaway for Interview

TestNG questions test whether you understand lifecycle, not whether you can write `@Test`.

The precision that shows real usage:

> **"`@BeforeTest` runs before each `<test>` tag in the suite XML, not before each test method. `@BeforeMethod` is the per-method one — the naming is genuinely misleading."**

And the behaviour most people get wrong:

> **"A failing `@BeforeMethod` skips the test rather than failing it, and `@AfterMethod` still runs. A report full of skips usually means setup broke, not that tests failed."**

---

## Checklist

✅ Can state the full annotation order
✅ Know `@BeforeTest` is per XML `<test>` tag
✅ Know setup failure produces a skip
✅ Groups used for tiered execution
✅ Separate XML suites per purpose
✅ `priority` avoided; `dependsOnMethods` where genuinely needed
✅ `assertAll()` never forgotten

---

**Remember:** Skipped is not failed, and that difference changes how you read a report. 🎯
