# Reporting with ExtentReports

> **Reporting across the repo:** this note covers the Java/TestNG side.
> `02_PLAYWRIGHT/05_CICD_AND_EXECUTION/04_Reporting_And_Artifacts.md` covers Playwright's
> built-in reporters. `05_PROCESS_AND_TOOLING/05_Allure_Reporting.md` covers Allure —
> which works with **both** stacks and adds history and trend tracking.

## What This Covers

Wiring ExtentReports into a TestNG framework — the listener pattern, screenshots on failure, and the thread-safety trap that catches almost everyone.

---

## Why It Matters (Interview Context)

*"How do you generate test reports?"* is a standard question with a shallow standard answer — "we use ExtentReports, it makes HTML reports." True of every reporter.

**The differentiator is the thread-safety problem.** Under parallel execution, a shared `ExtentTest` means log entries land under the wrong test. The suite passes, the report is confidently wrong, and nothing errors. Knowing that failure mode proves you've run a parallel suite rather than read about one.

---

## Your 30-Second Interview Answer

> "ExtentReports driven from a TestNG `ITestListener`, so screenshot-on-failure and report entries live in one place rather than in every test. The critical detail is that `ExtentTest` needs to be in a ThreadLocal — same as the driver. Without it, under parallel execution log entries from different threads land under the wrong test, and nothing errors, so you get a green build with a report that's silently wrong. And `flush()` in `onFinish`, or no file is written at all."

---

## Your 2-Minute Deep Answer

> "**The manager — and the ThreadLocal that matters:**
>
> ```java
> public class ExtentManager {
>     private static ExtentReports extent;
>     private static final ThreadLocal<ExtentTest> test = new ThreadLocal<>();
>
>     public static synchronized ExtentReports getInstance() {
>         if (extent == null) {
>             ExtentSparkReporter reporter =
>                 new ExtentSparkReporter("reports/ExtentReport.html");
>             reporter.config().setDocumentTitle("Automation Report");
>             reporter.config().setTheme(Theme.DARK);
>
>             extent = new ExtentReports();
>             extent.attachReporter(reporter);
>             extent.setSystemInfo("Environment", ConfigReader.get("env"));
>             extent.setSystemInfo("Browser", System.getProperty("browser"));
>         }
>         return extent;
>     }
>
>     public static void createTest(String name) {
>         test.set(getInstance().createTest(name));
>     }
>
>     public static ExtentTest getTest() { return test.get(); }
>
>     public static void unload() { test.remove(); }
> }
> ```
>
> Two things to notice:
>
> **`ExtentReports` is a shared singleton** — one report for the whole run, hence `synchronized` on the getter so concurrent threads don't create two.
>
> **`ExtentTest` is per-thread** — each test needs its own handle. With a single shared field, several tests log simultaneously and whichever thread wrote last owns the entries.
>
> That second one is the trap. The symptoms are nasty precisely because **nothing fails**: green build, all tests pass, and the report shows steps attributed to the wrong tests. You only discover it when someone tries to debug a failure and the logged steps make no sense for that test.
>
> `remove()` matters for the same reason as with the driver — TestNG reuses pooled threads, so a stale entry can outlive the test that created it.
>
> ---
>
> **Wiring through a listener:**
>
> ```java
> public class TestListener implements ITestListener {
>
>     @Override
>     public void onTestStart(ITestResult result) {
>         ExtentManager.createTest(result.getMethod().getMethodName());
>     }
>
>     @Override
>     public void onTestSuccess(ITestResult result) {
>         ExtentManager.getTest().pass("Passed");
>         ExtentManager.unload();
>     }
>
>     @Override
>     public void onTestFailure(ITestResult result) {
>         String path = ScreenshotUtil.capture(
>             DriverFactory.getDriver(),                    // ← current thread's driver
>             result.getMethod().getMethodName());
>
>         ExtentManager.getTest()
>             .fail(result.getThrowable())
>             .addScreenCaptureFromPath(path);
>         ExtentManager.unload();
>     }
>
>     @Override
>     public void onTestSkipped(ITestResult result) {
>         ExtentManager.getTest().skip(result.getThrowable());
>         ExtentManager.unload();
>     }
>
>     @Override
>     public void onFinish(ITestContext context) {
>         ExtentManager.getInstance().flush();              // ← mandatory
>     }
> }
> ```
>
> Register in the suite XML rather than with `@Listeners` on the base class — it keeps the wiring in configuration, so you can vary it per suite without touching code.
>
> **Two things that bite:**
>
> **`flush()` in `onFinish` is mandatory.** Extent buffers everything and writes on flush. Omit it and there's no file and no error — the run looks entirely successful and the report simply isn't there. Confusing to debug because nothing points at it.
>
> **The screenshot must come from the current thread's driver.** `DriverFactory.getDriver()` returns the ThreadLocal instance. Capture from a shared reference under parallel execution and you get another test's browser — which is worse than no screenshot, because it actively sends you the wrong way.
>
> ---
>
> **The screenshot utility:**
>
> ```java
> public static String capture(WebDriver driver, String testName) {
>     File src = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
>     String path = "reports/screenshots/"
>                 + testName + "_" + System.currentTimeMillis() + ".png";
>     FileUtils.copyFile(src, new File(path));
>     return path;
> }
> ```
>
> The **timestamp matters with retries** — without it, attempt two overwrites attempt one and you lose the original evidence, which is usually the more informative failure.
>
> **Base64 instead of file paths** is worth considering:
>
> ```java
> String base64 = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BASE64);
> ExtentManager.getTest().fail(throwable,
>     MediaEntityBuilder.createScreenCaptureFromBase64String(base64).build());
> ```
>
> The report becomes a single self-contained HTML file you can attach to a Jira ticket or email. With file paths, the images break the moment the report leaves the workspace — which is exactly when someone else needs to read it.
>
> ---
>
> **Adding useful context:**
>
> ```java
> ExtentManager.getTest().assignCategory("Regression", "Payments");
> ExtentManager.getTest().assignAuthor("Sagar");
> ExtentManager.getTest().info("Navigating to checkout");
> ExtentManager.getTest().log(Status.INFO,
>     MarkupHelper.createCodeBlock(responseBody, CodeLanguage.JSON));
> ```
>
> Categories give you filterable groupings in the report. Logging the API response body on a failure is genuinely useful — it's often the actual answer.
>
> ---
>
> **Extent vs Allure — the honest comparison:**
>
> **Extent** is lighter, simpler to wire up, and produces a single self-contained HTML file. It tells you about the run that just finished.
>
> **Allure** needs two-stage generation and history persisted in CI, and in exchange gives you trends, automatic flaky detection, and failure categorization.
>
> My view: Extent is right when you want a readable report per run — and for a small suite Allure is over-engineering. Once you're tracking suite health over time, Allure's history is the thing you can't get otherwise. Both need the same thread-safe handling under parallel execution."

---

## The Failure Modes

| Mistake | Symptom | Why it's nasty |
|---------|---------|----------------|
| Shared `ExtentTest` field | Entries under wrong tests | **Nothing errors** — green build, wrong report |
| No `flush()` | No report file | No error either — run looks fine |
| Screenshot from shared driver | Another test's browser | Actively misleading |
| No timestamp on filename | Retry overwrites original | Lose the informative failure |
| File-path screenshots | Broken images outside workspace | Report useless when shared |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "ExtentReports wired through a TestNG listener, so screenshot-on-failure and report entries exist in one place rather than a try/catch in every test.
>
> The thing that caught us out was thread safety in the reporting layer. We had thread-safe drivers via ThreadLocal and the suite was passing — but the report was wrong. Log entries from different threads were landing under the wrong tests, because `ExtentTest` also needs its own ThreadLocal. What made it nasty is that nothing errors: green build, confidently incorrect report, and you only notice when someone tries to debug from it and the steps make no sense.
>
> The general lesson I took from that is thread safety isn't just the driver — it's anything holding per-test state.
>
> Two other details. `flush()` in `onFinish` is mandatory; without it no file is written and nothing complains. And the failure screenshot has to come from the current thread's driver via the factory — capture from a shared reference and you attach another test's browser, which is worse than attaching nothing.
>
> I'd also switch to base64 screenshots rather than file paths, so the report is a single self-contained file. With paths, the images break as soon as you attach the report to a ticket — which is exactly when someone else needs to read it."

---

## Common Interview Questions & Answers

### Q1: "How do you generate test reports?"

**A:** "ExtentReports driven from a TestNG `ITestListener` — creates a test entry on start, logs pass or fail, attaches a screenshot on failure. Keeping it in the listener means it's one implementation rather than scattered through every test. `flush()` in `onFinish` is mandatory or no file gets written."

### Q2: "What's the catch with reporting under parallel execution?"

**A:** "`ExtentTest` needs its own ThreadLocal, same as the driver. With a shared field, several tests log simultaneously and whichever thread wrote last owns the entries — so steps get attributed to the wrong tests. The dangerous part is that nothing fails: green build, wrong report, discovered only when someone tries to debug from it."

### Q3: "Your report isn't being generated. What's wrong?"

**A:** "Almost certainly `flush()` not being called in `onFinish`. Extent buffers and writes on flush — without it there's no file and no error, so the run looks completely fine and the report just isn't there. First thing I'd check."

### Q4: "Why base64 screenshots instead of file paths?"

**A:** "The report becomes a single self-contained HTML file. With file paths the images break as soon as the report leaves the workspace — attach it to a Jira ticket and it's a report full of broken images, which is precisely when someone else needs to read it."

### Q5: "Extent or Allure?"

**A:** "Extent for a readable report per run — lighter, simpler, self-contained, and for a small suite Allure is over-engineering. Allure once you're tracking suite health over time, because its history and trend view is the thing you genuinely can't get elsewhere. Both need the same thread-safe handling."

---

## Weak Areas to Watch

❌ **Don't use a shared `ExtentTest` field**
→ Wrong report, green build, no error.

❌ **Don't forget `flush()`**
→ No report, no error, hard to diagnose.

❌ **Don't capture screenshots from a shared driver**
→ Under parallel you attach the wrong browser.

❌ **Don't omit timestamps on screenshot filenames**
→ Retries overwrite the original evidence.

❌ **Don't answer "it makes nice HTML reports"**
→ True of every reporter. The thread-safety detail is the differentiator.

---

## Key Takeaway for Interview

The thread-safety failure is what proves you've run a parallel suite.

The line that lands:

> **"`ExtentTest` needs its own ThreadLocal, not just the driver. With a shared field, log entries land under the wrong tests — and nothing errors. Green build, confidently wrong report, discovered only when someone tries to debug from it."**

And the practical one:

> **"`flush()` in `onFinish`, or no file is written — and nothing complains, so the run looks completely successful."**

---

## Checklist

✅ `ExtentReports` a synchronized singleton
✅ `ExtentTest` in a ThreadLocal, with `remove()`
✅ Wired through a listener, registered in suite XML
✅ `flush()` in `onFinish`
✅ Screenshots from the ThreadLocal driver
✅ Timestamped filenames so retries don't overwrite
✅ Base64 screenshots for a portable report
✅ API response bodies logged on failure

---

**Remember:** A passing suite with a wrong report is worse than a failing one — nobody knows to check. 🎯
