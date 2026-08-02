# Troubleshooting - Debugging a Flaky Test (Step by Step)

## The Problem

A test that passes and fails on identical code. This note is the **hands-on workflow** — the concepts and root-cause theory are in `01_PLAYWRIGHT/06_ADVANCED_TOPICS/01_Test_Flakiness_Mastery.md`. This one is what you actually do, in order.

---

## Why It Matters (Interview Context)

Interviewers often make this a live exercise:

> "Here's a flaky test. Talk me through debugging it."

They want to hear a **repeatable process**, not intuition. The tell for a strong candidate is that they refuse to change code before they can reproduce the failure.

---

## Your 30-Second Interview Answer

> "The rule I hold to is: don't touch the test until you can reproduce it. I stress it with `--repeat-each` and high worker count until I have a reliable failure rate, capture the trace from a failing run, then compare a passing trace against a failing one. The diff between those two is almost always the answer. Then I fix the cause and verify with the same stress run — a single green run proves nothing when you're chasing a 1-in-20."

---

## Your 2-Minute Deep Answer

> "Six steps, and the discipline is not skipping ahead.
>
> ---
>
> **STEP 1: REPRODUCE IT. Reliably.**
>
> ```bash
> npx playwright test suspect.spec.ts --repeat-each=20 --workers=4
> ```
>
> You want a rate, not an anecdote. "Fails 3 out of 20" is something you can verify a fix against. "It failed once last Tuesday" is not.
>
> If it won't reproduce, turn up the pressure — more workers to create contention, `--repeat-each=50`, run it alongside the full suite rather than alone. Contention is often the missing ingredient.
>
> **Do not modify the test at this stage.** Changing code before you have a reproduction means you'll never know whether your fix worked or the flake just didn't show up.
>
> ---
>
> **STEP 2: CAPTURE BOTH TRACES**
>
> ```typescript
> use: { trace: 'retain-on-failure' }
> ```
>
> You want a trace from a **failing** run and one from a **passing** run of the same test.
>
> ---
>
> **STEP 3: DIFF THEM**
>
> This is the step that actually finds the bug, and it's the one people skip.
>
> Open both traces side by side and compare:
>
> - **Network** — did a request appear in one and not the other? Did one respond in 200ms and the other in 3s? Did one return a different payload?
> - **Timing** — where did the two runs diverge? The last point they match is where to look.
> - **DOM state** — at the failing assertion, what was actually on the page?
> - **Console** — a JS error in the failing run that isn't in the passing one is a strong lead.
>
> Nine times out of ten the answer is visible in this comparison. Something arrived late, arrived different, or didn't arrive.
>
> ---
>
> **STEP 4: CLASSIFY THE CAUSE**
>
> Before fixing, name the category — it determines the fix:
>
> | What the diff shows | Category | Fix |
> |---|---|---|
> | Response late or missing | Data readiness | Wait on the response |
> | Different data in each run | Shared/non-deterministic data | Test owns its data |
> | Element moving at click time | Animation | Disable animations |
> | Only fails alongside other tests | Interference | Isolate state |
> | Fails at the same step, same error | **Real bug** | Fix the product |
>
> That last row is the one to stay honest about. The most expensive mistake in this whole area is labelling a genuine bug "flaky" and retrying past it into production.
>
> ---
>
> **STEP 5: FIX THE CAUSE, NOT THE CLOCK**
>
> A useful self-check: **if my fix is a longer timeout or a `waitForTimeout`, I haven't found the cause yet.**
>
> ```typescript
> // ❌ Hides it
> await page.waitForTimeout(3000);
> await expect(page.getByTestId('total')).toHaveText('$99');
>
> // ✅ Fixes it
> await page.waitForResponse(r => r.url().includes('/api/cart') && r.ok());
> await expect(page.getByTestId('total')).toHaveText('$99');
> ```
>
> Both go green today. Only one is still green when the API gets slower.
>
> ---
>
> **STEP 6: VERIFY UNDER THE SAME STRESS**
>
> ```bash
> npx playwright test suspect.spec.ts --repeat-each=50 --workers=8
> ```
>
> Fifty clean at the load level that used to break it. One green run means nothing against a 1-in-20 failure — there's a 92% chance a broken test passes a single run.
>
> ---
>
> **The overall shape:** reproduce, observe, classify, fix structurally, verify under stress. The mistake almost everyone makes is starting at step 5."

---

## Quick Command Reference

| Command | Purpose |
|---------|---------|
| `--repeat-each=20` | Establish a failure rate |
| `--workers=8` | Create contention to surface race conditions |
| `--repeat-each=20 --workers=8` | Both — the standard stress run |
| `CI=true npx playwright test` | Use CI config locally |
| `npx playwright show-trace trace.zip` | Replay a failing run |
| `--debug` | Step through with the inspector |
| `--grep @flaky` | Run just the quarantined set |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "My process is reproduce-first, and I'm strict about it, because the failure mode in flakiness debugging is fooling yourself. You change something, the test goes green, you declare victory — and you've actually just run a 1-in-20 failure once.
>
> So: `--repeat-each` with workers turned up until I have a rate. Then traces from both a passing and a failing run, side by side. That comparison is where the answer usually is — a response that came back late in one run, or came back with different content.
>
> The pattern I hit most on a 300-test parallel suite was interference: tests that were fine alone and collided when the scheduler happened to overlap them. The diagnostic for that is cheap — run it alone, run it in the suite. If it only fails in the suite, you've eliminated the entire environmental category in one run and you know you're looking at shared state.
>
> And I verify at the same stress level that exposed it. Fifty clean runs at eight workers, not one green run on a quiet machine."

---

## Common Interview Questions & Answers

### Q1: "Walk me through debugging a flaky test."

**A:** "Reproduce it first with `--repeat-each` and high worker count until I have a failure rate — I won't change code before that, because otherwise I can't tell whether a fix worked. Then capture traces from both a passing and a failing run and diff them. Almost always something arrived late or arrived different, and that comparison shows it. Classify the cause, fix it structurally, then verify with the same stress run."

### Q2: "What if you can't reproduce it?"

**A:** "Turn up the pressure — more workers for contention, higher repeat count, and run it inside the full suite rather than isolated, since interference needs neighbours to show up. If it still won't reproduce, I'd enable trace capture and wait for CI to catch it. What I won't do is make a speculative fix, because then I've changed the code and still don't know if it's fixed."

### Q3: "The trace shows the element was there but the assertion failed. What does that mean?"

**A:** "Element present, data not. That's the classic auto-waiting gap — Playwright waited for the DOM node, but the API response populating it was still in flight. I'd confirm on the network panel of the trace, then wait on the response rather than the element."

### Q4: "How do you know when a flaky test is fixed?"

**A:** "Fifty clean runs under the same conditions that exposed it. A single pass is meaningless — with a 1-in-20 failure rate, a broken test has better than a 90% chance of passing one run. The verification has to be at least as aggressive as the reproduction."

### Q5: "When would you quarantine instead of fix?"

**A:** "When it's blocking the team and I don't yet have the root cause. Quarantine with an owner and a deadline, so it stays visible rather than becoming permanent. What I won't do is leave it flaky in the blocking suite — one ignored red test teaches everyone that red is ignorable, and then the whole suite stops meaning anything."

---

## Weak Areas to Watch

❌ **Don't change code before reproducing**
→ You'll never know whether it worked.

❌ **Don't verify with a single run**
→ Statistically meaningless against an intermittent failure.

❌ **Don't skip the trace diff**
→ It's the step that actually locates the cause.

❌ **Don't reach for `waitForTimeout`**
→ Guaranteed sign the root cause is still unknown.

❌ **Don't assume it's flaky**
→ Rule out a real bug first.

---

## Key Takeaway for Interview

Flaky debugging is a **discipline against self-deception**. The whole method exists to stop you believing a fix that didn't work.

The line that signals experience:

> **"I won't touch the test until I can reproduce it, and I won't call it fixed until fifty runs are clean at the load that broke it. One green run against a 1-in-20 failure proves nothing."**

And the technique that shows real trace fluency:

> **"Diff a passing trace against a failing one. The point where they diverge is the bug."**

---

## Debugging Checklist

✅ Reproduce with `--repeat-each` before changing anything
✅ Establish a failure rate, not an anecdote
✅ Capture traces from both a passing and a failing run
✅ Diff network, timing, DOM, console between them
✅ Classify the cause before fixing
✅ Rule out a real product bug
✅ Fix structurally — never with a timeout
✅ Verify with 50 runs at the same stress level

---

**Remember:** The goal isn't a green run. It's knowing *why* it's green. 🎯
