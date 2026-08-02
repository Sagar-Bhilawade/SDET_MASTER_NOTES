# Playwright - Real-World Problems: What Actually Breaks in Production Suites

## What This Topic Covers

Everything the tutorials skip. The problems that show up at month six, not day one.

Research on large Playwright suites consistently finds the same thing: **teams spend up to 50% of their time on test maintenance rather than writing new coverage.** That number is the reason this topic exists. Interviewers at 20 LPA have lived it, and they can tell in thirty seconds whether you have too.

---

## Why It Matters (Interview Context)

This is where interviewers separate "wrote tests" from "owned a suite."

Anyone can describe the happy path. The signal they're looking for:
1. **Have you felt the maintenance burden?** (or only built greenfield?)
2. **Do you recognize the failure patterns?** (or are you surprised each time?)
3. **Do you fix causes or symptoms?**
4. **Can you tell an honest story about something that went wrong?**

An interviewer will often ask: *"What's the worst problem you've hit with your test suite?"* — and a vague answer here is disqualifying at senior level.

---

## Your 30-Second Interview Answer

> "The biggest problems in a mature suite aren't technical bugs — they're erosion. Brittle selectors that break on every redesign, hardcoded data that goes stale, environmental drift between local and CI, and test debt that accumulates silently while the dashboard stays green. My focus is on the practices that prevent erosion: stable locator strategy, data ownership per test, and tracking first-attempt pass rate so decay is visible before it's a crisis."

---

## Your 2-Minute Deep Answer

> "Let me go through the six that actually cost teams time.
>
> **1. BRITTLE SELECTORS**
>
> The classic. Tests written against CSS classes or deep DOM paths. Designer ships a redesign, 80 tests go red, none of them because of a real bug.
>
> *Why it happens:* CSS classes are an implementation detail. They're allowed to change — that's what they're for.
> *Prevention:* Role-based locators first, `data-testid` second. And crucially, get `data-testid` treated as a contract with the dev team, not a favour they do you. Once developers understand that removing a testid breaks the pipeline, they stop removing them.
>
> **2. LOCAL vs CI DRIFT**
>
> Passes on your machine, fails in the pipeline. Different timing, different data volume, different screen size, cold caches, contended CPU.
>
> *Why it happens:* Your laptop is a fast, warm, single-tenant machine with a small dev database. The CI runner is none of those things.
> *Prevention:* Run against the same containerized environment, keep viewport explicit in config, and never assume the dataset size is the same.
>
> **3. HARDCODED EVERYTHING**
>
> URLs, credentials, dates, IDs, payment details baked into test files.
>
> *Why it happens:* It's faster on day one.
> *Cost:* Credentials in a repo is a security incident. A hardcoded date makes the suite fail on a specific day next year. A hardcoded record ID breaks the day someone cleans the database.
> *Prevention:* `baseURL` in config, credentials in env vars or a secret store, relative dates computed at runtime.
>
> **4. UI-HEAVY TEST COVERAGE**
>
> Everything tested through the browser, including things with no UI reason to be there.
>
> *Why it happens:* The team owns UI tests, so every requirement becomes a UI test.
> *Cost:* Slow, flaky, expensive to maintain — for validation that an API test would cover in 200ms with far more stability.
> *Prevention:* Push logic validation down to the API layer. Reserve UI tests for what genuinely requires a browser: rendering, interaction, integration across screens.
>
> This is a strong point to raise unprompted, because it shows you think about test *strategy*, not just test *implementation*.
>
> **5. SILENT TEST DEBT**
>
> The dangerous one, because nothing goes red.
>
> Three things accumulate quietly:
> - Tests that pass but assert nothing meaningful (assertion was weakened during a past debugging session and never restored)
> - Tests that are permanently skipped and forgotten
> - Tests that only pass because of retries
>
> Your dashboard says green. Your actual coverage is a fraction of what the count implies.
>
> *Prevention:* Track first-attempt pass rate, audit skipped tests on a schedule, and treat a weakened assertion as a bug.
>
> **6. THIRD-PARTY DRIFT**
>
> If your flow crosses into a system you don't control — a payment provider, an ERP, a CRM — their UI changes on their schedule, and your tests break without any commit on your side.
>
> *Prevention:* Mock the third party for functional tests. Keep a small, separate, non-blocking suite that hits the real integration so you find out when it changes — but it doesn't block your team's merges.
>
> **The pattern across all six:** none of them are Playwright problems. They're ownership problems. Playwright gives you excellent tools; the suite still decays if nobody is watching the trend lines."

---

## The Six Problems at a Glance

| Problem | Symptom | Prevention |
|---------|---------|-----------|
| **Brittle selectors** | Redesign kills 80 tests | Role locators + testid as a contract |
| **Local/CI drift** | Green locally, red in pipeline | Containerized env, explicit viewport |
| **Hardcoded values** | Stale data, leaked secrets | Config + env vars + relative dates |
| **UI-heavy coverage** | Slow, flaky, expensive | Push validation to API layer |
| **Silent test debt** | Green board, no real coverage | First-attempt rate, skip audits |
| **Third-party drift** | Breaks with no commit | Mock it; separate non-blocking integration suite |

---

## Your Real Project Example (NielsenIQ)

**A strong way to tell this — pick ONE problem and go deep:**

> "The one that cost us the most was selector stability. Early on, tests were written against whatever selector was convenient at the time, which meant CSS classes in a lot of places. Every time the design team shipped a change, we'd get a wave of failures that had nothing to do with product bugs — and the team started treating red as normal, which is the real damage.
>
> The fix had two halves. The technical half was a locator hierarchy — role first, `data-testid` second, CSS only as a genuine last resort. The organizational half mattered more: we got `data-testid` recognized as a contract with the dev team rather than something QA bolted on. Once removing a testid was understood to break the pipeline, they stopped being removed casually.
>
> What I'd call out is that the second half was the harder one. The technical pattern is easy — it's a page-object refactor. Getting another team to treat your test hooks as production concerns takes actual conversation."

**Why this version of the story works:**

Every interviewer at this level has lived through a selector crisis. The structure that lands is: **symptom → what I found when I dug in → the fix → the part that was harder than expected.**

That last beat is what makes it credible. A story where everything went smoothly reads as rehearsed. Naming the organizational friction — getting another team to treat your test hooks as production concerns — is exactly the signal that separates a 20 LPA candidate from a strong mid-level one.

---

## Common Interview Questions & Answers

### Q1: "What's the biggest problem you've faced with your test suite?"

**A:** Pick one, go deep. Structure: *the symptom → what I found when I dug in → the fix → what surprised me.* The last part is what makes it sound real. Avoid listing five problems shallowly — depth beats breadth here every time.

### Q2: "How do you prevent test maintenance from eating your team's time?"

**A:** "Three things, in order of impact. First, locator strategy — most maintenance is selectors breaking, and role-based locators plus testids kill the majority of it. Second, test level — a lot of UI tests shouldn't be UI tests. Moving logic validation to the API layer removes maintenance permanently rather than reducing it. Third, visibility — track first-attempt pass rate so you see decay early instead of discovering it during a release."

### Q3: "How do you handle testing a flow that touches a third-party system?"

**A:** "Mock it for the functional tests — I need those deterministic and fast, and a payment provider's sandbox going down shouldn't block a merge. Then keep a small integration suite that hits the real thing on a schedule, non-blocking. That way I find out when their contract changes, but their availability isn't coupled to my team's velocity."

### Q4: "Your test suite is green but bugs are reaching production. What's happening?"

**A:** "That's the silent-debt signature, and I'd look at three things. Are the assertions actually meaningful — is anything asserting `toBeVisible` on a container and calling that a check? How many tests are skipped and forgotten? And what's the first-attempt pass rate, because if retries are carrying the suite, the green is cosmetic. Green with no coverage is worse than red, because it buys false confidence."

### Q5: "How much of your time goes to maintenance vs new tests?"

**A:** Be honest — the industry number is around 50%, and claiming 5% will read as either inexperience or spin. The good answer names your number and what you did to move it. *"It was high, here's what I identified as the driver, here's what changed."*

---

## Weak Areas to Watch

❌ **Don't claim your suite has no problems**
→ Nobody believes it, and it signals you haven't operated at scale.

❌ **Don't blame the tool**
→ "Playwright is flaky" tells them you haven't diagnosed the actual cause.

❌ **Don't list problems without fixes**
→ Diagnosis plus remedy. Complaints alone read as junior.

❌ **Don't ignore the organizational dimension**
→ Selector contracts, dev-team collaboration, and coverage strategy are people problems as much as technical ones. Naming that is a seniority signal.

---

## Key Takeaway for Interview

Mature suites don't fail from bugs. They **erode** — through brittle selectors, silent debt, and drift.

Your value at 20 LPA is that you see erosion coming and build against it.

The line that lands:

> **"Green with weak assertions is more dangerous than red, because red gets fixed and green gets trusted."**

---

## Erosion Prevention Checklist

✅ Locator hierarchy enforced (role → testid → text → css)
✅ `data-testid` treated as a contract with dev team
✅ Zero hardcoded URLs, credentials, dates, or IDs
✅ Validation pushed to the API layer where a browser isn't needed
✅ First-attempt pass rate tracked
✅ Skipped tests audited on a schedule
✅ Third-party systems mocked; real integration suite is separate and non-blocking
✅ Assertions reviewed for meaning, not just presence

---

**Remember:** The suite you don't maintain is the suite you'll eventually delete. 🎯
