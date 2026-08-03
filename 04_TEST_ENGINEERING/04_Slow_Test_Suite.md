# Troubleshooting - "The Test Suite Is Too Slow"

## The Problem

Feedback takes so long that developers stop waiting for it. That's the real failure — not the minutes themselves, but the moment the suite stops influencing behaviour because nobody watches it finish.

The theory and the full optimization ladder are in the previous note, `03_Performance_Optimization.md`. This one is the **diagnostic** — how to find where the time went.

---

## Why It Matters (Interview Context)

The scenario comes up constantly:

> "Your suite went from 10 minutes to 45 minutes over six months. What do you do?"

They're checking whether you **investigate** or **guess**. Adding workers is the guess. Finding out where the 35 minutes went is the investigation.

---

## Your 30-Second Interview Answer

> "I'd profile before changing anything, because time in a test suite is almost never evenly distributed — usually a small subset of tests owns most of the runtime, and you can't find that by intuition. Then I'd separate the two questions: is it slow because individual tests are slow, or because it isn't parallelizing well? Those have completely different fixes, and treating one as the other is how teams waste a quarter."

---

## Your 2-Minute Deep Answer

> "The first thing I want is the shape of the problem, not the total.
>
> ---
>
> **DIAGNOSTIC 1: WHERE IS THE TIME?**
>
> Playwright's JSON reporter gives per-test durations. What I'm looking for is the distribution.
>
> Two very different pictures:
>
> **Concentrated** — 20 tests out of 400 account for half the runtime. Great news: a small, targeted fix gets most of the win.
>
> **Flat** — every test takes about the same and it's all slow. That points at something systemic, in shared setup: login per test, heavy fixtures, slow global config.
>
> The fix for those two situations has almost nothing in common, so identifying which one you have is the entire first move.
>
> ---
>
> **DIAGNOSTIC 2: IS IT THE TESTS OR THE PARALLELISM?**
>
> Cheap check:
>
> ```
> Sum of individual test durations:  40 minutes
> Actual wall-clock with 4 workers:  38 minutes
> ```
>
> With 4 workers you'd expect roughly 10-12 minutes. Getting 38 means parallelism isn't working — tests are serializing on something. Usually a shared resource: one database, one test account, a rate-limited API.
>
> Whereas:
>
> ```
> Sum of individual durations:  40 minutes
> Wall-clock with 4 workers:    11 minutes
> ```
>
> That's parallelizing correctly. The suite is slow because the *tests* are slow, and no amount of infrastructure changes that.
>
> This single comparison tells you which half of the problem you have, and it takes two minutes to compute.
>
> ---
>
> **DIAGNOSTIC 3: WHAT'S INSIDE THE SLOW TESTS?**
>
> Open a trace for one of the worst offenders and look at the timeline. The time is almost always in one of four places:
>
> **Login.** If every test authenticates through the UI, that's 3-5 seconds each, hundreds of times, for zero test value. Storage state moves it to once per run. This is usually the largest single win available.
>
> **UI-driven setup.** Clicking through screens to build preconditions. Fifteen seconds of setup for a two-second assertion. Same work via API is under a second — and you get better failure attribution, because now a failure means the feature under test broke, not some unrelated upstream screen.
>
> **Hard waits.** `waitForTimeout` is pure dead time. Every one is a guess that's either too long or too short.
>
> **Genuinely slow application operations.** Report generation, large uploads, batch jobs. These are real and you can't optimize them away from the test side — but you can decide they don't belong in the fast feedback loop.
>
> ---
>
> **DIAGNOSTIC 4: DOES ALL OF IT NEED TO RUN?**
>
> The lever teams forget. Not every test needs to run on every commit.
>
> - Smoke on every push — fast, blocking
> - Full regression on merge to main
> - Cross-browser and visual suites nightly
>
> A 45-minute suite that runs nightly plus a 4-minute smoke suite on every push is a dramatically better developer experience than 45 minutes on every push, with essentially the same coverage over a day.
>
> ---
>
> **What I'd avoid:** jumping to more workers or sharding before profiling. Both cost money and neither reduces per-test time. If each test carries 20 seconds of avoidable login and setup, sharding across four machines means you're paying four machines to do the same wasted work concurrently."

---

## The Diagnostic Flow

```
Suite is slow
   │
   ├─ Profile per-test durations
   │
   ├─ Is time CONCENTRATED or FLAT?
   │     Concentrated → fix the top 10-20 tests
   │     Flat         → look at shared setup (login, fixtures)
   │
   ├─ Compare: sum of durations ÷ workers  vs  actual wall-clock
   │     Close to expected → tests are slow, fix the tests
   │     Far above        → parallelism is blocked, find the shared resource
   │
   ├─ Open a trace on the worst test
   │     Login? → storage state
   │     UI setup? → API seeding
   │     waitForTimeout? → delete it
   │     Genuinely slow app operation? → move it out of the fast loop
   │
   └─ Does it all need to run every time?
         → Tier it: smoke / regression / nightly
```

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "On the Playwright suite, we took the CI/CD feedback loop from **40 minutes to 12** on Azure DevOps.
>
> The thing I'd point out is that the gap isn't mainly Playwright being faster per action — it's the architecture around the tests. Authentication happens once through storage state rather than per test, which at 300 tests is the difference between one login and three hundred. Setup goes through the API rather than clicking through the UI. Parallel workers multiply whatever's left.
>
> If that suite started creeping up, my first move wouldn't be more workers — it'd be the distribution. Whether the growth is concentrated in a few tests or spread evenly across all of them tells you completely different things. Concentrated means someone added a handful of heavy tests. Flat means shared setup got more expensive, and that's usually a fixture that quietly started doing more work than it needs to."

---

## Common Interview Questions & Answers

### Q1: "Suite went from 10 to 45 minutes. Where do you start?"

**A:** "Profile it before changing anything. I want the distribution of per-test durations, because concentrated and flat growth mean different things — concentrated says someone added heavy tests, flat says shared setup got more expensive. Then I'd compare the sum of test durations divided by worker count against actual wall-clock time, which tells me in two minutes whether the problem is slow tests or blocked parallelism."

### Q2: "How do you know if parallelism is working?"

**A:** "Sum the individual test durations and divide by the worker count — that's roughly what wall-clock should be. If actual wall-clock is much higher, tests are serializing on something shared: a single database, one test account, a rate-limited endpoint. That's a completely different fix from making tests faster, and it's worth knowing which one you're solving."

### Q3: "What's usually the biggest single win?"

**A:** "Authentication, in a suite that hasn't addressed it. If every test logs in through the UI at 3-5 seconds each, moving to storage state removes essentially all of that. Data setup via API rather than UI is usually second. Both are free — no infrastructure spend — which is why I'd do them before considering more machines."

### Q4: "Would you just add more machines?"

**A:** "Only after the free wins. Sharding buys wall-clock time with money, and it multiplies per-test time rather than reducing it. If each test wastes 20 seconds on avoidable setup, four machines means four concurrent wastes. Remove the waste first — teams often find they no longer need the extra capacity."

### Q5: "Is there a risk in making a suite faster?"

**A:** "Optimizing by quietly reducing what's checked — cutting tests, weakening assertions. The suite gets faster and nobody notices it stopped catching things. That's why I'd watch escape rate alongside execution time. Fast and blind is worse than slow and useful."

---

## Weak Areas to Watch

❌ **Don't optimize before profiling**
→ The distribution is never what you'd guess.

❌ **Don't jump to workers or sharding**
→ Neither reduces per-test time, and both cost money.

❌ **Don't ignore the parallelism check**
→ Two minutes of arithmetic tells you which problem you have.

❌ **Don't forget tiering**
→ Running less is a legitimate lever, not a cop-out.

---

## Key Takeaway for Interview

Slowness is a **diagnosis before it's a fix**. The distribution tells you what kind of problem you have.

The check that shows real rigour:

> **"Sum the test durations, divide by workers, compare to wall-clock. If they're far apart, parallelism is blocked on a shared resource — that's a completely different problem from slow tests, and treating one as the other wastes a quarter."**

And the framing that shows commercial judgement:

> **"Everything above sharding on the optimization ladder is free. I'd exhaust the free wins before I spend money to run waste in parallel."**

---

## Diagnostic Checklist

✅ Per-test durations profiled first
✅ Distribution identified: concentrated vs flat
✅ Parallelism check: sum ÷ workers vs wall-clock
✅ Trace opened on the worst offenders
✅ Login moved to storage state
✅ Setup moved to API
✅ `waitForTimeout` calls eliminated
✅ Tiered execution considered before buying machines
✅ Escape rate watched so speed doesn't eat coverage

---

**Remember:** A suite nobody waits for has already failed, regardless of what it covers. 🎯
