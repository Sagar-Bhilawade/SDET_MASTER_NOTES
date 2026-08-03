# Defect Lifecycle & Bug Reporting

## What This Covers

How a defect moves from discovery to closure, the severity-versus-priority distinction, and what makes a bug report actionable rather than annoying.

---

## Why It Matters (Interview Context)

This is a guaranteed question at your experience level, and the severity/priority one is asked almost verbatim:

> "What's the difference between severity and priority? Give an example where they differ."

It's easy marks if you have a crisp example ready, and it's a visible gap if you don't — because it signals you've automated tests without participating in the defect process.

The deeper signal: **a good bug report is a communication skill.** How you write one says a lot about how you work with developers.

---

## Your 30-Second Interview Answer

> "Severity is technical impact — how badly the system is broken. Priority is business urgency — how soon it needs fixing. They're set by different people and often diverge: a typo in the company name on the homepage is low severity and high priority, because it's cosmetic but embarrassing publicly. A crash in a feature nobody uses is high severity, low priority. For the report itself, the thing that matters most is reproducibility — steps, expected, actual, environment, evidence."

---

## Your 2-Minute Deep Answer

> "**The lifecycle:**
>
> ```
> New → Assigned → Open (in progress) → Fixed → Retest → Closed
>                                          ↓
>                                       Reopened (if retest fails)
>
> Other terminal states: Rejected, Duplicate, Deferred, Not Reproducible
> ```
>
> The states that matter in conversation:
>
> **Reopened** — the fix didn't work, or it broke something adjacent. A high reopen rate is a signal worth tracking; it usually means fixes are being made without understanding the root cause.
>
> **Deferred** — accepted as real, scheduled for later. Legitimate, but it should have an owner and a target, otherwise "deferred" is just "closed" with extra steps.
>
> **Not Reproducible** — this one needs care. It often means the report lacked environment detail or preconditions, which is a reporting failure rather than a phantom bug. When I get one back, my first move is to check what I left out.
>
> ---
>
> **Severity vs Priority — the distinction:**
>
> - **Severity** = technical impact. Usually set by QA. How broken is it?
> - **Priority** = business urgency. Usually set by product or the PO. How soon must it be fixed?
>
> They're independent, and the interesting cases are where they diverge:
>
> | Scenario | Severity | Priority | Why |
> |---|---|---|---|
> | Company name misspelled on homepage | Low | **High** | Cosmetic, but publicly embarrassing |
> | Crash in a rarely-used admin report | **High** | Low | Severe, but almost nobody hits it |
> | Payment fails at checkout | High | High | Broken and revenue-critical |
> | Tooltip misaligned on an internal page | Low | Low | Nobody's blocked |
>
> The first two are the ones to have ready — they're the answer that shows you understand the two axes are genuinely independent.
>
> ---
>
> **What makes a bug report actionable:**
>
> ```
> Title:        Concise and specific — "Checkout fails with valid Visa when
>               cart contains a discounted item"
>
> Environment:  Build/version, browser, OS, environment (staging/prod), user role
>
> Preconditions: The state needed before the steps make sense
>
> Steps:        Numbered, minimal, reproducible from a clean state
>
> Expected:     What should happen
> Actual:       What does happen — including the exact error text
>
> Evidence:     Screenshot, video, trace, network log, correlation ID
>
> Frequency:    Always / intermittent (with a rate if you have one)
> ```
>
> **The pieces people leave out, and why they matter:**
>
> **Minimal steps.** If it takes twelve steps and only four matter, you've made the developer do your reduction work. Narrowing it down before filing is the single highest-value thing you can do.
>
> **Exact error text**, not "it shows an error." That string is often searchable straight into the codebase.
>
> **Correlation ID or request ID** where the system has one. That lets a developer find the exact request in the logs in seconds instead of guessing at timestamps.
>
> **Frequency.** "Fails every time" and "fails roughly one in ten" lead to completely different investigations.
>
> ---
>
> **The automation connection:**
>
> When an automated test finds a defect, the report writes itself — the trace shows every action, the network calls, and the DOM at failure. Attaching a Playwright trace or a Selenium screenshot with the failure log is often more useful than a written repro, because the developer can replay it exactly.
>
> That's a real argument for artifact capture in CI: it isn't just for debugging your own tests, it's what makes your bug reports credible and fast to act on.
>
> ---
>
> **A judgement point worth raising:**
>
> Not every defect should be filed. A trivial cosmetic issue found mid-sprint on a feature still in development is often better raised directly with the developer than logged formally — filing it creates triage overhead for something that'll be fixed in an hour anyway. Knowing when the process adds value versus when it's ceremony is part of working well with a team."

---

## Lifecycle States

| State | Means | Watch for |
|-------|-------|-----------|
| New | Filed, untriaged | |
| Assigned | Owned by a developer | |
| Open | Being worked | |
| Fixed | Dev believes resolved | |
| Retest | Back with QA | |
| Closed | Verified fixed | |
| **Reopened** | Fix failed | High rate = root causes not found |
| Deferred | Real, scheduled later | Needs owner + target |
| Not Reproducible | Couldn't repro | Often a reporting gap |
| Duplicate / Rejected | | |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "The habit that made the biggest difference to how my reports were received was minimizing the repro before filing. If something takes twelve steps and only four actually matter, filing all twelve makes the developer do the reduction work — and it slows the fix. Narrowing it down first is the highest-value thing you can do for a bug report.
>
> The other thing is evidence. When an automated test catches something, I attach the trace rather than writing a prose repro. The developer replays the exact run — every action, every network call, the DOM at failure. That's far more useful than my description of it, and it's a real argument for capturing artifacts in CI beyond debugging your own tests.
>
> On severity versus priority, the example I'd give is a misspelled company name on a public page. Technically trivial — nothing is broken. But it's publicly visible and embarrassing, so priority is high while severity is low. The inverse is a crash in an obscure admin report: severe, but almost nobody reaches it.
>
> And I'd say not every defect needs formal filing. Something cosmetic on a feature still under active development is usually better raised directly with the developer — logging it creates triage overhead for something that'll be gone in an hour. Knowing when the process helps and when it's ceremony matters."

---

## Common Interview Questions & Answers

### Q1: "Difference between severity and priority?"

**A:** "Severity is technical impact, usually set by QA — how broken is it. Priority is business urgency, usually set by product — how soon must it be fixed. They're independent axes. A misspelled company name on the homepage is low severity, high priority: nothing's broken but it's publicly embarrassing. A crash in a rarely-used admin report is the reverse."

### Q2: "Walk me through the defect lifecycle."

**A:** "New, assigned, open while it's being worked, fixed, retest with QA, then closed if verified — or reopened if the fix didn't hold. Alongside that: rejected, duplicate, deferred, not reproducible. The two I'd watch are reopened, because a high rate means fixes are being made without finding root causes, and not-reproducible, which is often a reporting gap rather than a phantom bug."

### Q3: "What makes a good bug report?"

**A:** "Reproducibility above everything — minimal numbered steps from a clean state, expected versus actual, environment and build, and the exact error text rather than 'it shows an error.' Then evidence: screenshot, trace, correlation ID. And frequency, because 'always' and 'one in ten' lead to completely different investigations."

### Q4: "A developer marks your bug 'not reproducible.' What do you do?"

**A:** "Assume my report was incomplete before assuming they're wrong. Usually it's a missing precondition, a different environment or data set, or a specific user role. I'd add what I left out — build number, exact data, role — and where possible attach a trace so they can replay it rather than reconstruct it. If it genuinely is environmental, that's worth knowing in itself."

### Q5: "Should every defect be logged?"

**A:** "No, and I'd say so directly. A trivial cosmetic issue on a feature still in active development is usually better raised with the developer directly — formally logging it creates triage overhead for something that'll be fixed within the hour. Anything that reaches a release branch, affects a user, or needs tracking should absolutely be logged. Knowing the difference is part of working well with a team rather than following process for its own sake."

---

## Weak Areas to Watch

❌ **Don't confuse severity and priority**
→ Have both divergent examples ready.

❌ **Don't file un-minimized repro steps**
→ You're offloading reduction work onto the developer.

❌ **Don't write "it shows an error"**
→ Exact text is often searchable in the codebase.

❌ **Don't argue when marked not-reproducible**
→ Check what you omitted first.

❌ **Don't claim everything must be formally logged**
→ Shows process-following rather than judgement.

---

## Key Takeaway for Interview

Easy marks if prepared, visible gap if not.

The severity/priority example that lands:

> **"A misspelled company name on the homepage — low severity, high priority. Nothing's broken, but it's publicly embarrassing. The inverse is a crash in a rarely-used admin report: high severity, low priority."**

And the habit that shows you work well with developers:

> **"Minimize the repro before filing. If it takes twelve steps and four matter, filing all twelve makes the developer do your reduction work."**

---

## Checklist

✅ Severity vs priority, with divergent examples both ways
✅ Full lifecycle including reopened, deferred, not-reproducible
✅ Reports carry environment, build, role, exact error text
✅ Steps minimized before filing
✅ Trace or artifact attached where automation found it
✅ Frequency stated
✅ Judgement about what's worth formally logging

---

**Remember:** A bug report is a message to a developer. Write it so they can act, not so you've recorded it. 🎯
