# Jenkins - CI for Selenium Frameworks

## What This Covers

Running the Java suite in Jenkins — jobs, pipelines, parameters, artifacts, and scheduling.

---

## Why It Matters (Interview Context)

Same principle as the Playwright CI notes: a suite that only runs locally isn't automation. Jenkins is the CI tool on most legacy Java stacks, so this is where you demonstrate the Track 2 work was operational.

Interviewers check whether you configured it or just watched someone else's job run.

---

## Your 30-Second Interview Answer

> "Declarative pipeline in a Jenkinsfile, kept in the repo so it's version-controlled with the tests. Parameterized for environment, browser, and suite, so one job serves multiple purposes. The parts that matter beyond running the command are publishing the TestNG results so the dashboard shows trends, archiving reports and screenshots with `always` so they survive a failed build, and headless execution because build agents have no display."

---

## Your 2-Minute Deep Answer

> "**A declarative pipeline:**
>
> ```groovy
> pipeline {
>     agent any
>
>     parameters {
>         choice(name: 'ENV', choices: ['staging', 'qa'], description: 'Target environment')
>         choice(name: 'BROWSER', choices: ['chrome', 'firefox'], description: 'Browser')
>         choice(name: 'SUITE', choices: ['smoke', 'regression'], description: 'Suite')
>     }
>
>     triggers {
>         cron('H 2 * * *')          // nightly regression
>     }
>
>     options {
>         timeout(time: 2, unit: 'HOURS')
>         buildDiscarder(logRotator(numToKeepStr: '30'))
>     }
>
>     stages {
>         stage('Checkout') {
>             steps { checkout scm }
>         }
>
>         stage('Test') {
>             steps {
>                 sh "mvn clean test -P${params.SUITE} -Denv=${params.ENV} -Dbrowser=${params.BROWSER}"
>             }
>         }
>     }
>
>     post {
>         always {
>             testNG()                                    // publish TestNG results
>             archiveArtifacts artifacts: 'reports/**/*', allowEmptyArchive: true
>             publishHTML([reportDir: 'reports', reportFiles: 'ExtentReport.html',
>                          reportName: 'Extent Report'])
>         }
>         failure {
>             emailext to: 'qa-team@example.com',
>                      subject: "Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
>                      body: "${env.BUILD_URL}"
>         }
>     }
> }
> ```
>
> ---
>
> **The details that matter:**
>
> **`post { always { ... } }`** — the same principle as `if: always()` in GitHub Actions and `succeededOrFailed()` in Azure DevOps. Without it, the archive and publish steps are skipped when the build fails, which is exactly when you need the report. It's the most common CI mistake across every platform.
>
> **`buildDiscarder`** — screenshots and reports accumulate fast. Thirty builds of retention is usually enough, and without it the Jenkins master fills up. That's a real operational failure people hit.
>
> **`timeout`** — a hung browser session will otherwise occupy an executor indefinitely.
>
> **Jenkinsfile in the repo**, not job configuration in the UI. Version-controlled with the tests, reviewable, and recoverable if the Jenkins instance dies. Job config living only in the UI is a genuine single point of failure.
>
> ---
>
> **Headless execution — the practical necessity:**
>
> ```java
> ChromeOptions options = new ChromeOptions();
> if (Boolean.parseBoolean(System.getProperty("headless", "true"))) {
>     options.addArguments("--headless=new");
> }
> options.addArguments("--no-sandbox");
> options.addArguments("--disable-dev-shm-usage");
> options.addArguments("--window-size=1920,1080");
> ```
>
> Build agents typically have no display, so headless is required. The flags matter:
>
> `--disable-dev-shm-usage` — containers default to a small `/dev/shm`, which crashes Chrome on heavy pages. Exactly the same issue as the Playwright container note.
>
> `--window-size` explicitly — headless defaults to a small viewport, so responsive layouts render differently than locally. That's a classic source of "passes locally, fails in CI" where the element genuinely isn't visible because the nav collapsed into a hamburger menu.
>
> ---
>
> **Reporting integration:**
>
> The `testNG()` step consumes TestNG's XML output and populates Jenkins' native test dashboard — pass/fail counts and trends across builds. Without it, results only exist in console output, which nobody reads and which gives you no trend visibility.
>
> `publishHTML` surfaces the ExtentReport in the build page. Note it needs the HTML Publisher plugin and Jenkins' content security policy sometimes blocks report styling, which catches people out.
>
> ---
>
> **Job strategy:**
>
> - **Smoke on deployment** — triggered by the deploy job, blocking, fast
> - **Full regression nightly** — cron scheduled
> - **Cross-browser weekly** — the slow suite, non-blocking
>
> Same tiering principle as everywhere: running less more often beats running everything rarely.
>
> ---
>
> **Notifications:**
>
> Email or Slack on failure only. Notifying on every build means people filter it, and then the failure that mattered gets missed too. That's a discipline point rather than a technical one, but it determines whether the pipeline actually influences behaviour."

---

## Pipeline Essentials

| Element | Why |
|---------|-----|
| Jenkinsfile in repo | Version-controlled, recoverable |
| `parameters` | One job, multiple purposes |
| `post { always }` | Reports survive failed builds |
| `buildDiscarder` | Screenshots fill the disk otherwise |
| `timeout` | Hung sessions hold executors |
| `testNG()` | Native dashboard and trends |
| Headless + window size | Agents have no display |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Jenkins ran the Selenium suite — smoke triggered on deployment and full regression on a nightly schedule. The pipeline was parameterized for environment, browser, and suite, so one job served several purposes by passing different Maven profiles.
>
> The detail I'd emphasize is `post { always }` on the archive and publish steps. Without it those steps skip when the build fails, which is precisely when you need the screenshots and the report. It's the same mistake as forgetting `if: always()` in GitHub Actions — different syntax, identical self-defeating outcome.
>
> Headless execution needed explicit window sizing. Headless Chrome defaults to a small viewport, so responsive layouts render differently than they do locally — the nav collapses into a hamburger menu and elements you expect aren't there. That's a classic 'passes locally, fails in CI' where the element genuinely isn't visible, and setting `--window-size` explicitly removes it.
>
> `--disable-dev-shm-usage` was the other necessary flag, because containers default to a small shared memory allocation and Chrome crashes on heavy pages without it.
>
> On retention, `buildDiscarder` matters more than it sounds. Screenshots and reports accumulate quickly and will fill the Jenkins master if nothing prunes them."

---

## Common Interview Questions & Answers

### Q1: "How do you run your suite in Jenkins?"

**A:** "Declarative pipeline in a Jenkinsfile kept in the repo, parameterized for environment, browser, and suite so one job serves multiple purposes. It runs `mvn test` with the corresponding profile, then publishes TestNG results and archives reports in a `post { always }` block so they survive failures."

### Q2: "Why keep the Jenkinsfile in the repo rather than configuring the job in the UI?"

**A:** "Version control. It's reviewed with the tests, changes are traceable, and it's recoverable if the Jenkins instance is lost. Job configuration living only in the UI is a single point of failure and there's no history of who changed what."

### Q3: "Tests pass locally but fail in Jenkins. What do you check first?"

**A:** "Viewport. Headless Chrome defaults to a small window, so responsive layouts render differently — the nav collapses and elements genuinely aren't visible. Setting `--window-size` explicitly fixes a surprising share of it. After that, environment configuration and data differences between the local and CI targets."

### Q4: "How do you make Jenkins failures debuggable?"

**A:** "Archive screenshots and the ExtentReport with `post { always }`, and publish TestNG results so the dashboard shows pass rates and trends rather than just the current build. Without the always block, artifacts are skipped on failure, which is the one case you needed them."

### Q5: "How is your job scheduling structured?"

**A:** "Smoke triggered on deployment — fast and blocking. Full regression nightly on cron. Slower cross-browser runs weekly and non-blocking. Same tiering as I'd apply anywhere: running less more often beats running everything rarely, and it's free compared to adding agents."

---

## Weak Areas to Watch

❌ **Don't omit `post { always }`**
→ Artifacts skipped exactly when needed.

❌ **Don't configure jobs only in the UI**
→ No version control, no recovery.

❌ **Don't run headless without setting window size**
→ Responsive layouts differ; classic CI-only failure.

❌ **Don't skip `buildDiscarder`**
→ Screenshots fill the master.

❌ **Don't notify on every build**
→ Alert fatigue; the real failure gets missed.

---

## Key Takeaway for Interview

CI is what makes the suite operational. Artifacts are what make it maintainable.

The cross-platform principle:

> **"`post { always }` on archiving — same as `if: always()` in GitHub Actions and `succeededOrFailed()` in Azure DevOps. Skip it and reports vanish exactly when tests fail."**

And the CI-specific gotcha:

> **"Headless Chrome defaults to a small viewport, so responsive layouts collapse and elements genuinely aren't visible. Setting `--window-size` explicitly removes a whole class of 'passes locally, fails in CI'."**

---

## Checklist

✅ Jenkinsfile version-controlled in the repo
✅ Parameterized for env, browser, suite
✅ `post { always }` on archive and publish
✅ `buildDiscarder` configured
✅ `timeout` set
✅ TestNG results published for trends
✅ Headless with explicit `--window-size`
✅ `--disable-dev-shm-usage` in containers
✅ Notifications on failure only

---

**Remember:** A CI failure with no archived screenshot is a guessing game. 🎯
