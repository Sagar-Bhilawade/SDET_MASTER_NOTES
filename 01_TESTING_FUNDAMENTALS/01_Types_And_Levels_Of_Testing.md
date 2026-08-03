# Types & Levels of Testing

## What This Covers

The classification questions — functional vs non-functional, the four test levels, and the pairs interviewers love: smoke vs sanity, regression vs retesting.

---

## Why It Matters (Interview Context)

These are **warm-up questions**, and that's exactly why they're dangerous. An interviewer opens with *"difference between smoke and sanity"* expecting a quick clean answer before moving to the real material.

Fumble it and you've set a bad tone before reaching the automation depth where you're strong. Nail it in fifteen seconds and you move on.

The senior addition: knowing that some of these distinctions matter operationally and some are mostly vocabulary. Saying which is which is a differentiator.

---

## Your 30-Second Interview Answer

> "Test levels go unit, integration, system, then acceptance — increasing scope, decreasing isolation. Functional testing checks what the system does; non-functional checks how well it does it — speed, security, usability. Smoke is a shallow check that the build is worth testing at all; sanity is a narrow check that one specific fix works. Regression is verifying that unrelated things still work after a change; retesting is verifying the specific defect is actually fixed."

---

## Your 2-Minute Deep Answer

> "**The four test levels — scope increasing, isolation decreasing:**
>
> | Level | Scope | Usually owned by |
> |---|---|---|
> | **Unit** | One function or class, dependencies mocked | Developers |
> | **Integration** | Two or more components talking | Developers / SDET |
> | **System** | The whole application end to end | QA / SDET |
> | **Acceptance (UAT)** | Does it meet business needs | Business / product |
>
> The practical point is the **test pyramid**: many unit tests, fewer integration, fewest end-to-end. Because cost and fragility rise as you go up. That's the same argument I make about pushing validation to the API layer — a UI test costs roughly fifty times an API test on every run.
>
> The failure mode is the **inverted pyramid** — a team with almost everything at the E2E level. Slow, flaky, and expensive to maintain, and it's what happens when QA owns testing entirely and developers own none of it.
>
> ---
>
> **Functional vs non-functional:**
>
> Functional = *what* it does. Does login work, does the discount calculate correctly.
>
> Non-functional = *how well*. How fast, how many concurrent users, how secure, how usable.
>
> Worth saying: non-functional defects are often the more expensive ones. A feature that works but takes eleven seconds to load is a real problem that functional testing passes cleanly.
>
> ---
>
> **Smoke vs Sanity — the classic:**
>
> **Smoke** — shallow and wide. Does the application start, can you log in, do the main screens load? The question it answers is *"is this build worth testing at all?"* Run it first; if it fails, reject the build and don't waste the team's time.
>
> **Sanity** — narrow and deep. A specific area, after a specific change. *"They fixed the discount calculation — does discount now work?"*
>
> **The honest addition:** in practice a lot of teams use these terms interchangeably, and the exact distinction matters less than the operational reality — you want a fast blocking suite that gates the build, and deeper suites that run less often. I'd give the textbook answer and then say that, because it shows you know the definition *and* how it actually plays out.
>
> ---
>
> **Regression vs Retesting — the other classic:**
>
> **Retesting** — verifying the specific defect is fixed. Same steps, same data, on the new build. Only makes sense on a build claiming the fix.
>
> **Regression** — verifying that *other* things still work. Nothing to do with the defect itself; it's about the side effects of changing code.
>
> Two differences worth naming: retesting is targeted and can't be planned in advance (you don't know what will need retesting until something fails), whereas regression is broad and is exactly what you automate. And **retesting comes first** — no point running the full regression if the fix itself doesn't work.
>
> ---
>
> **Other types worth knowing:**
>
> **Exploratory** — simultaneous learning, design, and execution. Not random; it's structured investigation with a charter. It finds a category of bug automation structurally can't, because automation only checks what you thought to tell it.
>
> **Ad-hoc** — genuinely unstructured, no documentation. Occasionally useful, not a strategy.
>
> **End-to-end** — a complete user journey across systems, often including integrations.
>
> **Alpha / Beta** — alpha is internal before release, beta is with real users in their environment.
>
> ---
>
> **The thing I'd add unprompted:**
>
> The most useful classification in practice isn't functional versus non-functional — it's **what level should this test live at**. That decision affects speed, stability, and maintenance cost more than anything else, and it's where most suites go wrong."

---

## The Classic Pairs

| | Smoke | Sanity |
|---|---|---|
| Depth | Shallow | Deep |
| Breadth | Wide | Narrow |
| When | Every build | After a specific fix |
| Question | "Worth testing?" | "Does this fix work?" |

| | Retesting | Regression |
|---|---|---|
| Target | The specific defect | Everything else |
| Planned? | No — reactive | Yes — planned suite |
| Automate? | Rarely | **Yes, primarily** |
| Order | First | After retest passes |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Operationally, the split that matters to me is what gates a build versus what runs on a schedule. A fast smoke suite blocks the pipeline — if login or the main flows are broken, nothing else is worth running. Full regression runs on merge, and the slower cross-browser work runs nightly.
>
> That tiering is the practical version of these categories, and it's what actually saves time — running less more often beats running everything rarely, and it's free compared with adding machines.
>
> On regression versus retesting, the distinction I'd emphasize is that regression is what you automate. Retesting is targeted and reactive — you can't plan it, because you don't know what will need retesting until something fails. Regression is broad, repeated every release, and that repetition is exactly where automation compounds.
>
> And I'd mention that exploratory testing stays in the strategy. Automation only checks what you thought to tell it to check — it structurally cannot find the thing you didn't anticipate. So the two aren't alternatives."

---

## Common Interview Questions & Answers

### Q1: "Difference between smoke and sanity testing?"

**A:** "Smoke is shallow and wide — does the build start, can you log in, do the main screens load. It answers whether the build is worth testing at all. Sanity is narrow and deep — a specific area after a specific fix. In practice a lot of teams use the terms interchangeably; what matters operationally is having a fast blocking suite and deeper suites that run less often."

### Q2: "Regression vs retesting?"

**A:** "Retesting verifies the specific defect is fixed — same steps, same data, new build. Regression verifies everything *else* still works after the change. Retesting is reactive and can't be planned; regression is the planned suite, and it's what you automate because it repeats every release. Retesting comes first — no point running full regression if the fix doesn't work."

### Q3: "What are the levels of testing?"

**A:** "Unit, integration, system, acceptance — scope increasing, isolation decreasing. The practical point is the test pyramid: most tests at the unit level, fewest end-to-end, because cost and fragility rise as you go up. The failure mode is an inverted pyramid where everything is E2E — slow, flaky, and expensive."

### Q4: "Functional vs non-functional testing?"

**A:** "Functional is what the system does — does login work, does the calculation come out right. Non-functional is how well — speed, load capacity, security, usability. Worth noting non-functional defects are often more expensive: a feature that works but takes eleven seconds to load passes functional testing cleanly and is still a real problem."

### Q5: "Is exploratory testing still relevant with automation?"

**A:** "Yes, and they're not alternatives. Automation checks what you told it to check — it can't find something you didn't anticipate. Exploratory testing finds that category, and it's structured investigation with a charter rather than random clicking. I'd keep it in the strategy alongside automation, not instead of it."

---

## Weak Areas to Watch

❌ **Don't hesitate on smoke vs sanity**
→ It's a warm-up. Fumbling it sets a bad tone early.

❌ **Don't say regression means "re-running the failed test"**
→ That's retesting. Common confusion.

❌ **Don't ignore the pyramid**
→ Test level is the highest-leverage decision in a suite.

❌ **Don't dismiss exploratory testing**
→ It finds what automation structurally can't.

---

## Key Takeaway for Interview

These are warm-ups. Answer cleanly and quickly, then add the operational reality.

The pair to have automatic:

> **"Smoke asks whether the build is worth testing. Sanity asks whether one specific fix works."**
> **"Retesting checks the defect is fixed. Regression checks everything else still works."**

And the addition that shows judgement:

> **"The most useful classification isn't functional versus non-functional — it's what level a test should live at. That decision drives speed, stability, and maintenance cost more than anything else."**

---

## Checklist

✅ Smoke vs sanity — instant, clean answer
✅ Regression vs retesting — instant, and know which you automate
✅ Four levels, and the pyramid rationale
✅ Functional vs non-functional, with why NFR defects are expensive
✅ Exploratory positioned as complementary, not obsolete
✅ Can connect level choice to cost and stability

---

**Remember:** Warm-up questions are cheap to answer and expensive to fumble. 🎯
