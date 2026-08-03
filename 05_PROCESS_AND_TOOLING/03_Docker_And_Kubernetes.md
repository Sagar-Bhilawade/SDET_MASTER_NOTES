# Docker & Kubernetes for Test Execution

## What This Covers

Containerized test execution and Kubernetes-based environments — both on your resume, and central to the BASES_PLAYWRIGHT project.

The Playwright-specific container detail is in `02_PLAYWRIGHT/05_CICD_AND_EXECUTION/03_Docker_And_Environments`. This note is the broader concept, and the Kubernetes half that isn't covered anywhere else.

---

## Why It Matters (Interview Context)

Kubernetes is on your resume and in your project description — *"Integrated Azure CI/CD pipelines and GitHub Actions for microservices within Kubernetes clusters."* That's an invitation to be asked about it.

The risk: describing K8s in generic terms rather than from the testing angle. What they want is **what changes about testing when the app runs in Kubernetes.**

---

## Your 30-Second Interview Answer

> "Docker gives identical browsers, fonts and OS libraries everywhere, which removes a whole category of local-versus-CI differences rather than debugging them one at a time. Kubernetes matters because it changes what you're testing against — a microservices deployment means the environment is ephemeral, services can be mid-rollout while your tests run, and 'is the app ready' is a real question rather than an assumption. The practical consequence is that readiness checks before the suite starts matter far more than they do against a static environment."

---

## Your 2-Minute Deep Answer

> "**Docker — consistency:**
>
> The value for testing is eliminating environment variables rather than debugging them. Identical browser binaries, identical fonts, identical OS libraries.
>
> ```dockerfile
> FROM mcr.microsoft.com/playwright:v1.49.0-noble
> WORKDIR /app
> COPY package*.json ./
> RUN npm ci
> COPY . .
> CMD ["npx", "playwright", "test"]
> ```
>
> Two rules that matter:
>
> **Match the image tag to the package version.** A mismatch means the library and browser binaries disagree and you get failures that point nowhere useful. Worth enforcing with a CI check rather than trusting memory.
>
> **`--shm-size=1gb` or `--ipc=host`.** Containers default to a small `/dev/shm`, which crashes Chromium on heavy pages. The error is just a browser crash, so it's genuinely hard to diagnose without knowing.
>
> Font rendering is the concrete win — screenshot baselines aren't portable across operating systems, so visual testing only works if baselines are generated in the same image CI uses.
>
> ---
>
> **Kubernetes — what actually changes for testing:**
>
> This is the part worth being specific about, because generic K8s knowledge isn't what they're asking for.
>
> **1. The environment is ephemeral.**
>
> Pods restart, get rescheduled, scale up and down. A test suite that assumes a stable long-lived environment will hit failures that have nothing to do with the application logic — a pod restarted mid-run, and your session died with it.
>
> **2. Deployments are gradual.**
>
> A rolling update means old and new pods serve traffic simultaneously. If your tests start while a rollout is in progress, some requests hit the old version and some the new. That produces genuinely confusing intermittent failures.
>
> The fix is checking readiness before the suite runs, not just that the deployment command returned:
>
> ```bash
> kubectl rollout status deployment/my-app -n staging --timeout=300s
> ```
>
> That waits for the rollout to actually complete. Running tests without it is a real source of CI-only flakiness — and it's a good answer to "passes locally, fails in CI" that most people don't reach for.
>
> **3. Service discovery and internal endpoints.**
>
> Services talk to each other by internal DNS names. Tests running inside the cluster can reach those; tests running outside need an ingress or port-forward. That distinction determines what you can test.
>
> ```bash
> kubectl port-forward svc/my-api 8080:80 -n staging
> ```
>
> **4. Logs are ephemeral too.**
>
> When a pod dies, its logs go with it unless they're shipped somewhere. So when a test fails against a K8s environment, `kubectl logs` may show you nothing useful if the pod already restarted. That's an argument for centralized logging, and for capturing correlation IDs in your test output so you can find the request later.
>
> ---
>
> **Running tests IN the cluster:**
>
> You can run the suite as a Kubernetes Job — a pod that runs to completion:
>
> ```yaml
> apiVersion: batch/v1
> kind: Job
> metadata:
>   name: e2e-tests
> spec:
>   backoffLimit: 0
>   template:
>     spec:
>       restartPolicy: Never
>       containers:
>         - name: tests
>           image: myregistry/e2e-tests:1.0.0
>           env:
>             - name: BASE_URL
>               value: "http://my-app-service"
> ```
>
> The advantage is reaching internal services directly by their cluster DNS names, without ingress. The trade-off is that getting artifacts out is harder — you need a volume mount or to ship them somewhere, because when the pod finishes the filesystem goes.
>
> `backoffLimit: 0` matters — otherwise a failed test job gets retried by Kubernetes, which isn't what you want when the failure is a genuine test failure.
>
> ---
>
> **The honest boundary:**
>
> I'd be straightforward that my Kubernetes work is from the testing and CI/CD integration side — pipelines deploying to clusters, tests running against those environments, understanding what breaks and why. I'm not administering clusters. Overclaiming there would unravel quickly, and the testing angle is what the role actually needs."

---

## What Changes When Testing Against K8s

| Aspect | Static environment | Kubernetes |
|--------|-------------------|------------|
| Stability | Long-lived | Pods restart, reschedule |
| Deployment | Atomic | Rolling — old and new coexist |
| Readiness | Assumed | Must be checked explicitly |
| Endpoints | Fixed URLs | Internal DNS, ingress, port-forward |
| Logs | Persistent | Ephemeral unless shipped |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "On BASES_PLAYWRIGHT the application runs as microservices in Kubernetes, with Azure DevOps and GitHub Actions handling the pipeline. The thing that changes about testing is that the environment isn't a fixed target.
>
> The concrete example is rolling deployments. If tests start while a rollout is still in progress, some requests hit old pods and some hit new ones — and you get intermittent failures that look like flakiness and aren't. The fix is waiting on `kubectl rollout status` before the suite runs, rather than assuming the deployment command returning means the app is ready. That's a genuinely common source of CI-only failures that people don't think to look for.
>
> The other thing is that logs are ephemeral. When a pod restarts, its logs go with it — so a failure investigated an hour later may have nothing left to look at. That's why correlation IDs in test output matter: you can find the request in centralized logging even after the pod is gone.
>
> On Docker, the win is eliminating environment differences rather than debugging them. Identical browsers and fonts everywhere — which is what makes screenshot baselines portable at all.
>
> I'd be clear my Kubernetes work is from the testing and pipeline integration side rather than cluster administration."

---

## Common Interview Questions & Answers

### Q1: "Why run tests in Docker?"

**A:** "Consistency — identical browsers, fonts and OS libraries everywhere, which eliminates a category of local-versus-CI failures structurally instead of debugging them individually. The concrete case is font rendering: screenshot baselines aren't portable across operating systems, so visual testing only works if baselines come from the same image CI uses."

### Q2: "What changes about testing when the app runs in Kubernetes?"

**A:** "The environment stops being a fixed target. Pods restart and reschedule, so a session can die mid-run for infrastructure reasons. Deployments are rolling, so old and new versions serve traffic simultaneously — if tests start during a rollout you get genuinely confusing intermittent failures. And logs are ephemeral, so a failure investigated later may have nothing to look at."

### Q3: "How do you make sure the app is ready before tests run?"

**A:** "`kubectl rollout status` with a timeout, rather than assuming the deploy command returning means it's ready. Plus a health-check poll on the actual endpoint. Skipping that is a real source of CI-only flakiness, and it's one people don't think to check because it doesn't look like a test problem."

### Q4: "Would you run the test suite inside the cluster?"

**A:** "It's an option — a Kubernetes Job — and the advantage is reaching internal services by cluster DNS without needing ingress. The trade-off is getting artifacts out, since the pod filesystem disappears when it completes, so you need a volume or to ship them. And `backoffLimit: 0`, or Kubernetes retries a job that failed because a test genuinely failed."

### Q5: "What's the most common Docker mistake with browser testing?"

**A:** "Two. Version mismatch between the image tag and the test library package — the browsers and library disagree and you get failures that point nowhere. And not setting `--shm-size`, because the default shared memory in a container crashes Chromium on heavy pages, and the error is just a browser crash with no useful signal."

---

## Weak Areas to Watch

❌ **Don't describe Kubernetes generically**
→ They want what changes about *testing*, not what K8s is.

❌ **Don't assume deploy-returned means ready**
→ Rolling updates mean old and new coexist.

❌ **Don't overclaim cluster administration**
→ Testing and pipeline integration is the honest scope.

❌ **Don't forget `--shm-size`**
→ Chromium crashes with no useful error.

❌ **Don't mismatch image tag and package version**
→ Failures that point nowhere.

---

## Key Takeaway for Interview

Kubernetes is on your resume — answer it from the testing angle.

The specific insight that lands:

> **"If tests start while a rolling deployment is still in progress, some requests hit old pods and some hit new. That's intermittent failure that looks like flakiness and isn't — the fix is waiting on `kubectl rollout status`, not retrying the test."**

And the Docker rule:

> **"Match the image tag to the package version, and set `--shm-size`. A version mismatch and a Chromium crash from default shared memory both produce failures that point nowhere near the cause."**

---

## Checklist

✅ Image tag matched to package version, enforced in CI
✅ `--shm-size=1gb` or `--ipc=host` set
✅ `kubectl rollout status` before the suite runs
✅ Health check on the real endpoint, not just deploy success
✅ Correlation IDs captured for post-hoc log lookup
✅ `backoffLimit: 0` on test Jobs
✅ Artifact extraction planned if running in-cluster
✅ Honest scope: testing and pipeline integration, not cluster admin

---

**Remember:** In Kubernetes, "the deploy finished" and "the app is ready" are different statements. 🎯
