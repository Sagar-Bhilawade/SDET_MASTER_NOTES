# Troubleshooting - "It Passes Locally But Fails in CI"

## The Problem

The most common complaint in test automation, and the one that wastes the most hours. Same code, same test, green on your laptop, red in the pipeline.

This note is a **diagnostic playbook** — the order to check things in, and what each symptom points to.

---

## Why It Matters (Interview Context)

Expect this almost verbatim:

> "A test passes on your machine but fails in CI. Walk me through how you'd debug it."

What they're evaluating is whether you have a **method** or whether you flail. The wrong answer is a list of random things to try. The right answer is an ordered set of hypotheses, cheapest to check first.

---

## Your 30-Second Interview Answer

> "I'd start with the trace from the CI run rather than trying to reproduce blind — Playwright gives me the exact failing execution, so I don't have to guess. Then I work through the differences between the environments in order of likelihood: timing and machine load, data volume, viewport size, and environment configuration. Nine times out of ten it's one of those four, and the trace usually points straight at which."

---

## Your 2-Minute Deep Answer

> "The core insight is that 'passes locally, fails in CI' is never really about CI. It's about the test having a hidden dependency on something that happens to be true on your laptop. The job is finding that assumption.
>
> **First move: get the trace. Don't reproduce blind.**
>
> ```bash
> npx playwright show-trace trace.zip
> ```
>
> The trace shows every action, every network call, the DOM at each step, and the exact assertion that failed. Debugging without it is guesswork, and this is the single biggest advantage Playwright has over what I used with Selenium — there I'd have a stack trace and a screenshot and I'd be inferring the rest.
>
> **Then work the differences in order:**
>
> ---
>
> **1. TIMING AND MACHINE LOAD** *(most common)*
>
> Your laptop is fast, idle, and single-tenant. A CI runner is shared, loaded, and often throttled. Anything with an implicit timing assumption surfaces here.
>
> *Signature:* Timeout errors. Assertions on content that hasn't arrived yet.
> *Root cause:* Usually waiting on the element rather than the data behind it.
> *Fix:* Wait on the actual condition.
> ```typescript
> await page.waitForResponse(r => r.url().includes('/api/items') && r.ok());
> await expect(page.getByTestId('row')).toHaveCount(5);
> ```
> *Not the fix:* Raising the timeout. That converts a fast failure into a slow one.
>
> ---
>
> **2. DATA VOLUME AND STATE**
>
> Your dev database has 12 records. The CI environment has 40,000, or 3, or a seeded set from last month.
>
> *Signature:* `strict mode violation — resolved to 15 elements`. Pagination appearing that wasn't there locally. "First row" being a different row.
> *Fix:* Test creates its own data. Never assert against whatever happens to be in the environment.
>
> This one is worth calling out specifically because the failure message often looks like a locator problem when it's actually a data problem.
>
> ---
>
> **3. VIEWPORT SIZE**
>
> Local runs headed at your monitor's size. CI runs headless at a default that's often smaller.
>
> *Signature:* Element not visible. Click intercepted. A nav collapsed into a hamburger menu that doesn't exist locally.
> *Fix:* Pin it in config so both environments match.
> ```typescript
> use: { viewport: { width: 1280, height: 720 } }
> ```
>
> ---
>
> **4. ENVIRONMENT CONFIGURATION**
>
> Different `baseURL`, missing env var, different feature flags, different auth provider.
>
> *Signature:* 401s, 404s, redirects to an unexpected page, a feature that isn't there.
> *Fix:* Fail loudly at startup if required config is absent, rather than failing mysteriously mid-test.
>
> ---
>
> **5. PARALLELISM DIFFERENCES**
>
> You ran one test locally. CI ran it alongside 300 others, some of which touch the same data.
>
> *Signature:* Passes when run alone in CI, fails in the full suite.
> *Diagnostic:* Run the single test in CI. If it passes, it's interference, not environment.
> *Fix:* Data isolation.
>
> ---
>
> **6. HEADED vs HEADLESS**
>
> Genuinely rare now — Playwright's headless mode is close to identical. But it exists, mostly around animations, fonts, and media.
>
> *Diagnostic:* `npx playwright test --headed` locally to compare.
>
> ---
>
> **The reproduction trick that saves the most time:**
>
> Rather than debugging in CI, make your laptop behave like CI:
>
> ```bash
> CI=true npx playwright test failing.spec.ts --repeat-each=10 --workers=8
> ```
>
> `CI=true` picks up your CI-specific config. High worker count creates resource contention. `--repeat-each` surfaces intermittency. That combination reproduces a large share of CI-only failures locally, where the feedback loop is seconds instead of minutes.
>
> And when the environment difference genuinely can't be replicated, running the tests in the same container image CI uses removes the variable entirely."

---

## The Diagnostic Table

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Timeout waiting for element | Machine slower, data not loaded | Wait on the response, not the element |
| `strict mode violation` | Different data volume | Test owns its data |
| Element not visible / click intercepted | Viewport difference | Pin viewport in config |
| 401 / 404 / unexpected redirect | Config or env var missing | Fail fast on missing config |
| Passes alone in CI, fails in suite | Test interference | Data isolation |
| Screenshot mismatch | Font/rendering difference | Run in the CI container image |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "We run on Azure DevOps VMs, and the environment gap is real — the runner is a shared, loaded machine with a very different dataset than a local dev environment.
>
> My workflow is trace-first. Rather than trying to reproduce a CI failure by guessing, I pull the trace artifact and replay it locally. That collapses the debugging time enormously, because I'm looking at what actually happened — the network calls, the DOM state, the failing assertion — rather than theorizing about it.
>
> The pattern I've seen most is tests that were implicitly relying on the local dataset. They'd assert against 'the first row' and pass locally where there were a handful of records, then fail in an environment with a different volume or ordering. The failure surfaces as a locator error, which sends people looking in the wrong place. The actual fix is data ownership — the test creates what it needs so the environment's contents stop mattering.
>
> For reproduction, running locally with `CI=true` and a high worker count catches a good share of it before it ever reaches the pipeline."

---

## Common Interview Questions & Answers

### Q1: "Test passes locally, fails in CI. Walk me through it."

**A:** "Trace first — Playwright gives me the exact failing run, so I'm not reproducing blind. Then I check environment differences in likelihood order: machine speed and load, data volume, viewport, config. The trace usually narrows it immediately, because I can see whether the element was missing, the response was late, or the request failed outright."

### Q2: "The trace shows a timeout on an element. What now?"

**A:** "I'd check whether the data behind it had arrived. Auto-waiting handles the element appearing — it doesn't wait for the API response that populates it. A list can be visible and empty. If the trace shows the request still in flight when the assertion ran, the fix is waiting on the response, not extending the timeout."

### Q3: "How do you reproduce a CI-only failure?"

**A:** "Make the local environment resemble CI: `CI=true` to pick up the CI config, high worker count to create contention, `--repeat-each` to surface intermittency. That catches most of it. If it's genuinely environmental, running in the same container image removes the difference rather than approximating it."

### Q4: "Would you just increase the timeout?"

**A:** "Only if I'd established the operation is legitimately slow — a heavy report generation, for example. Otherwise raising the timeout hides the cause and makes the eventual failure slower. If my fix is a timeout bump and I can't explain *why* it was slow, I haven't finished diagnosing."

### Q5: "The test passes when run alone in CI but fails in the full suite. What does that tell you?"

**A:** "That it's interference, not environment — which is actually good news, because it rules out half the search space. Something else in the suite is touching shared state or shared data. The fix is isolation: the test creates its own data with a unique identifier rather than relying on anything ambient."

---

## Weak Areas to Watch

❌ **Don't debug without the trace**
→ Playwright's biggest debugging advantage; not using it looks like you don't know it exists.

❌ **Don't default to raising timeouts**
→ Symptom management. Name the reason it was slow.

❌ **Don't assume the environments are identical**
→ Data volume, viewport, and load differ by default.

❌ **Don't skip the "run it alone" diagnostic**
→ It's free and it eliminates an entire category in one run.

---

## Key Takeaway for Interview

This is a **method** question. They want an ordered set of hypotheses, not a list of tricks.

The framing that lands:

> **"Passing locally and failing in CI is never really about CI. It means the test has a hidden dependency on something that happens to be true on my machine — usually data volume or timing. The job is finding the assumption."**

And the practical one:

> **"Trace first. I'm not going to theorize about a failure I can replay."**

---

## Diagnostic Checklist

✅ Pull and replay the CI trace before anything else
✅ Check for data-volume assumptions (`strict mode violation` is the tell)
✅ Confirm viewport is pinned in config
✅ Verify env vars and `baseURL` for the CI environment
✅ Run the test alone in CI to isolate interference
✅ Reproduce locally with `CI=true --repeat-each=10 --workers=8`
✅ Use the CI container image when the difference is environmental

---

**Remember:** The test isn't failing in CI. It's failing everywhere — CI is just the place that noticed. 🎯
