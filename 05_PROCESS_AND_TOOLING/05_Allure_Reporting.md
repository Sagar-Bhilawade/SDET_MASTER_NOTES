# Allure Reporting

> **Reporting across the repo:** Allure sits here because it works with **both** stacks —
> it has adapters for Playwright and for Java/TestNG.
> `02_PLAYWRIGHT/05_CICD_AND_EXECUTION/04_Reporting_And_Artifacts.md` covers Playwright's
> built-in reporters. `03_SELENIUM_JAVA/06_FRAMEWORK_AND_BUILD/04_Reporting_ExtentReports.md`
> covers ExtentReports on the Java side.

## What This Covers

Allure — history, trends, categorized failures, and how it differs from Extent Reports and the built-in Playwright reporter.

**Both Allure and Extent Reports are on your resume**, so knowing what distinguishes them is fair game.

---

## Why It Matters (Interview Context)

Reporting questions usually get a shallow answer — "we use Allure, it makes nice HTML reports." That's true of every reporter.

What Allure actually gives you that others don't is **history across runs** — trend graphs, flakiness detection, and the ability to see that a test has failed intermittently for three weeks. That's the capability worth knowing, because it connects directly to the metrics discipline: first-attempt pass rate and suite decay are trend questions, and most reporters only show you one run.

---

## Your 30-Second Interview Answer

> "Allure's differentiator is history. Most reporters show you the current run; Allure keeps previous results and gives you trend graphs and automatic flaky-test detection — a test that passes and fails across runs gets flagged as flaky without you tracking it manually. It also supports categorizing failures, so infrastructure problems get separated from real product defects. Extent Reports is lighter and simpler to wire up; Allure is better when you care about suite health over time rather than just the last run."

---

## Your 2-Minute Deep Answer

> "**Setup — two stages, which is the thing to understand:**
>
> Allure works differently from most reporters. Tests write raw **results** during the run; a separate step **generates** the report from them.
>
> ```bash
> # 1. Tests run, producing allure-results/
> mvn clean test
> npx playwright test
>
> # 2. Generate the HTML report from those results
> allure generate allure-results --clean -o allure-report
>
> # 3. Or serve it directly
> allure serve allure-results
> ```
>
> That two-stage model is what enables history — you can merge results from multiple shards, and you can carry previous history forward into a new report.
>
> ---
>
> **The history feature — the actual differentiator:**
>
> ```bash
> # Carry previous history into the new report
> cp -r allure-report/history allure-results/history
> allure generate allure-results --clean -o allure-report
> ```
>
> With history present, you get:
>
> - **Trend graphs** — pass/fail counts across the last N runs
> - **Duration trend** — is the suite getting slower over time?
> - **Retries and flaky flagging** — a test that both passed and failed gets marked flaky automatically
> - **Per-test history** — this specific test's outcome across previous runs
>
> That last one is genuinely useful. When a test fails, you immediately see whether it's been failing intermittently for weeks or whether this is new — which changes the investigation entirely.
>
> **This is the connection to the metrics note.** Suite decay is a trend, and a single-run report can't show it. Allure gives you the trend for free.
>
> In CI, the history directory needs to be persisted between runs — usually as a cached artifact — otherwise every report starts fresh and you get none of this.
>
> ---
>
> **Categories — separating infrastructure from product:**
>
> ```json
> [
>   {
>     "name": "Product defects",
>     "matchedStatuses": ["failed"]
>   },
>   {
>     "name": "Test infrastructure",
>     "matchedStatuses": ["broken"],
>     "traceRegex": ".*(TimeoutException|WebDriverException|ConnectionRefused).*"
>   }
> ]
> ```
>
> Allure distinguishes **failed** (an assertion failed — likely a real defect) from **broken** (an unexpected exception — likely an environment or infrastructure problem). Categories let you route them into separate buckets.
>
> That distinction matters practically: a report showing "40 failures" reads very differently once you know 35 were the environment being down and 5 were real. Without it, everyone dismisses the whole run.
>
> ---
>
> **Annotations for readable reports:**
>
> ```java
> @Epic("Checkout")
> @Feature("Discount handling")
> @Story("Percentage discounts apply at threshold")
> @Severity(SeverityLevel.CRITICAL)
> @Link(name = "PROJ-1234", url = "https://jira/PROJ-1234")
> @Test
> public void discountAppliesAtThreshold() {
>     step("Add item above threshold");
>     step("Verify discount applied");
> }
> ```
>
> `@Step` is the one that matters for readability — same principle as `test.step` in Playwright. A failure reports which step broke rather than a line number, which is what makes the report useful to someone who didn't write the test.
>
> Attachments — screenshots, request/response bodies, logs — go inline in the report.
>
> ---
>
> **Allure vs Extent vs built-in:**
>
> **Extent Reports** — lighter, simpler to set up, single self-contained HTML. Good enough when you want a readable report per run. What it doesn't give you is history and trends. And it needs `ExtentTest` in a ThreadLocal under parallel execution, or entries land under the wrong test.
>
> **Playwright's built-in HTML reporter** — excellent for debugging a single run, with traces embedded. Not designed for cross-run trends.
>
> **Allure** — heavier setup, two-stage generation, needs history persisted in CI. In exchange you get trends, flaky detection and categorization.
>
> My view: for a small suite where you just need to read what failed, Extent or the built-in reporter is fine and Allure is over-engineering. Once you're tracking suite health over time — which you should be at any real scale — Allure's history is the thing you can't get elsewhere without building it yourself."

---

## Reporter Comparison

| | Allure | Extent | Playwright HTML |
|---|---|---|---|
| Setup | Two-stage | Simple | Built in |
| Single-run readability | ✅ | ✅ | ✅ Excellent |
| **History / trends** | ✅ | ❌ | ❌ |
| **Auto flaky detection** | ✅ | ❌ | ⚠️ Partial |
| Failure categorization | ✅ | ❌ | ❌ |
| Embedded traces | Attachments | Screenshots | ✅ Native |
| Needs CI history persistence | ✅ Yes | No | No |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "The reason Allure earns its place over a simpler reporter is history. Most reporters tell you about the run that just finished; Allure keeps previous results and gives you trend graphs plus automatic flaky detection — a test that both passed and failed across runs gets flagged without anyone tracking it manually.
>
> That matters because the metrics I actually care about are trends. First-attempt pass rate drifting, execution time creeping up quarter over quarter — none of that is visible in a single-run report. Allure gives it to you as a by-product.
>
> The per-test history is the practical part. When something fails, you can see immediately whether it's been failing intermittently for three weeks or whether this is the first time. That completely changes how you investigate — new failure means look at the recent change; long-running intermittent means it's a flakiness root cause you haven't found.
>
> The other feature I'd mention is categories, splitting failed from broken. An assertion failure is likely a real defect; an unexpected exception is usually infrastructure. A report saying '40 failures' gets dismissed wholesale — one saying '35 environment, 5 product' gets acted on.
>
> The catch in CI is that the history directory has to be persisted between runs, usually as a cached artifact. Miss that and every report starts fresh and you lose the entire benefit."

---

## Common Interview Questions & Answers

### Q1: "Why Allure over other reporters?"

**A:** "History. Other reporters show the run that just finished; Allure keeps previous results and gives trend graphs, duration trends, and automatic flaky-test flagging. That matters because suite health is a trend — first-attempt pass rate drifting or execution time creeping up isn't visible in a single-run report."

### Q2: "What's the difference between failed and broken in Allure?"

**A:** "Failed means an assertion failed — likely a real defect. Broken means an unexpected exception — usually infrastructure or environment. Categories let you route them separately, which matters because a report saying '40 failures' gets dismissed wholesale, while '35 environment, 5 product' gets acted on."

### Q3: "How does Allure detect flaky tests?"

**A:** "Through history plus retries. If a test both passed and failed — either across runs or on retry within a run — it flags it automatically. That's useful precisely because flakiness is invisible in any single run, and manual tracking is the thing teams stop doing."

### Q4: "What's the setup catch with Allure in CI?"

**A:** "You have to persist the history directory between runs, typically as a cached artifact, and copy it into the results before generating. Miss that and every report generates fresh with no history — which removes the entire reason for choosing Allure. It's an easy thing to overlook because the report still looks fine."

### Q5: "Allure or Extent?"

**A:** "Extent for a small suite where you just need a readable report per run — it's lighter and simpler, and Allure is over-engineering at that scale. Allure once you're tracking suite health over time, because the history and trend view is the thing you can't get elsewhere without building it. Both need thread-safe handling under parallel execution."

---

## Weak Areas to Watch

❌ **Don't say "it makes nice HTML reports"**
→ True of every reporter. History is the differentiator.

❌ **Don't forget history persistence in CI**
→ Removes the whole benefit, silently.

❌ **Don't ignore failed vs broken**
→ Separating infrastructure from product is what stops reports being dismissed.

❌ **Don't over-engineer a small suite**
→ Extent or built-in is the right call at small scale.

---

## Key Takeaway for Interview

The differentiator is history, and it connects to the metrics discipline.

The answer that shows you understand why it exists:

> **"Most reporters tell you about the run that just finished. Allure keeps history — so you see whether a failing test has been intermittent for three weeks or whether this is new. That changes the investigation completely."**

And the CI gotcha:

> **"The history directory has to be persisted between runs. Miss it and every report generates fresh — the report still looks fine, and you've lost the entire reason for choosing Allure."**

---

## Checklist

✅ Understand the two-stage results → generate model
✅ History persisted between CI runs
✅ Categories configured to split product from infrastructure
✅ `@Step` used for readable failure reporting
✅ Screenshots and request/response attached on failure
✅ Thread-safe under parallel execution
✅ Can justify Allure vs Extent by suite scale

---

**Remember:** A report that only shows today can't tell you the suite is decaying. 🎯
