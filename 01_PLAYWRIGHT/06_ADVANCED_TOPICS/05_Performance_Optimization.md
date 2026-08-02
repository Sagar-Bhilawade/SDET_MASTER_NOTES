# Playwright - Performance Optimization: Making a Large Suite Fast

## What This Topic Covers

How to take a suite that takes too long and make it fast — and how to know which lever to pull, because pulling the wrong one wastes weeks.

The classic interview scenario:

> "You have 1000 tests taking 2 hours. How do you get that down?"

Most candidates immediately say "add more workers." That's sometimes right and often the least effective answer available.

---

## Why It Matters (Interview Context)

This question tests engineering judgement more than Playwright knowledge. They want to see whether you:
1. **Measure before optimizing** (or guess)
2. **Understand where time actually goes**
3. **Know the difference between parallelizing waste and removing waste**
4. **Think about cost**, not just speed

The senior insight: **parallelization makes slow tests finish sooner. It doesn't make them less slow.** If each test wastes 20 seconds on UI-driven setup, throwing 8 workers at it means you're now wasting 20 seconds eight times concurrently — and paying for eight machines to do it.

---

## Your 30-Second Interview Answer

> "First I'd measure where the time actually goes, because the intuition is usually wrong. Typically the biggest wins aren't parallelization — they're removing waste: seeding data through the API instead of the UI, reusing authentication via storage state instead of logging in every test, and moving validation that doesn't need a browser down to the API layer. Once the tests themselves are lean, then I parallelize and shard. Doing it in that order matters, because parallelizing waste just means paying more machines to do the same wasted work."

---

## Your 2-Minute Deep Answer

> "I work through this in a fixed order, because the order determines how much you get for the effort.
>
> ---
>
> **STEP 1: MEASURE. Don't guess.**
>
> Playwright's reporters give per-test duration. First thing I want is the distribution, not the total.
>
> Suite time is almost never uniform. It's usually a small number of tests owning a large share. If 15 tests out of 400 account for 40% of the runtime, that's where the work is — and you'd never find that by intuition.
>
> ---
>
> **STEP 2: REMOVE WASTE (biggest wins, almost always)**
>
> **Authentication.** If every test logs in through the UI, you're paying that cost hundreds of times for zero test value. Storage state moves it to once per run.
>
> ```
> 300 tests × ~3s login  = 15 minutes of pure waste
> Storage state          = one login, then milliseconds to load
> ```
>
> This is usually the single largest win in a suite that hasn't done it.
>
> **Data setup through the UI.** Same logic. Clicking through three screens to reach the state you want to test costs 15-20 seconds per test. The same setup via API is under a second.
>
> **Tests at the wrong level.** A meaningful share of UI tests are validating logic that has no UI dependency — field validation rules, calculation correctness, permission logic. Those run in ~200ms as API tests instead of ~15s as browser tests, and they're more stable.
>
> This one is strategic rather than technical, which is why it's worth raising unprompted. You're not making the test faster, you're removing it from the browser suite entirely.
>
> **Hard waits.** `waitForTimeout` is dead time by definition. Every one of them is a guess, and the guess is always too long or too short.
>
> ---
>
> **STEP 3: PARALLELIZE (multiply what's left)**
>
> Now that tests are lean, workers actually pay off.
>
> ```typescript
> workers: process.env.CI ? 4 : undefined   // undefined = half the CPU cores
> ```
>
> The ceiling here is resource contention. Past a certain point, workers compete for CPU and memory and each test gets slower — you add workers and total time gets *worse*. That inflection point is machine-specific, so it's an empirical question: test 2, 4, 8 and look at the curve.
>
> ---
>
> **STEP 4: SHARD (scale beyond one machine)**
>
> When one machine is saturated, split across machines.
>
> ```bash
> npx playwright test --shard=1/4    # on machine 1
> npx playwright test --shard=2/4    # on machine 2, concurrently
> ```
>
> Playwright merges the reports afterward.
>
> The economics matter here and it's worth saying out loud: sharding buys wall-clock time with money. Four machines for 10 minutes costs roughly the same compute as one machine for 40. What you're buying is developer feedback speed, and that's usually a good trade — but it *is* a trade, and framing it that way shows you understand the cost side.
>
> ---
>
> **STEP 5: RUN LESS**
>
> The lever people forget entirely.
>
> Not every test needs to run on every commit:
> - Smoke suite on every push — fast, blocking
> - Full regression on merge to main
> - Slow cross-browser and visual suites nightly
>
> Or scope by change: touched the payments module, run payments plus smoke.
>
> The fastest test is the one that correctly didn't need to run.
>
> ---
>
> **The order is the point.** Someone who jumps straight to sharding a bloated suite spends real money to run waste in parallel. Someone who removes the waste first often finds they don't need the extra machines at all."

---

## The Optimization Ladder

| Step | Lever | Typical Impact | Cost |
|------|-------|---------------|------|
| 1 | **Measure** | — | Free |
| 2 | **Storage state auth** | Very high | Free |
| 3 | **API data seeding** | Very high | Dev collaboration |
| 4 | **Move tests to API layer** | High | Strategy change |
| 5 | **Remove hard waits** | Medium | Free |
| 6 | **Add workers** | High until contention | Free |
| 7 | **Shard across machines** | Linear | 💰 Machines |
| 8 | **Run fewer tests per trigger** | High | Risk judgement |

**Read it top down.** Everything above sharding is free.

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Our suite runs 300+ tests in 8-10 minutes on Azure DevOps VMs — the legacy Selenium equivalent was 25-30 minutes for comparable coverage.
>
> The gap isn't mostly Playwright being faster per action, though it is. It's the architectural choices around the tests. Authentication happens once through storage state rather than per test, which at 300 tests is the difference between one login and three hundred. Setup goes through the API rather than the UI. And parallel execution across workers multiplies whatever's left.
>
> The way I'd frame the general principle: parallelization multiplies your per-test time, so it's worth getting per-test time down first. If each test carries 20 seconds of avoidable setup, adding workers means you're paying for more machines to run the same waste concurrently. Remove the waste, then multiply."

---

## Common Interview Questions & Answers

### Q1: "1000 tests, 2 hours. How do you fix it?"

**A:** "I'd start by measuring the distribution, because it's rarely uniform — usually a small subset owns most of the time. Then I'd look for structural waste before touching infrastructure: is every test logging in through the UI, is setup going through the browser, are there tests here that don't need a browser at all. Those are free wins and they're usually large. Only after that would I add workers and shard, because sharding a bloated suite means paying for machines to run waste in parallel."

### Q2: "Why not just add more workers?"

**A:** "It helps, up to the point of resource contention — past that, workers compete for CPU and each test gets slower, so total time can actually increase. But the bigger issue is that parallelization multiplies per-test time rather than reducing it. If each test wastes 20 seconds on setup, 8 workers means 8 concurrent wastes. Fix the per-test cost first and the same worker count goes further."

### Q3: "How many workers is right?"

**A:** "It's empirical — depends on the machine and how heavy the tests are. I'd test 2, 4, and 8 and plot the curve. It improves and then flattens or reverses. Locally I'd let Playwright default to half the cores; on CI I'd tune to the runner. There's no universal number, and a candidate who gives you one hasn't measured."

### Q4: "When does sharding make sense?"

**A:** "When one machine is saturated and wall-clock feedback time is the binding constraint. It's worth being explicit that sharding trades money for time — four machines for 10 minutes is roughly the compute cost of one machine for 40. That's usually a good trade because developer waiting time is expensive, but it's a business decision, not a purely technical one."

### Q5: "Is there a downside to making tests too fast?"

**A:** "The risk is optimizing by cutting coverage or weakening assertions — a suite gets faster and nobody notices it stopped checking things. That's why I'd watch escape rate alongside execution time. Fast and blind is worse than slow and useful."

---

## Weak Areas to Watch

❌ **Don't jump to "add workers"**
→ It's step 6, not step 1.

❌ **Don't optimize without measuring**
→ Time distribution is almost never what you'd guess.

❌ **Don't ignore cost**
→ Sharding buys speed with money. Say so — it shows commercial awareness.

❌ **Don't forget "run less"**
→ Tiered suites are a legitimate and underused lever.

❌ **Don't trade coverage for speed silently**
→ Watch escape rate while you optimize.

---

## Key Takeaway for Interview

Optimization is an **ordered discipline**. Measure, remove waste, then multiply.

The line that separates a senior answer:

> **"Parallelization multiplies per-test time — it doesn't reduce it. Adding workers to a suite full of UI-driven setup just means paying for more machines to run the same waste concurrently."**

And the commercial one:

> **"Sharding buys wall-clock time with money. Four machines for 10 minutes is the compute cost of one for 40 — usually worth it for feedback speed, but it's a trade worth naming."**

---

## Optimization Checklist

✅ Per-test durations measured before any change
✅ Authentication via storage state, once per run
✅ Data seeded via API, never through the UI
✅ Non-browser validation pushed to the API layer
✅ Zero `waitForTimeout` calls
✅ Worker count tuned empirically, not guessed
✅ Sharding only after single-machine limits are hit
✅ Tiered execution: smoke on push, full on merge, slow suites nightly
✅ Escape rate monitored so speed doesn't eat coverage

---

**Remember:** The fastest test is the one that correctly didn't need to run. 🚀
