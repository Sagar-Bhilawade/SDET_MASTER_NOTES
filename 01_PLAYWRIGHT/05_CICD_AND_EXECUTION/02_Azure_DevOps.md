# CI/CD - Azure DevOps Pipelines with Playwright

## What This Covers

Running Playwright in Azure DevOps — pipeline YAML, the one setting that catches everyone, and publishing results so they show up in the Azure test dashboard.

This is your actual CI platform at NielsenIQ, so this is the one to be fluent in.

---

## Why It Matters (Interview Context)

If you say "Azure DevOps pipelines on VMs" in your project story, expect a follow-up. Being able to describe the YAML, artifacts, and test result publishing turns a claim into demonstrated experience.

The single detail that proves hands-on use: **Azure DevOps doesn't set `CI=true` automatically.** GitHub Actions does. Anyone who has actually run Playwright on Azure has hit this.

---

## Your 30-Second Interview Answer

> "Azure DevOps pipeline is Node setup, `npm ci`, install browsers with `--with-deps`, run tests, publish results and artifacts. The one thing that catches people is that Azure doesn't set `CI=true` automatically the way GitHub Actions does — you have to set it explicitly, otherwise the HTML reporter tries to open a browser and the pipeline hangs after tests finish. Beyond that, publishing JUnit XML is what makes results show up in the Azure test dashboard rather than being buried in logs."

---

## Your 2-Minute Deep Answer

> "The baseline pipeline:
>
> ```yaml
> trigger:
>   branches:
>     include: [main]
>
> pool:
>   vmImage: 'ubuntu-latest'
>
> variables:
>   CI: 'true'                    # ← critical, see below
>
> steps:
>   - task: NodeTool@0
>     inputs:
>       versionSpec: '20.x'
>
>   - script: npm ci
>     displayName: 'Install dependencies'
>
>   - script: npx playwright install --with-deps
>     displayName: 'Install browsers'
>
>   - script: npx playwright test
>     displayName: 'Run Playwright tests'
>     env:
>       BASE_URL: $(BASE_URL)
>       API_TOKEN: $(API_TOKEN)
>
>   - task: PublishTestResults@2
>     condition: succeededOrFailed()
>     inputs:
>       testResultsFormat: 'JUnit'
>       testResultsFiles: 'test-results/junit.xml'
>       testRunTitle: 'Playwright Tests'
>
>   - task: PublishBuildArtifacts@1
>     condition: succeededOrFailed()
>     inputs:
>       pathToPublish: 'playwright-report'
>       artifactName: 'playwright-report'
> ```
>
> ---
>
> **The `CI: 'true'` issue — worth understanding properly:**
>
> Playwright keys several behaviours off the `CI` environment variable: retries, `forbidOnly`, and — the one that bites — whether the HTML reporter auto-opens a browser when the run finishes.
>
> GitHub Actions sets `CI=true` automatically. **Azure DevOps does not.** So without setting it explicitly:
> - The HTML reporter tries to serve and open a report, and the pipeline hangs until it times out
> - `retries: process.env.CI ? 2 : 0` evaluates to 0, so you get no retries and no trace-on-retry
> - `forbidOnly: !!process.env.CI` is false, so a committed `test.only()` silently runs one test and reports green
>
> That last one is the dangerous consequence. Someone commits a `test.only()` from local debugging, the pipeline runs a single test, everything is green, and the actual suite is broken on main. `forbidOnly` exists to catch exactly that, and it's inert without `CI` set.
>
> ---
>
> **`condition: succeededOrFailed()`**
>
> This is Azure's equivalent of GitHub's `if: always()`. Without it, publishing steps are skipped when tests fail — which is when you need the report. Same self-defeating mistake, different syntax.
>
> ---
>
> **Publishing test results properly:**
>
> ```typescript
> // playwright.config.ts
> reporter: [
>   ['html', { outputFolder: 'playwright-report', open: 'never' }],
>   ['junit', { outputFile: 'test-results/junit.xml' }],
>   ['list'],
> ],
> ```
>
> The JUnit XML is what `PublishTestResults@2` consumes, and it's what makes results appear in the Azure **Tests** tab — pass/fail counts, trends over time, flaky test detection. Without it, results only exist in raw log output, which nobody reads.
>
> `open: 'never'` on the HTML reporter is belt-and-braces alongside `CI=true`.
>
> ---
>
> **Parallel jobs across browsers:**
>
> ```yaml
> jobs:
>   - job: Chromium
>     steps: [ ... , script: npx playwright test --project=chromium ]
>
>   - job: Firefox
>     dependsOn: []          # empty = runs in parallel
>     steps: [ ... , script: npx playwright test --project=firefox ]
> ```
>
> `dependsOn: []` is the Azure idiom for "don't wait for anything" — jobs run concurrently. Leave it off and Azure serializes them by default in some configurations.
>
> ---
>
> **Sharding with a matrix:**
>
> ```yaml
> strategy:
>   matrix:
>     shard1: { SHARD: '1/4' }
>     shard2: { SHARD: '2/4' }
>     shard3: { SHARD: '3/4' }
>     shard4: { SHARD: '4/4' }
>
> steps:
>   - script: npx playwright test --shard=$(SHARD)
> ```
>
> ---
>
> **Secrets:**
>
> Variable groups linked to Azure Key Vault, marked secret so they're masked in logs. Referenced as `$(VARIABLE_NAME)` and passed through `env:`. Never in the YAML file itself.
>
> ---
>
> **Azure DevOps vs GitHub Actions, having used both:**
>
> Conceptually the same — checkout, install, run, publish. The differences are ergonomic: Azure's `task:` steps are more verbose than GitHub's `uses:` actions, but the built-in test results dashboard is genuinely better than anything GitHub gives you out of the box. Azure surfaces pass rates and flaky test trends natively, which is useful when you're tracking suite health over time rather than just per-run status."

---

## GitHub Actions vs Azure DevOps

| | GitHub Actions | Azure DevOps |
|---|---|---|
| `CI=true` | Automatic | **Must set manually** |
| Run on failure | `if: always()` | `condition: succeededOrFailed()` |
| Reusable step | `uses: action@v4` | `task: Name@version` |
| Parallel jobs | Default | `dependsOn: []` |
| Test dashboard | Basic | Built-in, with trends |
| Secrets | GitHub Secrets | Variable groups / Key Vault |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Our architecture is GitHub Actions handling deployment triggers and Azure DevOps orchestrating test execution on VMs — 300+ tests running in parallel, completing in 8-10 minutes.
>
> The gotcha I'd mention because it demonstrates hands-on rather than theoretical experience: Azure doesn't set `CI=true` automatically the way GitHub Actions does. Without it the HTML reporter tries to open a browser and the pipeline hangs after tests complete. But the more serious consequence is that `forbidOnly` goes inert — someone commits a `test.only()` from local debugging, the pipeline runs one test, reports green, and main is actually broken. That's the failure mode `forbidOnly` exists to prevent, and it does nothing unless `CI` is set.
>
> The other thing I'd highlight is publishing JUnit XML through `PublishTestResults`. That's what makes results appear in Azure's Tests tab with pass rates and trends over time, rather than being buried in log output. Given that I care about first-attempt pass rate as a health metric, having that surfaced natively matters."

---

## Common Interview Questions & Answers

### Q1: "How do you run Playwright in Azure DevOps?"

**A:** "NodeTool for the Node version, `npm ci`, `npx playwright install --with-deps`, run tests, then `PublishTestResults` for the JUnit XML and `PublishBuildArtifacts` for the HTML report and traces. Both publishing steps need `condition: succeededOrFailed()` so they run when tests fail — which is when you actually need them."

### Q2: "What's different about Azure DevOps versus GitHub Actions?"

**A:** "The one that matters practically is that Azure doesn't set `CI=true` automatically. That's not cosmetic — without it the HTML reporter hangs the pipeline, and more importantly `forbidOnly` and retries go inert, so a committed `test.only()` runs one test and reports green. Beyond that, Azure's test results dashboard is better out of the box; GitHub's step syntax is less verbose."

### Q3: "How do results show up in the Azure dashboard?"

**A:** "Configure the JUnit reporter in Playwright config, then consume that XML with `PublishTestResults@2`. That populates the Tests tab with pass/fail counts and trends across runs. Without it results only exist in raw logs, which means nobody's tracking suite health over time."

### Q4: "How do you handle secrets in Azure Pipelines?"

**A:** "Variable groups linked to Key Vault, marked secret so they're masked in logs, referenced as `$(NAME)` and passed through `env:`. Never in the YAML. Same principle as anywhere — the pipeline definition is in source control, so anything in it is effectively public to everyone with repo access."

### Q5: "How do you run tests across browsers in parallel?"

**A:** "Separate jobs per project with `dependsOn: []`, which is Azure's way of saying don't wait for anything. Each job runs its own project and internally parallelizes across workers. So Chromium and Firefox suites run concurrently rather than sequentially, which roughly halves wall-clock time for cross-browser coverage."

---

## Weak Areas to Watch

❌ **Don't forget `CI: 'true'`**
→ Pipeline hangs, and `forbidOnly` silently stops protecting you.

❌ **Don't omit `condition: succeededOrFailed()`**
→ Reports skipped exactly when tests fail.

❌ **Don't skip JUnit publishing**
→ Results stay buried in logs with no trend tracking.

❌ **Don't put secrets in YAML**
→ The pipeline file is in source control.

---

## Key Takeaway for Interview

Azure specifics are where you demonstrate hands-on rather than theoretical CI experience.

The detail that proves it:

> **"Azure doesn't set `CI=true` automatically like GitHub Actions does. The visible symptom is the HTML reporter hanging the pipeline — but the serious one is `forbidOnly` going inert, so a committed `test.only()` runs one test and reports green while main is broken."**

And the operational one:

> **"Publishing JUnit XML is what puts results in Azure's Tests tab with trends. Without it you have per-run pass/fail in logs and no way to see the suite decaying."**

---

## Checklist

✅ `CI: 'true'` set in pipeline variables
✅ `--with-deps` on browser install
✅ `condition: succeededOrFailed()` on both publish steps
✅ JUnit reporter configured and published
✅ HTML report published as build artifact
✅ `open: 'never'` on the HTML reporter
✅ Secrets from variable groups / Key Vault
✅ `dependsOn: []` for parallel browser jobs

---

**Remember:** `forbidOnly` protects nothing if `CI` isn't set. 🎯
