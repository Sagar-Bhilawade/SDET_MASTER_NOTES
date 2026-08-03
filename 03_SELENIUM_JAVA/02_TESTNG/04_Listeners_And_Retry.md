# TestNG - Listeners & Retry Analyzers

## What This Covers

`ITestListener` for hooking into test lifecycle events — screenshots on failure, logging, reporting — and `IRetryAnalyzer` for retrying failed tests.

The retry part carries a judgement question that's worth getting right.

---

## Why It Matters (Interview Context)

Listeners are how a Java framework gets its cross-cutting behaviour: screenshot-on-failure, report entries, logging. "How do you capture a screenshot when a test fails?" is a standard question, and the answer is a listener, not a try/catch in every test.

The retry analyzer question is more interesting, because the naive answer — "retry failures so the suite goes green" — is the wrong instinct, and interviewers know it.

---

## Your 30-Second Interview Answer

> "`ITestListener` hooks the test lifecycle — I use `onTestFailure` to capture a screenshot and attach it to the report, which means that logic lives in one place rather than a try/catch in every test. `IRetryAnalyzer` lets you re-run failures, and my position there is that retries are for capturing diagnostics, not for making unstable tests pass. A test that only goes green on retry is still broken, so I'd track the retry rate rather than let it silently absorb instability."

---

## Your 2-Minute Deep Answer

> "**`ITestListener` — lifecycle hooks:**
>
> ```java
> public class TestListener implements ITestListener {
>
>     @Override
>     public void onTestStart(ITestResult result) {
>         ExtentReportManager.createTest(result.getMethod().getMethodName());
>     }
>
>     @Override
>     public void onTestSuccess(ITestResult result) {
>         ExtentReportManager.getTest().pass("Passed");
>     }
>
>     @Override
>     public void onTestFailure(ITestResult result) {
>         WebDriver driver = DriverFactory.getDriver();
>         String path = ScreenshotUtil.capture(driver, result.getMethod().getMethodName());
>         ExtentReportManager.getTest()
>             .fail(result.getThrowable())
>             .addScreenCaptureFromPath(path);
>     }
>
>     @Override
>     public void onTestSkipped(ITestResult result) {
>         ExtentReportManager.getTest().skip(result.getThrowable());
>     }
>
>     @Override
>     public void onFinish(ITestContext context) {
>         ExtentReportManager.flush();      // ← without this, no report is written
>     }
> }
> ```
>
> Registered either in the suite XML or with `@Listeners(TestListener.class)` on the base test class. XML is better — it keeps the wiring in configuration rather than scattered across classes.
>
> **Two things worth calling out:**
>
> `onFinish` calling `flush()` is mandatory with ExtentReports. Forget it and the report file is never written — you get a green run and no report, which is a confusing failure to debug because nothing errored.
>
> **The screenshot must come from the right driver.** With parallel execution, `DriverFactory.getDriver()` returns the current thread's driver. If you'd stored a driver reference somewhere shared, you'd capture the wrong browser — a screenshot of another test's failure, which is worse than no screenshot because it sends you the wrong way.
>
> ---
>
> **Screenshot utility:**
>
> ```java
> public static String capture(WebDriver driver, String testName) {
>     File src = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
>     String path = "reports/screenshots/" + testName + "_" + System.currentTimeMillis() + ".png";
>     FileUtils.copyFile(src, new File(path));
>     return path;
> }
> ```
>
> The timestamp matters with retries — otherwise attempt two overwrites attempt one and you lose the original evidence.
>
> ---
>
> **Other listener interfaces:**
>
> - `IInvokedMethodListener` — fires around every method including configuration methods, so it catches `@BeforeMethod` failures that `ITestListener` doesn't
> - `ISuiteListener` — suite start and finish
> - `IReporter` — custom report generation after the run
> - `IAnnotationTransformer` — modify annotations at runtime; this is how you apply a retry analyzer globally
>
> ---
>
> **`IRetryAnalyzer` — and the judgement around it:**
>
> ```java
> public class RetryAnalyzer implements IRetryAnalyzer {
>     private int count = 0;
>     private static final int MAX = 2;
>
>     @Override
>     public boolean retry(ITestResult result) {
>         if (count < MAX) {
>             count++;
>             return true;      // re-run
>         }
>         return false;
>     }
> }
> ```
>
> Applied per test with `@Test(retryAnalyzer = RetryAnalyzer.class)`, or globally via `IAnnotationTransformer`:
>
> ```java
> public class RetryTransformer implements IAnnotationTransformer {
>     @Override
>     public void transform(ITestAnnotation annotation, Class c, Constructor m, Method method) {
>         annotation.setRetryAnalyzer(RetryAnalyzer.class);
>     }
> }
> ```
>
> **My position, which I'd state clearly:**
>
> Retries are a **diagnostic** tool, not a **stability** tool. The legitimate use is capturing evidence — screenshot and logs on the first failure — at low cost.
>
> What retries must not become is a way to make an unstable suite look green. A test that only passes on retry is still broken; the root cause is missing synchronization, shared state, or non-deterministic data, not insufficient attempts. Retries let that rot spread quietly while the dashboard stays clean.
>
> So I'd track **first-attempt pass rate** separately from final pass rate. If the gap widens, the suite is decaying and retries are hiding it. That's the same metric discipline I'd apply in any framework — the tool changes, the principle doesn't.
>
> **A practical bug worth knowing:** the `count` field in that analyzer is an instance field, and TestNG's instantiation behaviour means retry counts can behave unexpectedly across tests. Safer to key the count by test method — a `ConcurrentHashMap<String, Integer>` — especially under parallel execution where a shared counter is a race condition."

---

## Listener Interfaces

| Interface | Fires on | Use for |
|-----------|----------|---------|
| `ITestListener` | Test start/pass/fail/skip | Screenshots, reporting |
| `IInvokedMethodListener` | Every method incl. config | Catching `@BeforeMethod` failures |
| `ISuiteListener` | Suite start/finish | Global setup, report init |
| `IReporter` | After the run | Custom reports |
| `IAnnotationTransformer` | At load | Applying retry globally |
| `IRetryAnalyzer` | On failure | Re-running |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Our `ITestListener` handled screenshot-on-failure and ExtentReports entries, so that logic lived in one place instead of a try/catch in every test. Registered through the suite XML rather than annotations, which kept the wiring in configuration.
>
> Two details mattered under parallel execution. The screenshot has to come from the current thread's driver via the factory — if you capture from a shared reference you get another test's browser, which is worse than no screenshot because it actively misleads. And `flush()` in `onFinish` is mandatory; without it no report file is written at all, and nothing errors, so it's a confusing thing to debug.
>
> On retries, my position is that they're for capturing diagnostics rather than creating stability. We had a retry analyzer, but I'd track first-attempt pass rate separately from final, because a test that only goes green on retry is still broken. Letting retries absorb instability means the dashboard stays clean while the suite decays underneath.
>
> One bug worth knowing: the retry counter as an instance field misbehaves across tests given how TestNG instantiates the analyzer, and it's a race condition under parallel execution. Keying the count by method name in a concurrent map is the safer version."

---

## Common Interview Questions & Answers

### Q1: "How do you capture a screenshot when a test fails?"

**A:** "`ITestListener.onTestFailure` — cast the driver to `TakesScreenshot`, save the file, and attach it to the report. Putting it in a listener means it's one implementation rather than a try/catch in every test. Under parallel execution the driver must come from the ThreadLocal factory, or you capture another test's browser."

### Q2: "How do you register listeners?"

**A:** "Either `@Listeners` on the base test class or a `<listeners>` block in the suite XML. I prefer XML — it keeps the wiring in configuration, so you can vary it per suite without touching code."

### Q3: "What's `IRetryAnalyzer` and would you use it?"

**A:** "It re-runs failed tests. I'd use it for diagnostics — capturing a screenshot and logs on failure cheaply — not for stability. A test that only passes on retry is still broken; the cause is synchronization or shared state, not insufficient attempts. So I'd track first-attempt pass rate separately, because otherwise retries hide a decaying suite behind a green dashboard."

### Q4: "Your ExtentReport isn't generating. What's wrong?"

**A:** "Almost certainly `flush()` not being called in `onFinish`. Extent buffers everything and writes on flush — without it there's no file and nothing errors, so the run looks fine and the report just isn't there. It's the first thing I'd check."

### Q5: "What's a subtle bug in a typical retry analyzer?"

**A:** "The count as an instance field. TestNG's instantiation means the counter can behave unexpectedly across test methods, and under parallel execution a shared counter is a race condition. Keying by test method name in a `ConcurrentHashMap` is the safer implementation."

---

## Weak Areas to Watch

❌ **Don't put screenshot logic in every test**
→ That's what listeners are for.

❌ **Don't capture from a shared driver reference**
→ Under parallel, you get the wrong browser.

❌ **Don't forget `flush()`**
→ No report, no error, confusing to debug.

❌ **Don't present retries as a stability solution**
→ Senior interviewers push back hard on this.

❌ **Don't use a plain instance counter in the retry analyzer**
→ Misbehaves across tests and races under parallel.

---

## Key Takeaway for Interview

Listeners are how cross-cutting concerns stay out of test code. Retries are where judgement gets tested.

The position to hold:

> **"Retries are a diagnostic tool, not a stability tool. A test that only passes on retry is still broken — so I track first-attempt pass rate separately, because otherwise retries hide a decaying suite behind a green dashboard."**

And the parallel-execution detail:

> **"The failure screenshot must come from the current thread's driver. Capture from a shared reference and you get another test's browser, which is worse than no screenshot because it sends you the wrong way."**

---

## Checklist

✅ `ITestListener` handling screenshots and report entries
✅ Registered via suite XML
✅ Screenshot driver pulled from ThreadLocal factory
✅ Timestamped filenames so retries don't overwrite evidence
✅ `flush()` in `onFinish`
✅ Retry count keyed by method, not an instance field
✅ First-attempt pass rate tracked separately

---

**Remember:** A retry that hides a real failure costs more than the failure would have. 🎯
