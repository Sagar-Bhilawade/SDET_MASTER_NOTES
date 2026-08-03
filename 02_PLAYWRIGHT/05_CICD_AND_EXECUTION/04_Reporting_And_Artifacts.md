# CI/CD - Reporting & Artifacts

> **Reporting across the repo:** this note covers Playwright's built-in reporters and artifacts.
> `03_SELENIUM_JAVA/06_FRAMEWORK_AND_BUILD/04_Reporting_ExtentReports.md` covers the Java side.
> `05_PROCESS_AND_TOOLING/05_Allure_Reporting.md` covers Allure — which has adapters for
> **both** stacks and adds history and trend tracking the built-in reporter doesn't have.

## What This Covers

How test results get communicated — to you, to developers, and to management. And how to make a CI failure debuggable rather than a mystery.

---

## Why It Matters (Interview Context)

Reporting is where automation either delivers value or quietly stops mattering.

A suite that produces a raw log dump nobody reads has the same practical effect as no suite. Interviewers are checking whether you think about **who consumes the output** — because that's an operational concern, and operational thinking is what distinguishes senior candidates.

---

## Your 30-Second Interview Answer

> "I run multiple reporters at once because different audiences need different things — HTML for debugging, JUnit XML for the CI dashboard and trend tracking, and a console reporter for immediate feedback. The critical piece is artifacts: traces on first retry, video and screenshots on failure, uploaded so they survive the run. A CI failure without a trace is guesswork; with one it's a five-minute fix."

---

## Your 2-Minute Deep Answer

> "Reporters and artifacts serve different purposes and you want both.
>
> **Reporters — multiple, because audiences differ:**
>
> ```typescript
> reporter: [
>   ['list'],                                              // console, live
>   ['html', { outputFolder: 'playwright-report', open: 'never' }],
>   ['junit', { outputFile: 'test-results/junit.xml' }],
>   ['json', { outputFile: 'test-results/results.json' }],
> ],
> ```
>
> **`list`** — live console output. What you watch while it runs.
>
> **`html`** — the debugging report. Filterable, clickable, and traces embedded per test. This is what you open when something failed.
>
> **`junit`** — the machine-readable one. This is what Azure DevOps and Jenkins consume to populate their test dashboards. Without it, results exist only in log output and nobody tracks trends.
>
> **`json`** — for custom tooling. If I'm calculating first-attempt pass rate or feeding a metrics dashboard, this is the source.
>
> **`blob`** — used when sharding, so results from multiple machines can be merged into one report.
>
> `open: 'never'` on HTML matters in CI — otherwise the reporter tries to serve and open a browser, and the pipeline hangs.
>
> ---
>
> **Artifacts — what makes failures debuggable:**
>
> ```typescript
> use: {
>   trace: 'on-first-retry',
>   video: 'retain-on-failure',
>   screenshot: 'only-on-failure',
> },
> ```
>
> **Why these specific settings:**
>
> `trace: 'on-first-retry'` is the important one. Traces are large and slow to produce, so capturing them on every test would bloat storage and slow the pipeline. Capturing on the first retry means you pay the cost only when something failed — you get the trace of a genuine failure at near-zero cost on the happy path.
>
> This is the honest use of retries. Retries shouldn't be there to make flaky tests pass; they're there so you capture a trace when something fails. The trace is the payoff.
>
> `retain-on-failure` for video keeps recordings only for failures and deletes the rest.
>
> **What a trace contains:** every action, the DOM at each step, all network requests and responses, console messages, and timing. Replayed with:
>
> ```bash
> npx playwright show-trace trace.zip
> ```
>
> The single most valuable debugging technique available is diffing a passing trace against a failing one — where they diverge is the bug.
>
> ---
>
> **Uploading artifacts — the mistake everyone makes:**
>
> ```yaml
> # GitHub Actions
> - uses: actions/upload-artifact@v4
>   if: always()                            # ← without this, skipped on failure
>   with:
>     name: playwright-artifacts
>     path: |
>       playwright-report/
>       test-results/
>     retention-days: 7
> ```
>
> ```yaml
> # Azure DevOps
> - task: PublishBuildArtifacts@1
>   condition: succeededOrFailed()          # ← same idea
> ```
>
> Omitting the always-condition means the upload step is skipped when tests fail — precisely when you need it. It's the most common mistake in Playwright CI setups and it's completely self-defeating.
>
> `retention-days` matters at scale. Traces and videos are large; keeping them indefinitely across hundreds of runs gets expensive fast.
>
> ---
>
> **Making results reach people:**
>
> A report nobody opens has no value. Practically:
>
> - **PR comments** with pass/fail summary and a link to the report — meets developers where they already are
> - **Slack or Teams notification** on main-branch failure, not on every run, or people mute the channel
> - **The dashboard** for trends: first-attempt pass rate, execution time, flaky test list
>
> The notification discipline matters. Alert on everything and people filter it out, and then the one alert that mattered gets missed too. I'd notify on main failures and nothing else.
>
> ---
>
> **What I'd report to different audiences:**
>
> To **developers:** did your change break something, and here's the trace.
>
> To **the team:** suite health — first-attempt pass rate, execution time trend, flaky list.
>
> To **management:** escapes prevented, feedback speed, maintenance cost. Not test counts or coverage percentage, because neither maps to anything they can act on."

---

## Reporter Reference

| Reporter | Output | Audience |
|----------|--------|----------|
| `list` | Console, live | You, during a run |
| `html` | Interactive report | Debugging a failure |
| `junit` | XML | CI dashboards, trends |
| `json` | JSON | Custom metrics tooling |
| `blob` | Mergeable blob | Sharded runs |
| `github` | PR annotations | GitHub Actions inline |

---

## Artifact Settings

| Setting | Value | Why |
|---------|-------|-----|
| `trace` | `on-first-retry` | Full detail, minimal cost |
| `video` | `retain-on-failure` | Visual context, deleted on pass |
| `screenshot` | `only-on-failure` | Cheap, quick glance |
| `retention-days` | 7–14 | Traces are large |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "We run multiple reporters because different consumers need different things — HTML for debugging, JUnit XML feeding Azure's Tests tab for trends, and console output during the run.
>
> The setting I'd call out is `trace: 'on-first-retry'`. Traces are expensive to produce and large to store, so capturing them on every test would slow the pipeline and bloat storage. On first retry means you only pay when something actually failed. That's also the honest role of retries in my view — they're not there to make flaky tests pass, they're there so you capture a trace of a real failure.
>
> The trace is what changed our debugging economics. A CI failure used to mean reading a stack trace and theorizing; now it's downloading the trace and replaying the exact run — every action, every network call, the DOM at each step. And diffing a passing trace against a failing one is the fastest way I know to find an intermittent failure.
>
> The mistake I'd flag is forgetting `if: always()` on the artifact upload. Without it the step is skipped when tests fail, which is exactly when the artifacts matter. It's the most common Playwright CI setup error and it's completely self-defeating."

---

## Common Interview Questions & Answers

### Q1: "What reporters do you use?"

**A:** "Several simultaneously, because audiences differ. HTML for debugging failures, JUnit XML so results populate the CI dashboard with trends, list for live console output, and JSON when I need to compute metrics like first-attempt pass rate. A single reporter always shortchanges one audience."

### Q2: "Why `trace: 'on-first-retry'` rather than always?"

**A:** "Cost. Traces are large and slow to generate, so capturing on every test bloats storage and slows the pipeline. On first retry means you only pay when something failed — full detail on genuine failures, near-zero overhead on the happy path. It's also the honest use of retries: they're for capturing traces, not for making unstable tests pass."

### Q3: "How do you make CI failures debuggable?"

**A:** "Traces, uploaded as artifacts with an always-condition so they survive a failed run. Then a failure is a replay rather than a guessing exercise — you see every action, the network calls, the DOM state, the exact assertion that failed. Without artifacts you're reading a stack trace and theorizing, which is where hours disappear."

### Q4: "How do you report results to management?"

**A:** "In terms they can act on — escapes prevented, feedback speed, manual regression displaced, and the maintenance cost stated honestly. Not test counts or coverage percentage; neither means anything outside QA and both invite the wrong optimization. Including the cost is what makes the rest credible."

### Q5: "How do you avoid alert fatigue?"

**A:** "Notify on main-branch failures only, not every run. If people get pinged for everything, they mute the channel, and then the alert that mattered gets missed too. PR comments are better for per-change feedback because they're already where the developer is working, rather than another channel to monitor."

---

## Weak Areas to Watch

❌ **Don't forget `if: always()` / `succeededOrFailed()`**
→ Artifacts skipped exactly when needed.

❌ **Don't use only the HTML reporter**
→ No JUnit means no dashboard, no trends.

❌ **Don't set `trace: 'on'`**
→ Slow and storage-heavy for little benefit.

❌ **Don't forget `open: 'never'` in CI**
→ Reporter tries to open a browser; pipeline hangs.

❌ **Don't notify on every run**
→ Alert fatigue means the important one gets missed.

❌ **Don't report test counts to management**
→ Means nothing outside QA.

---

## Key Takeaway for Interview

Reporting is what turns test execution into something people act on.

The setting that shows real understanding:

> **"`trace: 'on-first-retry'` — you get full detail on genuine failures at near-zero cost on the happy path. That's also the honest use of retries: capturing a trace, not making flaky tests pass."**

And the operational point:

> **"A report nobody opens has the same value as no report. Notify on main failures only — alert on everything and people mute the channel, and then the one that mattered gets missed too."**

---

## Checklist

✅ Multiple reporters configured for different audiences
✅ `open: 'never'` on HTML in CI
✅ JUnit XML published to the CI dashboard
✅ `trace: 'on-first-retry'`
✅ `video: 'retain-on-failure'`, `screenshot: 'only-on-failure'`
✅ Artifacts uploaded with always-condition
✅ `retention-days` set to control storage cost
✅ Notifications on main failures only
✅ Management reporting in escapes and feedback speed, not test counts

---

**Remember:** A failure you can't debug is a failure you'll retry until it passes. 🎯
