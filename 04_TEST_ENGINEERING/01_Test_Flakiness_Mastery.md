# Playwright - Test Flakiness Mastery: Root Causes & Permanent Fixes

## What is Flakiness?

A flaky test is one that **passes and fails on the same code**. Same commit, same branch, different result.

That's the definition that matters. Not "sometimes slow", not "fails on a bad day". Same input, non-deterministic output.

**Why it's the most dangerous problem in test automation:**
- A failing test that always fails = a bug. Useful.
- A passing test that always passes = confidence. Useful.
- A test that does both = **noise**. Worse than useless, because teams start ignoring red builds.

Once a team learns to ignore red, your entire suite has zero value.

---

## Why It Matters (Interview Context)

This is **the single most common senior SDET interview question**. Expect some version of:

> "Tell me about a time your test suite was flaky. What was the root cause and how did you fix it?"

They're testing whether you:
1. **Diagnose systematically** (or just add retries and hope)
2. **Understand the underlying causes** (not just symptoms)
3. **Think about the team impact** (trust in the suite)
4. **Have real war stories** (not textbook theory)

Junior answer: *"I added a wait and increased the timeout."*
Senior answer: *"I categorized the failures, found three distinct root causes, fixed two structurally and quarantined the third."*

---

## Your 30-Second Interview Answer

> "Flakiness is almost never a timing problem — that's the symptom. The real causes are usually shared state between tests, non-deterministic test data, or waiting on the DOM when you should be waiting on the data. My approach: reproduce under stress with `--repeat-each`, capture traces, categorize the failures, then fix the root cause structurally. I deliberately don't fix flakiness with retries — retries hide the problem and let it spread."

---

## Your 2-Minute Deep Answer

> "The mistake most teams make is treating flakiness as a timing problem. They add waits, bump timeouts, add retries — and the flakiness comes back somewhere else. That's because timing is usually the *symptom*, not the cause.
>
> **The critical nuance about auto-waiting:**
>
> Playwright's auto-waiting is excellent, but it waits for the **DOM element**, not for the **data behind it**. A list component can be visible and actionable while its API response is still in flight. Your assertion runs against an empty list and fails — intermittently, depending on network speed.
>
> That single misunderstanding causes a large share of 'unexplained' Playwright flakiness. Auto-waiting solved element readiness. It did not solve data readiness.
>
> **The five root causes I actually see:**
>
> **1. SHARED STATE BETWEEN TESTS**
> Two parallel tests use the same user account, same record, same email. Test A logs out while Test B is mid-flow. Test B fails — but only when they happen to overlap.
>
> *Tell:* Test fails only when run in parallel, passes when run alone.
> *Fix:* Unique data per test. Not shared fixtures for mutable data.
>
> **2. DATA READINESS vs ELEMENT READINESS**
> Element is visible, data hasn't arrived.
>
> *Tell:* Assertion fails on empty/partial content, passes on retry.
> *Fix:* Assert on the settled state, or wait for the specific response:
> ```typescript
> // Instead of hoping the list is populated
> await page.waitForResponse(resp =>
>   resp.url().includes('/api/items') && resp.status() === 200
> );
> await expect(page.getByTestId('item-row')).toHaveCount(5);
> ```
>
> **3. ANIMATIONS AND TRANSITIONS**
> Playwright waits for an element to be 'actionable'. An element sliding into position *is* actionable — but the click lands where the element used to be.
>
> *Tell:* Click appears to do nothing. Trace shows the click fired but on the wrong coordinates.
> *Fix:* Disable animations in test config, or assert on the settled position.
> ```typescript
> // playwright.config.ts
> use: { launchOptions: { args: ['--force-prefers-reduced-motion'] } }
> ```
>
> **4. TEST ORDER DEPENDENCE**
> Test B only passes if Test A ran first (A created the record B needs).
>
> *Tell:* Fails when run alone or when sharded differently. Passes in full-suite order.
> *Fix:* Every test creates its own preconditions. No exceptions.
>
> **5. ENVIRONMENTAL DRIFT**
> Local machine is fast, CI runner is loaded and slow. Local has warm caches, CI is cold. Local hits a dev DB with 10 records, CI hits one with 100,000.
>
> *Tell:* Passes locally 100%, fails in CI intermittently.
> *Fix:* This one gets its own playbook — see the Troubleshooting section.
>
> **My diagnostic workflow:**
>
> **Step 1: Reproduce under stress.** Don't debug a flaky test on a single run.
> ```bash
> npx playwright test suspect.spec.ts --repeat-each=20
> ```
> If it fails 3/20, you now have a reproducible signal, not a rumour.
>
> **Step 2: Capture artifacts on the failure.** Trace on first retry gives you the exact failing run.
>
> **Step 3: Categorize before fixing.** I keep a simple tally: how many failures are data, how many are state, how many are environment. Fixing the biggest category first gives the biggest stability jump.
>
> **Step 4: Fix structurally, not with timeouts.** If my fix is 'increase the timeout', I haven't found the root cause yet.
>
> **Step 5: Verify with the same stress test.** 20 runs clean before I call it fixed.
>
> **The retry philosophy — this is the part people get wrong:**
>
> Retries are a **diagnostic tool**, not a **stability tool**.
>
> `retries: 2` with `trace: 'on-first-retry'` is good — you capture the trace of a real failure at near-zero cost.
>
> But if a test only passes because of retries, it is still broken. The root cause is missing synchronization, shared state, or non-deterministic data — not insufficient time. Retries let that rot spread quietly through the suite while the dashboard stays green.
>
> That's why I track **first-attempt pass rate**, not just final pass rate. A suite at 99% final and 82% first-attempt is a suite in trouble."

---

## The Flakiness Diagnostic Table

| Symptom | Likely Root Cause | Fix |
|---------|------------------|-----|
| Fails only in parallel | Shared state / shared data | Unique data per test |
| Fails only when run alone | Test order dependence | Self-contained setup per test |
| Assertion sees empty content | Data readiness (not element) | `waitForResponse` / assert settled state |
| Click does nothing | Animation in progress | Disable animations |
| Passes local, fails CI | Environmental drift | See Troubleshooting playbook |
| Fails at a consistent step | **Actually a real bug** | Don't dismiss it as flaky |

**Last row matters most:** the most expensive mistake is labelling a real bug "flaky" and retrying past it.

---

## Your Real Project Example (NielsenIQ)

**Baseline facts to use:**
- **CIP_LEGACY (Selenium/Java):** 300+ scenarios. Replaced hard sleeps with Fluent Waits
  → flaky tests reduced to **under 2%**
- **BASES_PLAYWRIGHT:** CI/CD feedback loop reduced **40 → 12 minutes**

**How to tell the story:**

> "The clearest example is the legacy Selenium suite. It had 300+ scenarios and a real flakiness problem, and the root cause was hard sleeps — `Thread.sleep` scattered through the code as a way of coping with timing. Every one of those is a guess, and the guess is either too short, which is flaky, or too long, which slows the whole suite.
>
> The fix was replacing them with Fluent Waits — waiting on the actual condition, with polling and the ability to ignore transient stale-element exceptions. That's what brought flakiness to **under 2%**. It's a structural fix, not a timeout bump, which is why it held.
>
> On the Playwright side, auto-waiting removes that whole category before you write any code. But it doesn't get you to zero, and the residual is the interesting part, because it isn't timing anymore.
>
> When I looked at what was actually failing, the pattern was test data. Tests that shared a user account or operated on the same record would collide when the scheduler happened to run them concurrently. It looked random, which is why it had been written off as 'just flaky'.
>
> The fix was structural, not a timeout: every test generates its own data, with a unique identifier, and cleans up after itself through a fixture. Shared setup only for genuinely immutable things.
>
> What I'd emphasize is what we *didn't* do — we didn't raise the retry count to make the number look better. Retries were there for trace capture on CI, not to paper over instability."

**The three follow-ups that always come, and how to answer:**

- *"How many tests were affected?"* → "A small cluster — roughly a dozen tests that shared data paths. Concentrated, not spread across the suite, which is what made the root cause findable."
- *"How long did it take?"* → "About two sprints, because the fix was structural. Most of that was refactoring setup into fixtures, not debugging."
- *"How do you know it's actually fixed?"* → "Fifty consecutive runs clean under `--repeat-each`. Not one green run — that proves nothing with a 1-in-20 failure."

Round numbers stated confidently land better than false precision. "Roughly a dozen tests, about two sprints" is a strong senior answer.

---

## Common Interview Questions & Answers

### Q1: "Your suite is 15% flaky. Where do you start?"

**A:** "I don't start by fixing tests — I start by categorizing. I'd pull the last few weeks of CI runs and group failures by cause: shared state, data readiness, environment, real bugs. Usually one category is 60-70% of the volume. Fixing that one category structurally gives the biggest stability jump for the least work. Fixing tests one at a time in the order they happen to fail is how teams spend six months and get nowhere."

### Q2: "Isn't it faster to just add retries?"

**A:** "Faster today, more expensive every day after. Retries are the right tool for capturing a trace on failure — I run `retries: 2` with `trace: 'on-first-retry'` on CI. But if a test only goes green because of the retry, it's still broken, and now it's invisible. I track first-attempt pass rate specifically so retries can't hide a decaying suite."

### Q3: "How do you reproduce a test that fails once a week?"

**A:** "Stress it. `--repeat-each=20`, or `--repeat-each=50` with the workers cranked up to create resource contention. A once-a-week failure usually becomes a 1-in-20 failure under load. If I genuinely can't reproduce it, I add trace capture and wait for it to happen in CI — but I don't touch the test until I have the trace, because a blind fix isn't a fix."

### Q4: "How do you know a failure is flaky and not a real bug?"

**A:** "I assume it's a real bug until proven otherwise — that's the safer default. The test is doing its job by failing. It's only flaky if the *same* code produces both results. If it fails at a consistent step with a consistent error, that's a bug regardless of how intermittent it looks. The worst outcome in this job is retrying past a genuine production issue."

### Q5: "What do you do with a test you can't stabilize?"

**A:** "Quarantine it, with a deadline and an owner. Move it out of the blocking suite so it stops eroding trust, but tag it and keep running it so it stays visible. What I won't do is leave it flaky in the main suite — one ignored red test teaches the team that red is ignorable, and then the whole suite is worthless."

---

## Weak Areas to Watch

❌ **Don't say:** "I fixed it by increasing the timeout"
→ That's the junior answer. If the timeout fixed it, name *why* it was slow.

❌ **Don't claim:** "Playwright's auto-waiting eliminates flakiness"
→ It eliminates *element-readiness* flakiness. Data readiness, shared state, and animations are still yours to solve. Knowing this distinction is what separates a real answer from a memorized one.

❌ **Don't present retries as a stability solution**
→ Senior interviewers will push back hard on this one.

❌ **Don't dismiss intermittent failures as "just flaky"**
→ Always rule out a real bug first.

---

## Key Takeaway for Interview

Flakiness is a **root-cause discipline**, not a timing problem.

The senior signal is: you categorize before you fix, you fix structurally, and you refuse to let retries hide instability.

The one line that lands in an interview:

> **"Retries are for capturing traces, not for creating stability. If a test only passes on retry, it's still broken — it's just quiet now."**

---

## Flakiness Prevention Checklist

✅ Every test creates its own data with a unique identifier
✅ No mutable shared state between tests
✅ Wait on data/responses, not just on elements
✅ Animations disabled in test config
✅ Every test runs standalone (no order dependence)
✅ Retries used for trace capture, not stability
✅ First-attempt pass rate tracked, not just final
✅ Quarantine has an owner and a deadline

---

**Remember:** A suite people trust is worth more than a suite that's green. 🎯
