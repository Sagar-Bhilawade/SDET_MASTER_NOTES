# Test Design Techniques (ISTQB)

## What This Covers

The systematic methods for deciding *what* test cases to write — equivalence partitioning, boundary value analysis, decision tables, and state transition testing.

**You hold ISTQB CTFL.** Interviewers see that on your resume and will assume you know these cold. Fumbling one is worse than not having the certification.

---

## Why It Matters (Interview Context)

Two reasons this comes up:

1. **The certification is on your resume.** It's a direct invitation to be tested on it.
2. **It's the answer to "how do you decide what to test?"** — which distinguishes systematic coverage from writing whatever cases occur to you.

The practical value: these techniques give you **maximum coverage with minimum cases**. That's the same efficiency argument as everything else in your notes — every test you write is maintained forever, so writing fewer, better-chosen ones is the goal.

---

## Your 30-Second Interview Answer

> "Equivalence partitioning groups inputs that should behave identically, so I test one value per group instead of every value. Boundary value analysis then targets the edges of those groups, because that's where off-by-one bugs live. Decision tables handle combinations of conditions, and state transition testing covers flows where the same action does different things depending on current state. Together they're how you get high coverage with a small number of cases — which matters because every test is maintained forever."

---

## Your 2-Minute Deep Answer

> "Four techniques, and they layer.
>
> ---
>
> **1. EQUIVALENCE PARTITIONING**
>
> Divide inputs into groups where every value should behave the same, then test one value from each.
>
> *Example — an age field accepting 18-65:*
> ```
> Invalid (too low):   < 18       → test 10
> Valid:               18 - 65    → test 30
> Invalid (too high):  > 65       → test 80
> Invalid (non-numeric):          → test "abc"
> Invalid (empty):                → test ""
> ```
>
> Five cases instead of testing every number. The reasoning: if 30 works, 31 almost certainly works — they're in the same partition and hit the same code path.
>
> **Partitions exist on the invalid side too**, and that's the part people miss. "Too low" and "too high" are separate partitions with potentially different error handling.
>
> ---
>
> **2. BOUNDARY VALUE ANALYSIS**
>
> Bugs cluster at boundaries, because that's where `<` versus `<=` mistakes live. So test the edges of each partition.
>
> *For the same 18-65 field:*
> ```
> 17  → invalid (just below)
> 18  → valid   (lower boundary)
> 19  → valid   (just above)
> 64  → valid   (just below upper)
> 65  → valid   (upper boundary)
> 66  → invalid (just above)
> ```
>
> The two-value version tests just the boundary and one side; the three-value version tests below, on, and above. I'd use three-value on anything where the boundary matters commercially — a credit limit, an age gate, a discount threshold.
>
> **This is the highest-value technique in practice.** Off-by-one at a boundary is the single most common validation bug.
>
> ---
>
> **3. DECISION TABLES**
>
> For business rules with multiple conditions combining.
>
> *Example — discount eligibility:*
>
> | Condition | R1 | R2 | R3 | R4 |
> |---|:-:|:-:|:-:|:-:|
> | Premium member | Y | Y | N | N |
> | Order > ₹5000 | Y | N | Y | N |
> | **Discount applied** | **20%** | **10%** | **5%** | **0%** |
>
> Four rules, four test cases. The value is that building the table **exposes gaps in the requirements** — you fill in the combinations and find one nobody specified. That conversation with the BA is often worth more than the tests.
>
> With N binary conditions you get 2^N combinations, which explodes fast. At that point you collapse rules where the outcome is identical regardless of one condition.
>
> ---
>
> **4. STATE TRANSITION TESTING**
>
> When the same action produces different results depending on current state.
>
> *Example — order lifecycle:*
> ```
> Created → Paid → Shipped → Delivered
>    ↓        ↓
> Cancelled  Refunded
> ```
>
> You test valid transitions, and — more importantly — **invalid ones**: can you cancel a Delivered order? Can you refund something never paid? Those are where real bugs live, because developers implement the happy path and often don't guard the rest.
>
> ---
>
> **How I actually apply this:**
>
> These are mostly design-time techniques, and their biggest value is at the **API layer**, not the UI. Twenty boundary cases through a browser is three minutes of execution; the same twenty as API tests is a few seconds and far more stable.
>
> So my pattern is: use equivalence partitioning and BVA to *derive* the cases, then implement them as data-driven API tests. That's the same test-level argument as everywhere else — and these techniques make the case volume large enough that the level choice really matters."

---

## Technique Reference

| Technique | Use for | Key idea |
|-----------|---------|----------|
| **Equivalence Partitioning** | Input ranges | One value per behaviour group |
| **Boundary Value Analysis** | The edges | Bugs cluster at boundaries |
| **Decision Table** | Combined conditions | Exposes requirement gaps |
| **State Transition** | Stateful flows | Test invalid transitions too |
| **Error Guessing** | Experience-based | Complements, doesn't replace |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "I use these to derive cases, then decide what level to implement them at — and that second decision matters more than people expect.
>
> Equivalence partitioning and boundary analysis on a validation rule might give you fifteen or twenty cases. Run those through the UI and you've added minutes to the suite for logic that never touches the browser. The same cases as data-driven API tests run in seconds and are far more stable. So the techniques generate the coverage; the level decision keeps it affordable.
>
> The one I'd say earns its keep beyond testing is the decision table. Building it forces you to enumerate every combination, and you routinely find one the requirements never specified. That conversation with the BA — 'what should happen when a premium member's order is under the threshold *and* they have an expired coupon?' — is often worth more than the test cases themselves. You're finding the bug before anyone writes the code.
>
> And on state transitions, the valuable half is the invalid ones. Developers implement the happy path carefully and often don't guard against a cancel on a delivered order. That's where the defects are."

---

## Common Interview Questions & Answers

### Q1: "How do you decide what test cases to write?"

**A:** "Systematically rather than by intuition. Equivalence partitioning to group inputs that behave the same, boundary value analysis on the edges of those groups since that's where off-by-one bugs live, decision tables where conditions combine, and state transition testing for stateful flows. The goal is maximum coverage with minimum cases — every test I write is maintained forever, so fewer well-chosen ones beat many arbitrary ones."

### Q2: "Explain equivalence partitioning with an example."

**A:** "Group inputs that should behave identically and test one from each group. An age field accepting 18-65 gives you partitions for below range, in range, above range, non-numeric, and empty — five cases instead of testing every value. If 30 works, 31 works; they're the same code path. The part people miss is that the invalid side has multiple partitions too — too low and too high may have different error handling."

### Q3: "Why boundary value analysis if you already have partitions?"

**A:** "Because bugs cluster at boundaries — that's where `<` versus `<=` mistakes live. Partitioning tells you 30 is a valid age; it doesn't tell you whether 18 and 65 themselves are accepted. I'd test below, on, and above each boundary. It's the highest-yield technique in practice, because off-by-one at a boundary is the most common validation bug there is."

### Q4: "When would you use a decision table?"

**A:** "When multiple conditions combine to produce different outcomes — discount rules, permission logic, pricing tiers. Its real value is that building it exposes gaps in the requirements. You enumerate the combinations and find one nobody specified, which means you're catching the bug before the code exists. That conversation is often worth more than the resulting tests."

### Q5: "What's most important in state transition testing?"

**A:** "The invalid transitions. Developers implement the happy path carefully — created to paid to shipped — and frequently don't guard the rest. Can you cancel a delivered order? Refund something never paid? That's where the defects are, and it's the half people skip."

---

## Weak Areas to Watch

❌ **Don't recite definitions without examples**
→ You're certified; they expect fluency, not textbook recall.

❌ **Don't forget invalid partitions**
→ Too-low and too-high are separate groups with separate handling.

❌ **Don't skip the invalid state transitions**
→ That's where the bugs are.

❌ **Don't implement large derived case sets through the UI**
→ Twenty boundary cases belong at API level.

---

## Key Takeaway for Interview

Your certification makes this a guaranteed question. Answer with examples, not definitions.

The framing that connects to everything else in your notes:

> **"These techniques give maximum coverage with minimum cases — which matters because every test I write is maintained forever. Fewer, better-chosen cases beat many arbitrary ones."**

And the point that shows you apply them rather than recite them:

> **"A decision table's real value is that building it exposes gaps in the requirements. You find the combination nobody specified — that's catching a bug before the code exists."**

---

## Checklist

✅ Can give a worked equivalence partitioning example
✅ Know two-value vs three-value boundary analysis
✅ Can build a decision table and explain the requirement-gap benefit
✅ Test invalid state transitions, not just valid ones
✅ Derive cases with these techniques, implement at the cheapest level
✅ Error guessing named as a complement, not a replacement

---

**Remember:** The techniques decide *what* to test. The level decision decides what it costs. 🎯
