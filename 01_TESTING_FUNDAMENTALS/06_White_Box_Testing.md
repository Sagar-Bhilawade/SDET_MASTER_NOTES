# White-Box Testing

## What This Covers

Testing with knowledge of the internal code — coverage criteria, static analysis, and where an SDET actually uses this.

**This is on your resume**, so expect it to be probed. The risk is answering it as a textbook definition rather than something you've applied.

---

## Why It Matters (Interview Context)

The distinction most candidates give — "white box is with code knowledge, black box is without" — is correct and shallow. It's the first sentence, not the answer.

What separates a real answer: knowing the **coverage criteria** (statement, branch, path), understanding why 100% statement coverage proves very little, and being honest that as an SDET your white-box work is mostly *reading* code rather than writing unit tests.

---

## Your 30-Second Interview Answer

> "White-box testing uses knowledge of the internal implementation to design tests, versus black-box which works from requirements only. The measures are statement coverage, branch coverage, and path coverage — and the important thing is that 100% statement coverage doesn't mean much, because you can execute every line without exercising both sides of a condition. Branch coverage is the more honest number. As an SDET my white-box work is mostly reading the code to find the edge cases the requirements didn't mention, and reviewing coverage reports to see what's genuinely untested."

---

## Your 2-Minute Deep Answer

> "**The three coverage criteria, and why the difference matters:**
>
> ```java
> public String checkAccess(User user) {
>     if (user.isActive() && user.hasPermission()) {
>         return "granted";
>     }
>     return "denied";
> }
> ```
>
> **Statement coverage** — did every line execute?
> Two tests get you 100%: one granted, one denied. Every line ran.
>
> **Branch coverage** — did every decision go both ways?
> Still fine with those two tests for the `if` as a whole.
>
> **Condition coverage** — did each individual condition evaluate both true and false?
> Now those two tests are *not* enough. If both tests have `isActive() == true`, you've never tested an inactive user. A bug in that path is invisible.
>
> **That's the point worth making:** 100% statement coverage is a weak signal. You can hit every line while leaving whole logical paths unexercised. Branch and condition coverage are the honest measures, and even those don't guarantee correctness — they guarantee execution, not that anyone asserted the right thing.
>
> Which connects to something in my own testing: a suite can have high coverage and weak assertions. Coverage tells you code *ran*. It doesn't tell you anything was *checked*.
>
> ---
>
> **Path coverage** — every possible route through the function. Combinatorially explosive; useful as a concept, rarely a practical target.
>
> ---
>
> **Where I actually use white-box knowledge as an SDET:**
>
> **1. Reading code to find edge cases the requirements missed.**
>
> This is the highest-value use. The requirement says "validate the email field." The code shows a regex, a length cap, and a domain blocklist nobody documented. Those last two are test cases you'd never derive from the spec.
>
> **2. Reviewing coverage reports to target gaps.**
>
> Not to chase a percentage — to find the module with a genuinely untested branch and ask whether it matters. Coverage as a diagnostic, not a target.
>
> **3. Understanding failures faster.**
>
> When a test fails, being able to read the implementation means you diagnose rather than guess — and you file a much better bug report, because you can point at the condition rather than describe symptoms.
>
> **4. Reviewing developer unit tests.**
>
> Checking that they assert meaningfully rather than just executing the code. Same weak-assertion problem as anywhere else.
>
> ---
>
> **Static analysis — the automated half:**
>
> Tools like SonarQube, SpotBugs, or ESLint analyse code without running it, flagging null-dereference risks, unused code, complexity, and security patterns.
>
> The metric worth knowing is **cyclomatic complexity** — the number of independent paths through a function. High complexity means more paths, which means more tests needed and more places for bugs. It's a useful signal for *where to focus testing effort*: a method with complexity 15 deserves far more attention than one with complexity 2.
>
> That's a genuinely practical use of a white-box metric for a tester.
>
> ---
>
> **The honest boundary:**
>
> I'd be straightforward that unit testing is primarily a developer responsibility, and my white-box work is mostly reading rather than writing. Claiming to own unit test suites when you don't would unravel in one follow-up. What I do own is using code knowledge to design better integration and API tests, and to push validation to the right level.
>
> Grey-box testing is really the accurate description of most SDET work — partial knowledge of internals, testing from outside. Knowing the database schema, the API contracts, and roughly how the service is structured, while testing through the interfaces."

---

## Coverage Criteria

| Criterion | Asks | Strength |
|-----------|------|----------|
| **Statement** | Did every line run? | Weak — misses logic paths |
| **Branch** | Did every decision go both ways? | Better |
| **Condition** | Did each condition evaluate both ways? | Stronger |
| **Path** | Every route through? | Thorough, usually impractical |

---

## Black vs White vs Grey

| | Knowledge | Who | Typical use |
|---|---|---|---|
| **Black-box** | Requirements only | QA | UI, E2E, acceptance |
| **White-box** | Full internals | Developers | Unit tests, coverage |
| **Grey-box** | Partial — schema, contracts | **SDET** | Integration, API, DB validation |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "The most valuable white-box work I do is reading implementation code to find edge cases the requirements never mentioned. A spec says 'validate the email field'; the code shows a length cap and a domain blocklist nobody wrote down. Those are test cases you can't derive from the requirement, and they're often where the defects are.
>
> On coverage, the point I'd make is that 100% statement coverage is a weak signal. You can execute every line without exercising both sides of a condition — if every test happens to pass an active user, the inactive path is untested regardless of what the percentage says. Branch and condition coverage are the honest measures.
>
> And that connects to something I care about in my own suites: coverage tells you code *ran*, not that anything was *checked*. A test with a weak assertion contributes coverage and catches nothing. It's the same failure in a different metric.
>
> Cyclomatic complexity is the static-analysis number I find genuinely useful — it tells you where to concentrate testing effort. A method with fifteen paths deserves far more attention than one with two.
>
> I'd also be straight that unit testing is mostly a developer responsibility. My work is grey-box really — I know the schema and the API contracts and roughly how things are structured, and I test through the interfaces with that knowledge."

---

## Common Interview Questions & Answers

### Q1: "What's white-box testing?"

**A:** "Designing tests using knowledge of the internal implementation, rather than working purely from requirements. It's measured by coverage criteria — statement, branch, condition, path. As an SDET most of my work is really grey-box: I know the schema and the API contracts, and I test through the interfaces with that knowledge."

### Q2: "Is 100% code coverage a good goal?"

**A:** "No, for two reasons. First, 100% statement coverage can be achieved without exercising both sides of a condition — every line runs, whole logic paths stay untested. Branch and condition coverage are more honest. Second, coverage measures execution, not verification. A test with a weak assertion contributes coverage and catches nothing. Chasing the number produces low-value tests that inflate maintenance cost without reducing risk."

### Q3: "Difference between statement and branch coverage?"

**A:** "Statement asks whether every line executed; branch asks whether every decision went both ways. Take `if (a && b)` — two tests can hit every line and still never test the case where `a` is false. That gap is exactly why statement coverage overstates how well-tested something is."

### Q4: "How do you use static analysis?"

**A:** "Mainly cyclomatic complexity, to decide where to focus testing effort — a method with fifteen independent paths deserves far more attention than one with two. Beyond that, tools like SonarQube flag null-dereference risks and security patterns without running the code, which catches a class of issue that testing wouldn't surface efficiently."

### Q5: "Do you write unit tests?"

**A:** "Unit testing is primarily a developer responsibility, and I'd rather be straight about that than overclaim. What I do is read the code to design better integration and API tests, review developer unit tests for whether they assert meaningfully, and use coverage reports as a diagnostic for gaps worth caring about."

---

## Weak Areas to Watch

❌ **Don't stop at "with code knowledge vs without"**
→ Correct but shallow. Name the coverage criteria.

❌ **Don't defend 100% coverage as a goal**
→ It measures execution, not verification.

❌ **Don't overclaim unit test ownership**
→ Unravels in one follow-up. Grey-box is the honest description.

❌ **Don't ignore the coverage/assertion connection**
→ Weak assertions inflate coverage and catch nothing — same problem, different metric.

---

## Key Takeaway for Interview

This is on your resume, so know it past the definition.

The insight that shows real understanding:

> **"100% statement coverage is a weak signal. `if (a && b)` — two tests can execute every line and never test the case where `a` is false. Branch and condition coverage are the honest measures."**

And the connection to test quality generally:

> **"Coverage tells you code ran. It doesn't tell you anything was checked. A weak assertion contributes coverage and catches nothing — same failure as a green suite that misses bugs."**

---

## Checklist

✅ Can explain statement / branch / condition / path
✅ Can show why statement coverage overstates quality
✅ Cyclomatic complexity used to target testing effort
✅ Coverage treated as diagnostic, not target
✅ Honest about unit testing being developer-owned
✅ Grey-box named as the accurate description of SDET work

---

**Remember:** Coverage measures what ran, not what was verified. 🎯
