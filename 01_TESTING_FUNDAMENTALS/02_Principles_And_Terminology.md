# Testing Principles & Terminology

## What This Covers

The seven ISTQB testing principles, the terminology distinctions interviewers probe (error vs defect vs failure, verification vs validation), and risk-based testing.

---

## Why It Matters (Interview Context)

You hold **ISTQB CTFL**. The seven principles are the most quotable thing in that syllabus, and an interviewer can reasonably open with *"name a few testing principles."*

But there's a better reason to know these: **two of them are the strongest arguments you have in real conversations.** "Exhaustive testing is impossible" is how you justify risk-based prioritization to a manager who wants everything tested. "Absence of errors is a fallacy" is how you explain that a green suite doesn't mean a good product.

They're not trivia. They're rhetorical tools.

---

## Your 30-Second Interview Answer

> "The seven principles: testing shows the presence of defects not their absence; exhaustive testing is impossible; test early; defects cluster; the pesticide paradox means the same tests stop finding new bugs; testing is context dependent; and absence-of-errors is a fallacy — a bug-free product that doesn't meet user needs is still a failure. The two I actually use in conversation are exhaustive testing being impossible, which justifies risk-based prioritization, and defect clustering, which tells you where to focus."

---

## Your 2-Minute Deep Answer

> "**The seven principles, with why each matters practically:**
>
> **1. Testing shows the presence of defects, not their absence.**
> You can prove a bug exists. You cannot prove none remain. This is why *"all tests passed"* isn't the same as *"the software works"* — and it's the honest framing when someone asks for a guarantee.
>
> **2. Exhaustive testing is impossible.**
> Every combination of inputs, paths and states is combinatorially impossible for any real system. So you prioritize by risk. **This is the principle I use most in actual conversation** — it's the answer to a manager who wants 100% coverage of everything.
>
> **3. Early testing saves time and money.**
> A defect found at requirements costs a conversation. The same defect found in production costs a hotfix, a release, and possibly customer trust. It's the justification for QA being in refinement rather than at the end of the sprint.
>
> **4. Defects cluster.**
> A small number of modules contain most of the defects. Practically: if something has broken three times, it'll break again — that's evidenced risk, and it's where automation and exploratory effort should concentrate. This is why I treat production bugs as direct input to what gets tested.
>
> **5. The pesticide paradox.**
> Run the same tests repeatedly and they stop finding new defects — the code becomes immune to them. So suites need reviewing and refreshing, not just running. It's also the argument for keeping exploratory testing alongside automation: automation runs the same checks forever by design.
>
> **6. Testing is context dependent.**
> You test a medical device differently from a marketing site. Risk tolerance, regulation, and cost of failure drive the approach. This is the principle behind *"it depends"* being a legitimate answer.
>
> **7. Absence-of-errors is a fallacy.**
> Software can be defect-free and still fail, because it doesn't do what users need. Testing against requirements isn't enough if the requirements are wrong. **This is the one that connects testing to product thinking**, and it's a good one to raise unprompted.
>
> ---
>
> **Terminology — error, defect, failure:**
>
> ```
> Error (mistake)  →  a human does something wrong
> Defect (bug)     →  the resulting flaw in the code
> Failure          →  the defect manifesting during execution
> ```
>
> A developer misreads a requirement (**error**), writes `>` instead of `>=` (**defect**), and a user at exactly the boundary sees the wrong result (**failure**).
>
> The useful implication: **a defect only becomes a failure if it's executed.** A bug in code nobody runs causes no failures — which is exactly why coverage matters, and why untested paths are where dormant defects sit.
>
> ---
>
> **Verification vs Validation:**
>
> - **Verification** — *are we building the product right?* Does it meet the specification. Reviews, static analysis, unit tests.
> - **Validation** — *are we building the right product?* Does it meet the user's actual need. UAT, beta testing, usability.
>
> The memorable framing: you can verify perfectly against a specification and still fail validation, because the specification was wrong. That's principle seven again, from a different angle.
>
> ---
>
> **Static vs Dynamic testing:**
>
> **Static** — examining without executing. Code review, requirements review, static analysis tools. Finds defects earliest and cheapest.
>
> **Dynamic** — executing the code. Everything else.
>
> Worth noting: a requirements review is testing. It finds defects before any code exists, which is the cheapest possible point. People forget static testing counts.
>
> ---
>
> **Risk-based testing:**
>
> Prioritize by **probability of failure × impact of failure**.
>
> ```
> High probability + high impact  →  test heavily, automate, monitor
> Low probability + high impact   →  test the critical paths
> High probability + low impact   →  test lightly
> Low probability + low impact    →  may not test at all
> ```
>
> This is principle two made operational. It's also how you answer *"you have two days and a week's worth of testing — what do you do?"* — you don't test everything shallowly, you test what would hurt most, deeply, and say clearly what you didn't cover.
>
> **Stating the gap matters as much as the coverage.** An unspoken gap is worse than a known one."

---

## The Seven Principles

| # | Principle | Practical use |
|---|-----------|--------------|
| 1 | Defects present, not absent | "All passed" ≠ "it works" |
| 2 | **Exhaustive testing impossible** | Justifies risk-based prioritization |
| 3 | Early testing saves cost | QA in refinement, not at the end |
| 4 | **Defects cluster** | Focus where it's broken before |
| 5 | Pesticide paradox | Refresh suites; keep exploratory |
| 6 | Context dependent | "It depends" is legitimate |
| 7 | Absence-of-errors fallacy | Right product, not just correct code |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Two of these I use as actual arguments rather than trivia.
>
> Defect clustering is why I treat production bugs as direct input to what gets automated. If a module has broken three times, that's evidenced risk rather than a guess — and it earns a test more than something I merely suspect might be fragile. That turns test prioritization from intuition into something you can defend.
>
> Exhaustive testing being impossible is the one I use in conversation with stakeholders. When someone asks for everything to be covered, this is the honest framing — you can't, so the real question is what we prioritize and what we consciously accept. That's a much more productive conversation than a coverage percentage.
>
> The pesticide paradox is the reason I don't treat automation as the whole strategy. A suite runs the same checks forever by design, so it stops finding new things. Exploratory testing finds the category automation structurally can't — it only checks what you thought to tell it.
>
> And principle seven — absence of errors being a fallacy — is worth raising because it connects testing to product. A defect-free feature nobody wanted is still a failed feature. Verifying against a specification doesn't help if the specification is wrong."

---

## Common Interview Questions & Answers

### Q1: "Name some testing principles."

**A:** "Seven of them: testing shows presence of defects not absence, exhaustive testing is impossible, test early, defects cluster, the pesticide paradox, context dependence, and absence-of-errors is a fallacy. The two I use most in practice are exhaustive testing being impossible — which justifies risk-based prioritization — and defect clustering, which tells you where to concentrate effort."

### Q2: "Error, defect, failure — what's the difference?"

**A:** "An error is the human mistake, a defect is the resulting flaw in the code, and a failure is that defect manifesting during execution. Developer misreads a requirement, writes `>` instead of `>=`, user at the boundary gets the wrong answer. The useful implication is that a defect only becomes a failure if the code is executed — which is why untested paths are where dormant defects sit."

### Q3: "Verification vs validation?"

**A:** "Verification is 'are we building the product right' — does it match the specification. Validation is 'are we building the right product' — does it meet the actual need. You can verify perfectly and still fail validation, because the specification was wrong. That's the absence-of-errors fallacy from another angle."

### Q4: "What's the pesticide paradox?"

**A:** "Run the same tests repeatedly and they stop finding new defects — the code becomes immune to them. It means suites need reviewing and refreshing rather than just running, and it's a solid argument for keeping exploratory testing alongside automation, since automation runs identical checks forever by design."

### Q5: "You have two days and a week of testing to do. What do you do?"

**A:** "Risk-based prioritization — probability times impact. I'd cover the revenue and trust paths deeply rather than everything shallowly, plus whatever changed in this release and anything historically fragile near it. And I'd state clearly what wasn't covered, because an unspoken gap is worse than a known one — that lets someone else make an informed release decision."

---

## Weak Areas to Watch

❌ **Don't recite all seven mechanically**
→ Name them, then say which two you actually use.

❌ **Don't confuse verification and validation**
→ "Right product" vs "product right."

❌ **Don't forget static testing counts**
→ A requirements review finds defects before code exists.

❌ **Don't answer the time-pressure question with "test everything quickly"**
→ Depth on what matters, and state the gap.

---

## Key Takeaway for Interview

These aren't trivia — the useful ones are rhetorical tools.

The two worth deploying:

> **"Exhaustive testing is impossible — so the real question isn't whether we cover everything, it's what we prioritize and what we consciously accept."**

> **"Defects cluster. If a module has broken three times, that's evidenced risk rather than a guess — it earns a test more than something I merely suspect is fragile."**

And the one that connects testing to product:

> **"Absence of errors is a fallacy. A defect-free feature nobody wanted is still a failed feature — verifying against a specification doesn't help if the specification is wrong."**

---

## Checklist

✅ Can name all seven principles
✅ Know which two you'd actually use in conversation
✅ Error → defect → failure chain, with an example
✅ Verification vs validation, memorably framed
✅ Static testing recognized as testing
✅ Risk-based prioritization as probability × impact
✅ Habit of stating uncovered gaps explicitly

---

**Remember:** A green suite proves your tests passed, not that the product works. 🎯
