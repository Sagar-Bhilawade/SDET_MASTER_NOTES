# CI/CD - Docker & Environment Consistency

## What This Covers

Running Playwright in containers, and managing configuration across local, staging, and production environments.

The problem both solve is the same: **making "works on my machine" stop being a meaningful sentence.**

---

## Why It Matters (Interview Context)

This connects directly to the most common troubleshooting question — "passes locally, fails in CI." Containerization is the structural fix for a whole category of that.

Interviewers are checking whether you understand that environment differences are a *design* problem you can eliminate, not just a debugging problem you cope with.

---

## Your 30-Second Interview Answer

> "The official Playwright Docker image gives you identical browsers, fonts, and OS libraries everywhere — which removes a whole category of local-versus-CI differences rather than debugging them one at a time. The critical rule is matching the image tag to your Playwright package version; a mismatch produces failures that make no sense. For environments, everything comes from config and environment variables — one suite runs against local, staging, or production by changing `baseURL`, never by changing test code."

---

## Your 2-Minute Deep Answer

> "Two related problems: environment consistency, and environment configuration.
>
> ---
>
> **DOCKER — consistency**
>
> Microsoft publishes official images with browsers and all OS dependencies preinstalled:
>
> ```dockerfile
> FROM mcr.microsoft.com/playwright:v1.49.0-noble
>
> WORKDIR /app
> COPY package*.json ./
> RUN npm ci
> COPY . .
> CMD ["npx", "playwright", "test"]
> ```
>
> **The version-matching rule is the thing to get right.** The image tag must match your `@playwright/test` version. If your package is 1.49.0, the image must be `v1.49.0`. A mismatch means the library and browser binaries disagree, and the resulting failures are genuinely baffling — selectors behaving oddly, launches failing, nothing that points at the real cause.
>
> I'd enforce it rather than remember it: derive the tag from `package.json` in the build, or have a CI check that they match. It's exactly the kind of thing that should be automated rather than relied on.
>
> **Running locally:**
> ```bash
> docker run --rm -v $(pwd):/app -w /app \
>   mcr.microsoft.com/playwright:v1.49.0-noble \
>   npx playwright test
> ```
>
> **What this actually solves:**
>
> - **Font rendering.** The big one for visual/screenshot tests. Different OSes render text differently, so a screenshot baseline captured on Windows will never match a Linux CI runner. Containerizing makes baselines portable.
> - **Missing OS libraries.** The `--with-deps` problem, solved permanently.
> - **Browser version drift.** Everyone runs identical binaries.
> - **Reproducing CI locally.** You can run the exact CI environment on your machine, which turns "I can't reproduce it" into a solvable problem.
>
> **What it doesn't solve:** data differences, timing under load, and network conditions. Those are still yours. Docker removes the environmental variables, not the environmental *reality* of a loaded shared runner.
>
> **A practical note:** containers default to a small `/dev/shm`, which can crash Chromium on heavy pages. Either `--shm-size=1gb` or `--ipc=host` when running. That's a real failure people hit and struggle to diagnose because the error is just a browser crash.
>
> ---
>
> **ENVIRONMENT CONFIGURATION**
>
> The principle: one suite, many environments, zero code changes.
>
> ```typescript
> // playwright.config.ts
> export default defineConfig({
>   use: {
>     baseURL: process.env.BASE_URL ?? 'http://localhost:3000',
>   },
> });
> ```
>
> ```bash
> BASE_URL=https://staging.example.com npx playwright test
> ```
>
> Tests reference relative paths — `page.goto('/login')` — so they're environment-agnostic by construction.
>
> **Fail fast on missing config:**
>
> ```typescript
> function required(name: string): string {
>   const value = process.env[name];
>   if (!value) throw new Error(`Missing required env var: ${name}`);
>   return value;
> }
>
> const apiToken = required('API_TOKEN');
> ```
>
> This matters more than it looks. Without it, a missing token means requests go out unauthenticated, you get 401s, and you spend twenty minutes investigating an auth bug that's actually a config problem. Failing at startup with a clear message costs nothing and saves that entirely.
>
> **Environment-specific behaviour:**
>
> ```typescript
> const isProd = process.env.ENVIRONMENT === 'production';
>
> export default defineConfig({
>   retries: process.env.CI ? 2 : 0,
>   workers: process.env.CI ? 4 : undefined,
>   grep: isProd ? /@smoke/ : undefined,     // only smoke against prod
> });
> ```
>
> That last line reflects a real constraint — you generally don't run a full destructive regression suite against production. Read-only smoke tests only.
>
> **Secrets never live in `.env` files that get committed.** `.env` for local development, gitignored; CI secrets from the platform's secret store. And `.env.example` checked in, documenting what's needed without the values."

---

## What Docker Fixes

| Problem | Fixed? |
|---------|:---:|
| Font rendering differences | ✅ |
| Missing OS libraries | ✅ |
| Browser version drift | ✅ |
| Reproducing CI locally | ✅ |
| Data volume differences | ❌ |
| Timing under runner load | ❌ |
| Network conditions | ❌ |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "We run on Azure DevOps VMs, and the environment consistency principle applies regardless of whether you're containerizing — everything comes from config, nothing is hardcoded. `baseURL` from an environment variable, credentials from the secret store, tests using relative paths. The same suite runs against any environment by changing variables, never by touching test code.
>
> Where containerizing pays off most is visual and screenshot testing. Font rendering differs across operating systems, so a baseline captured locally on Windows will never match a Linux runner. Running both in the same image makes baselines portable — that's the difference between visual testing being viable and being permanently broken.
>
> The rule I'd emphasize is matching the image tag to the Playwright package version. A mismatch means the library and browser binaries disagree, and the failures don't point at the cause at all. I'd enforce that with a CI check rather than trusting people to remember, which is the same principle as any repeated mistake — if a machine can catch it, it shouldn't be a human's job.
>
> And I'd validate required environment variables at startup. A missing API token otherwise surfaces as 401s that look like an auth bug and cost twenty minutes to trace back to config."

---

## Common Interview Questions & Answers

### Q1: "Why run Playwright in Docker?"

**A:** "Consistency. Identical browsers, fonts, and OS libraries everywhere, which eliminates a whole category of local-versus-CI failures structurally instead of debugging them individually. It also means I can reproduce the CI environment locally, which turns 'can't reproduce it' into a solvable problem. The big win specifically is font rendering for visual tests — screenshot baselines aren't portable across OSes otherwise."

### Q2: "What's the most common Docker mistake with Playwright?"

**A:** "Version mismatch between the image tag and the `@playwright/test` package. The library and browser binaries disagree and you get failures that don't point at the cause — odd selector behaviour, launch failures. I'd enforce the match with a CI check rather than relying on people remembering."

### Q3: "Does Docker fix 'passes locally, fails in CI'?"

**A:** "Part of it. It fixes the environmental differences — fonts, libraries, browser versions. It doesn't fix data volume differences, timing under a loaded runner, or network conditions, and those are usually the bigger causes. So it removes variables from the investigation rather than solving the whole class."

### Q4: "How do you manage config across environments?"

**A:** "Everything from environment variables, nothing hardcoded. `baseURL` in config, tests using relative paths, so the same suite runs anywhere by changing variables. I'd also validate required variables at startup and fail with a clear message — otherwise a missing token surfaces as 401s that look like an auth bug rather than a config problem."

### Q5: "Would you run tests against production?"

**A:** "Read-only smoke tests, yes — tagged and filtered so nothing destructive runs. A full regression suite against production risks creating or mutating real data. I'd gate that with a `grep` filter in config keyed off the environment, so it's structurally impossible rather than a convention people have to remember."

---

## Weak Areas to Watch

❌ **Don't mismatch image tag and package version**
→ Produces failures that don't point at the cause.

❌ **Don't claim Docker solves all CI differences**
→ Data and timing remain. Knowing the boundary is the depth signal.

❌ **Don't hardcode URLs or credentials**
→ Config and env vars, always.

❌ **Don't let missing config fail silently**
→ Validate at startup with a clear message.

❌ **Don't forget `--shm-size`**
→ Chromium crashes on heavy pages in default containers.

---

## Key Takeaway for Interview

Containerization is the **structural** fix for environment drift — it removes variables rather than debugging them.

The rule that matters:

> **"The image tag must match the Playwright package version. A mismatch means the library and browsers disagree, and the failures point nowhere useful. Enforce it with a CI check, don't rely on remembering."**

And the honest boundary:

> **"Docker fixes fonts, libraries, and browser versions. It doesn't fix data volume or timing under load — and those are usually the bigger causes of CI-only failures."**

---

## Checklist

✅ Official Playwright image, tag matched to package version
✅ Version match enforced by a CI check
✅ `--shm-size=1gb` or `--ipc=host` when running containers
✅ `baseURL` from environment variable
✅ Tests use relative paths only
✅ Required env vars validated at startup
✅ `.env` gitignored, `.env.example` committed
✅ Production runs restricted to read-only smoke via config

---

**Remember:** Eliminating a variable beats debugging it every time. 🎯
