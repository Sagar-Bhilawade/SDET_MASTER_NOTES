# Scenario Round Cheat Sheet — Problem-Solving

> Dense recall for `04_TEST_ENGINEERING`. The "walk me through how you'd debug/fix this"
> round. These are frameworks, not facts — have the *steps* ready, not memorized answers.

---

## Flakiness — the 5 root causes (know them cold)

1. **Timing/race** — acting before the app is ready (the biggest one).
2. **Shared/colliding test data** — tests stepping on each other under parallelism.
3. **Order dependence** — test B needs test A to have run.
4. **Environment** — CI differs from local (resources, timing, headless).
5. **External dependencies** — real APIs/third parties being slow or down.

**Why retries aren't the fix:** a retry hides the flake, it doesn't remove the cause. The suite still lies — you just stop seeing it. Retries are a safety net for genuinely nondeterministic externals, not a substitute for fixing races and shared state.

**The discipline:** never fix blind. **Reproduce first** (`--repeat-each`, high worker count) until you have a reliable failure rate — otherwise you can't tell if your fix worked or the flake just didn't show.

---

## "Passes locally, fails in CI" — the debug walk

1. **Reproduce** — run with `CI=true` locally; often reproduces immediately.
2. **Compare environments** — headless vs headed, viewport size, timezone/locale, resources.
3. **Pull the trace/video** from the CI failure — diff a passing vs failing run.
4. **Common culprits:** slower CI timing exposes races; different viewport hides elements; missing env var; test-data collision under CI's parallelism.
5. **Fix the cause**, not the symptom. Verify by reproducing green under the same stress.

---

## Slow suite — the optimization ladder

1. **Parallelize** (workers/sharding) — biggest lever, needs isolation first.
2. **Push down the pyramid** — move UI tests to API level where possible (200ms vs 10s).
3. **Kill wasted waits** — hard sleeps → web-first/explicit waits.
4. **Reuse setup** — storage-state auth instead of UI login every test.
5. **Cut redundant coverage** — 3 tests asserting the same thing → 1.

*Your real number: 40→12 min, mostly sharding + removing wasted waits.*

---

## "Tests break on every UI change" — selector breakage

**Root cause:** selectors tied to structure/styling (CSS classes, nth-child, deep XPath).
**Fix:** role/label/test-id locators that survive refactors. **The influence angle:** get devs to add `data-testid` as they build features — framed as shared pipeline health, not a QA favour. See `06/03` behavioral story.

---

## Test data management

- **Isolation:** every test creates its own data (fixture/factory), cleans up in teardown. No shared mutable state.
- **Seeding:** via API, not UI — faster and stabler.
- **Cleanup:** teardown even on failure; or ephemeral data (unique per run) so leftovers don't collide.
- **The collision bug:** two tests using the same record fail intermittently when scheduling overlaps them. Structural fix = own data per test.

---

## Metrics — which you track and why

| Metric | Why it matters |
|--------|---------------|
| **First-attempt pass rate** | The honest flakiness number — before retries hide it. |
| **Escape rate** | Bugs that reached production — did testing actually catch things? |
| **CI feedback time** | How fast devs learn they broke something (your 40→12). |
| **Flaky rate** | Tests that fail without a code change (yours: under 2%). |
| **Coverage** | Only useful as a risk conversation, not a target number. |

*Vanity metric to avoid: total test count. 40 flaky tests are worse than 10 trustworthy ones.*

---

## Real-world erosion patterns (silent test debt)

Tests that no longer assert anything meaningful, `.skip` accumulating, retry counts creeping up, "known flaky" tests everyone ignores, coverage numbers gaming. The damage: **the team stops trusting the suite** — and an untrusted suite is worse than none, because red stops meaning "stop."

---

## The meta-framework for any scenario question

1. **Clarify** the symptom precisely.
2. **Reproduce reliably** before touching anything.
3. **Isolate** the variable (diff passing vs failing).
4. **Fix the root cause**, name what you deliberately *didn't* do (e.g. "didn't just add retries").
5. **Verify** by reproducing green under the same conditions.
6. **Prevent** recurrence (a check, a convention, a lint rule).

---

## Key takeaway

> **These rounds test process, not trivia — so lead with "first I'd reproduce reliably,
> because you can't verify a fix you can't reproduce."** The senior signal throughout is
> refusing to fix blind and refusing to hide flakes behind retries. Every answer is a
> walk: reproduce → isolate → fix the cause → verify → prevent.
