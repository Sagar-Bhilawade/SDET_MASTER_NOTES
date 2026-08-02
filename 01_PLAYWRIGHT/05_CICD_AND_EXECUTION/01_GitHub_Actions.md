# CI/CD - GitHub Actions with Playwright

## What This Covers

Running Playwright in GitHub Actions — workflow structure, caching, sharding, and artifacts.

---

## Why It Matters (Interview Context)

CI questions test whether your automation is **operational** or just local. A suite that only runs on your machine isn't automation, it's a script collection.

They're checking:
1. Can you write a working pipeline?
2. Do you handle artifacts so failures are debuggable?
3. Do you understand sharding and when it's worth the cost?
4. Do you think about pipeline speed and caching?

---

## Your 30-Second Interview Answer

> "The core workflow is checkout, Node setup, install dependencies, install browsers with `--with-deps`, run tests, upload artifacts. The parts that matter beyond the basics are caching browsers so you're not downloading them every run, uploading traces and reports on failure so CI failures are actually debuggable, and sharding across jobs when one runner becomes the bottleneck. Uploading artifacts is the one people skip, and it's what turns a red build from a mystery into a five-minute fix."

---

## Your 2-Minute Deep Answer

> "The baseline workflow:
>
> ```yaml
> name: Playwright Tests
> on:
>   push:
>     branches: [main]
>   pull_request:
>
> jobs:
>   test:
>     runs-on: ubuntu-latest
>     timeout-minutes: 30
>     steps:
>       - uses: actions/checkout@v4
>
>       - uses: actions/setup-node@v4
>         with:
>           node-version: 20
>           cache: 'npm'
>
>       - run: npm ci
>
>       - run: npx playwright install --with-deps
>
>       - run: npx playwright test
>         env:
>           BASE_URL: ${{ secrets.BASE_URL }}
>
>       - uses: actions/upload-artifact@v4
>         if: always()
>         with:
>           name: playwright-report
>           path: playwright-report/
>           retention-days: 7
> ```
>
> **The details that matter:**
>
> **`--with-deps`** installs the OS-level libraries browsers need, not just the browsers. Skip it and you get cryptic launch failures on a clean runner — a very common first-time CI problem.
>
> **`if: always()`** on the artifact upload. Without it the step is skipped when tests fail — which is exactly when you need the report. This is the single most common mistake in Playwright CI setups, and it's self-defeating: artifacts exist for failures.
>
> **`timeout-minutes`** so a hung run doesn't burn an hour of runner time.
>
> **Secrets via `env`**, never committed. GitHub Actions sets `CI=true` automatically, which is what your config keys off for retries and `forbidOnly`.
>
> ---
>
> **Caching browsers — a real speed win:**
>
> ```yaml
> - uses: actions/cache@v4
>   id: playwright-cache
>   with:
>     path: ~/.cache/ms-playwright
>     key: playwright-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
>
> - if: steps.playwright-cache.outputs.cache-hit != 'true'
>   run: npx playwright install --with-deps
>
> - if: steps.playwright-cache.outputs.cache-hit == 'true'
>   run: npx playwright install-deps
> ```
>
> Browser download is a meaningful chunk of every run. The key point is that the cache key includes the lockfile hash, so when Playwright's version changes you get fresh browsers rather than a stale mismatch — that mismatch produces genuinely baffling failures.
>
> Note the `install-deps` on cache hit: browsers are cached but OS dependencies aren't.
>
> ---
>
> **Sharding across runners:**
>
> ```yaml
> jobs:
>   test:
>     strategy:
>       fail-fast: false
>       matrix:
>         shard: [1, 2, 3, 4]
>     steps:
>       # ... setup ...
>       - run: npx playwright test --shard=${{ matrix.shard }}/4
>       - uses: actions/upload-artifact@v4
>         if: always()
>         with:
>           name: blob-report-${{ matrix.shard }}
>           path: blob-report/
>
>   merge-reports:
>     needs: [test]
>     if: always()
>     steps:
>       - uses: actions/download-artifact@v4
>         with: { path: all-blob-reports, pattern: blob-report-* }
>       - run: npx playwright merge-reports --reporter html ./all-blob-reports
> ```
>
> Two things worth knowing:
>
> **`fail-fast: false`** — otherwise one failing shard cancels the others and you only see part of the picture. You want all results.
>
> **The merge job** uses the `blob` reporter, then combines into one HTML report. Without it you get four separate partial reports, which is much harder to read.
>
> And the honest framing: sharding buys wall-clock time with money. Four runners for 10 minutes is roughly the compute of one runner for 40. Worth it for developer feedback speed, but it's a trade, not free.
>
> ---
>
> **Tiered triggers — often better than more machines:**
>
> ```yaml
> on:
>   pull_request:              # smoke only, fast
>   push:
>     branches: [main]         # full regression
>   schedule:
>     - cron: '0 2 * * *'      # nightly cross-browser
> ```
>
> ```yaml
> - run: npx playwright test --grep @smoke     # on PR
> ```
>
> A four-minute smoke suite on every PR plus full regression on merge is usually a better developer experience than forty minutes on everything — and it costs less than sharding."

---

## Workflow Essentials

| Element | Why |
|---------|-----|
| `--with-deps` | OS libraries, not just browsers |
| `if: always()` on artifacts | Failures are when you need them |
| `timeout-minutes` | Stops hung runs burning time |
| Browser cache keyed on lockfile | Speed, without version mismatch |
| `fail-fast: false` on matrix | See all shard results |
| `blob` reporter + merge | One report from many shards |
| Secrets via `env` | Never commit credentials |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Our setup uses GitHub Actions for deployment triggers with Azure DevOps orchestrating test execution on VMs, so I've worked across both. The GitHub Actions patterns are the same regardless.
>
> The thing I'd emphasize is artifacts, because it's what makes CI failures actionable. Uploading the HTML report and traces with `if: always()` means a red build is a five-minute trace replay rather than an hour of guessing. Leaving off `if: always()` is the most common mistake I see — the step gets skipped on failure, which is precisely when you needed it.
>
> On caching, keying the browser cache on the lockfile hash matters. Cache on a static key and a Playwright version bump gives you browsers that don't match the library, which produces failures that make no sense.
>
> And on speed, I'd reach for tiered triggers before sharding — smoke on PRs, full regression on merge, cross-browser nightly. That gets most of the feedback-speed benefit without paying for parallel runners."

---

## Common Interview Questions & Answers

### Q1: "Walk me through a Playwright GitHub Actions workflow."

**A:** "Checkout, Node setup with npm caching, `npm ci`, `npx playwright install --with-deps`, run tests, upload artifacts with `if: always()`. The `--with-deps` installs OS libraries the browsers need — skip it and you get cryptic launch failures on a clean runner. And `if: always()` matters because otherwise the artifact step is skipped exactly when tests fail."

### Q2: "How do you make CI failures debuggable?"

**A:** "Upload the HTML report and traces as artifacts, configured with `trace: 'on-first-retry'` and `video: 'retain-on-failure'`. Then a failure is a trace replay rather than a guessing exercise. Without artifacts you're reading a stack trace and theorizing, which is where hours disappear."

### Q3: "How does sharding work in GitHub Actions?"

**A:** "A matrix strategy with `--shard=N/total`, then a merge job that combines blob reports into one HTML report. Two gotchas: set `fail-fast: false` so one failing shard doesn't cancel the others, and use the blob reporter so the results can actually be merged. Worth being clear that sharding trades money for wall-clock time — four runners for ten minutes is the compute of one for forty."

### Q4: "How do you handle secrets?"

**A:** "GitHub Secrets, injected as environment variables at the step level. Never in the repo, never in the workflow file. Playwright config reads them via `process.env`. For anything longer-lived I'd prefer OIDC federation over static secrets where the platform supports it."

### Q5: "Your pipeline takes too long. What do you do?"

**A:** "Cache first — browser installs and npm are a meaningful chunk and caching is free. Then tiered triggers: smoke on PRs, full regression on merge, slow suites nightly. Only after that would I shard, because that's the option that costs money. And before any of it I'd check whether the tests themselves are wasting time on UI-driven login and setup, which is usually the bigger win."

---

## Weak Areas to Watch

❌ **Don't omit `--with-deps`**
→ Browsers fail to launch on clean runners.

❌ **Don't forget `if: always()`**
→ Artifacts get skipped exactly when needed.

❌ **Don't cache browsers on a static key**
→ Version mismatch produces baffling failures.

❌ **Don't shard before caching and tiering**
→ Those are free; sharding costs money.

❌ **Don't skip `fail-fast: false`**
→ You lose visibility into other shards.

---

## Key Takeaway for Interview

CI is what makes a suite operational. Artifacts are what make it maintainable.

The detail that shows real experience:

> **"`if: always()` on the artifact upload. Without it the step is skipped when tests fail — which is exactly when you need the report. It's the most common mistake in Playwright CI setups and it's self-defeating."**

And the cost framing:

> **"I'd exhaust caching and tiered triggers before sharding. Those are free; sharding buys wall-clock time with money."**

---

## Checklist

✅ `--with-deps` on browser install
✅ `if: always()` on artifact upload
✅ `timeout-minutes` set on the job
✅ Browser cache keyed on lockfile hash
✅ `fail-fast: false` on shard matrix
✅ Blob reporter + merge job for sharded runs
✅ Secrets via GitHub Secrets, never committed
✅ Tiered triggers before adding runners

---

**Remember:** A CI failure without artifacts is a guessing game. 🎯
