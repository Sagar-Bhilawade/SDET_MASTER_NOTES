# Leadership - Mentoring & Code Review

## What This Topic Covers

How you make other engineers better — and how you keep quality high across a team rather than just in your own work.

This is the topic that most distinguishes a 20 LPA offer from a 14 LPA one. At mid-level, your value is what you build. At senior level, your value includes **what the people around you build**.

---

## Why It Matters (Interview Context)

Expect at least one of:

> "How would you onboard a junior SDET onto your framework?"
> "Tell me about a time you mentored someone."
> "What do you look for in a test code review?"

Many strong technical candidates lose the offer here, because they've never articulated it. The good news: if you've ever explained something to a colleague, unblocked someone, or reviewed a PR, you have material — it just needs framing.

---

## Your 30-Second Interview Answer

> "My approach to onboarding is to get someone shipping a real test on day one, not reading documentation for a week. Pair on the first one, let them drive the second, review the third. For code review, I separate what's blocking from what's preference — blocking is anything that creates flakiness or hides a failure, preference is style. Being clear about which is which is what keeps review from feeling like gatekeeping."

---

## Your 2-Minute Deep Answer

> "Two halves — bringing people in, and keeping quality up once they're in.
>
> ---
>
> **ONBOARDING**
>
> The mistake is handing someone documentation and expecting them to absorb a framework abstractly. Test frameworks are learned by writing tests.
>
> My sequence:
>
> **Day 1 — ship something real.** A small test, paired, with me driving the setup and them driving the assertions. It goes into the actual suite. The point is confidence: they've contributed to the real thing on day one.
>
> **Week 1 — they drive, I watch.** They pick the next test, I stay available. They'll go the wrong way on something — a CSS selector, a hardcoded user — and that's the moment the convention actually lands, because it's attached to their own code rather than a wiki page.
>
> **Week 2-3 — normal review.** They work independently, I review PRs. By now they've seen the patterns three or four times.
>
> **Ongoing — hand over a real area.** Ownership is what turns someone from a contributor into an engineer. Nothing accelerates learning like being responsible for something.
>
> The thing I watch for is whether the framework is teachable. If a new person takes weeks to be productive, that's a signal about my design, not about them. Onboarding is the best diagnostic you have for over-abstraction.
>
> ---
>
> **CODE REVIEW**
>
> What I look for in test code, roughly in order of severity:
>
> **1. Will this be flaky?** Hardcoded waits, shared data, order dependence, assertions on ambient state. This is blocking, always — a flaky test is worse than no test because it erodes trust in the whole suite.
>
> **2. Does the assertion actually assert something?** The common failure is a test that goes through a flow and then checks `toBeVisible` on a container that's always visible. It passes forever and catches nothing. Green with no coverage is more dangerous than red.
>
> **3. Is this at the right level?** A large share of proposed UI tests are validating logic with no UI dependency. Suggesting it moves to an API test is one of the highest-value review comments available — it's faster, more stable, and permanently cheaper.
>
> **4. Will this survive a UI change?** CSS selectors, deep DOM paths.
>
> **5. Is it readable?** Can someone tell what it verifies in ten seconds?
>
> **How I give the feedback matters as much as what I flag.**
>
> I mark clearly whether something is **blocking** or a **suggestion**. Reviews where every comment reads with equal weight are exhausting and cause people to either argue about everything or accept everything, both of which are bad.
>
> I explain the *why*, once. "Use `getByRole` here" is an instruction they'll follow. "Use `getByRole` here — CSS classes change during redesigns and this'll break next time design ships" is a rule they'll apply on their own next time. Instructions scale linearly with review effort; reasons scale with the person.
>
> And I ask rather than assert when I'm unsure. "What happens if this runs in parallel with the other checkout test?" gets a better outcome than "this will break in parallel" — sometimes they've already handled it, and if they haven't, they find the problem themselves, which sticks better.
>
> ---
>
> **The scale problem:**
>
> Review doesn't scale as the team grows. What scales:
>
> - **Lint rules** for the mechanical things. `no-wait-for-timeout` as a lint error is better than catching it in review every week — machines should enforce what machines can.
> - **A worked reference test.** One exemplary test people copy is worth more than a style guide nobody opens.
> - **Fixing the framework so the wrong thing is hard.** If people keep hardcoding users, the real fix is a fixture that makes generating one easier than hardcoding. Repeated identical review comments are a design smell, not a discipline problem.
>
> That last one is the senior instinct: when you're giving the same feedback repeatedly, the problem is the system, not the people."

---

## Code Review Priorities

| Priority | Check | Blocking? |
|----------|-------|:---:|
| 1 | Flakiness risk — waits, shared data, order dependence | ✅ Always |
| 2 | Assertion actually verifies something meaningful | ✅ Always |
| 3 | Right test level — could this be an API test? | ⚠️ Discuss |
| 4 | Selector stability | ⚠️ Usually |
| 5 | Readability and naming | 💬 Suggest |
| 6 | Style and formatting | 🤖 Automate it |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "When someone joins the framework, I get them shipping a real test on day one rather than reading documentation. Paired first test, they drive the second, normal review by the third. The reasoning is that a framework is learned by using it — conventions only stick when they're attached to code the person wrote themselves.
>
> In review, the thing I'm most rigorous about is whether an assertion actually verifies something. The failure mode that worries me isn't a test that fails — it's a test that passes without checking anything. Someone walks a flow and asserts a container is visible. It's green forever and catches nothing, and it's counted in our coverage. Green with no coverage is worse than red, because red gets fixed and green gets trusted.
>
> The other thing I've learned is to treat repeated review comments as a design problem. If I'm telling three different people not to hardcode a test user, the issue isn't their discipline — it's that hardcoding is currently easier than the right way. The fix is making the fixture more convenient than the shortcut, not repeating myself in review."

---

## Common Interview Questions & Answers

### Q1: "How would you onboard a junior SDET?"

**A:** "Shipping a real test on day one, paired. They drive the second, I review the third. Frameworks are learned by writing tests, not reading about them — and conventions only stick when attached to their own code. I'd also treat their ramp time as feedback on my design: if it takes weeks, the framework is too abstract or under-documented, and that's my problem to fix."

### Q2: "What do you look for in a test code review?"

**A:** "In order: will it be flaky, does the assertion actually verify something, is it at the right level, will it survive a UI change. Flakiness and empty assertions are blocking — a flaky test erodes trust in the whole suite, and a test that passes without checking anything is worse than not having it. Style I'd rather automate than argue about."

### Q3: "How do you give feedback without discouraging someone?"

**A:** "Mark what's blocking versus what's a suggestion, so it's clear what actually needs to change. Explain the reason once rather than just the instruction — a reason they can apply themselves next time, an instruction they can only follow. And ask when I'm not certain: 'what happens if this runs in parallel?' lands better than an assertion, and often they've already thought about it."

### Q4: "Tell me about a time you mentored someone."

**A:** Use a real, small story. Structure: *what they were stuck on → what I did → what changed → what I learned.* The last part matters — mentoring answers that are purely about how you helped read as self-congratulatory. Something like realizing your explanation was the problem, or that the framework was harder to learn than you thought, makes it credible.

### Q5: "You keep giving the same review comment. What do you do?"

**A:** "Stop giving it and fix the system. If three people hardcode a test user, the shortcut is easier than the right way, and that's a design problem. Either make the fixture more convenient, or enforce it with a lint rule. Repeated identical feedback means the framework is inviting the mistake — repeating myself is treating a symptom."

---

## Weak Areas to Watch

❌ **Don't say you have no mentoring experience**
→ Explaining something to a colleague, unblocking someone, reviewing a PR — that's mentoring. Frame it.

❌ **Don't describe review as catching mistakes**
→ It's a teaching mechanism. Frame it that way.

❌ **Don't treat all review comments as equal weight**
→ Blocking versus suggestion. Say which.

❌ **Don't blame people for repeated mistakes**
→ Repeated mistakes are a system signal.

❌ **Don't tell a mentoring story with no self-reflection**
→ It reads as self-promotion without it.

---

## Key Takeaway for Interview

At 20 LPA, your value includes what the people around you produce.

The framing that lands:

> **"A framework is learned by writing tests, not reading about them. If a new person takes weeks to be productive, that's feedback on my design, not on them."**

And the one that signals real seniority:

> **"When I find myself giving the same review comment repeatedly, the problem is the framework, not the people. If hardcoding is easier than the right way, the fix is making the right way easier — not repeating myself."**

---

## Mentoring Checklist

✅ New joiner ships a real test on day one
✅ Pair → they drive → normal review, over ~3 weeks
✅ Real ownership handed over once they're settled
✅ Onboarding time treated as a design signal
✅ Review comments marked blocking vs suggestion
✅ Reasons given, not just instructions
✅ Mechanical checks automated via lint
✅ One exemplary reference test to copy
✅ Repeated feedback triggers a framework fix

---

**Remember:** The best mentoring result is someone who no longer needs you. 🎯
