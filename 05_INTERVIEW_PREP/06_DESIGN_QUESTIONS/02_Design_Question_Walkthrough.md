# Interview Masterclass - The Design Question: A Full Worked Example

## What This Topic Covers

The open-ended design question — the one that most often decides a senior offer.

> "You're joining a team with no test automation. The product is a web app with a REST API. Walk me through how you'd build the test strategy and framework."

There's no single right answer. They're watching your **process**: do you gather requirements, or start coding? Do you name trade-offs, or present a design as obviously correct? Do you sequence delivery, or describe an end state you'd take a year to reach?

---

## The Shape of a Good Answer

```
1. Clarify         (2 min)  — ask before designing
2. State strategy  (2 min)  — what gets tested, at what level
3. Design framework(4 min)  — structure, with reasons
4. Sequence it     (2 min)  — what ships week 1 vs month 3
5. Name risks      (1 min)  — what could go wrong
```

**The most common failure is skipping step 1.** Candidates hear "design a framework" and start listing folders. Interviewers are specifically watching whether you ask questions — because in real work, someone who builds without gathering context is expensive.

---

## STEP 1: Clarify (ask 4-5, not 20)

> "Before I design anything, a few questions:
>
> **What's the application?** Modern SPA, server-rendered, how much of the value is in the UI versus the API?
>
> **What's the team?** How many engineers, what languages are they comfortable in, is there anyone who'll own this after I build it?
>
> **What's the current pain?** Bugs reaching production, slow release cycles, manual regression eating a week? That changes what I build first.
>
> **What's the deployment cadence?** Continuous, or a release every six weeks? That determines how fast feedback has to be.
>
> **Is there existing CI?** Or am I building that too?"

Then — and this matters — **make assumptions explicitly and move on:**

> "I'll assume a React SPA with a REST backend, a team of five who are JavaScript-comfortable, releasing weekly, with GitHub Actions already in place. Tell me if any of that changes the answer."

This shows you can operate with incomplete information rather than being blocked by it, which is exactly the real-world skill.

---

## STEP 2: Strategy Before Framework

> "Before the framework, what we're testing and at what level — because that decision drives everything else.
>
> **The level decision:** validation rules, calculations, permissions — API tests. They run in about 200ms, they're stable, no browser involved. UI tests are reserved for what genuinely needs a browser: rendering, interaction, and the critical end-to-end journeys.
>
> That matters because a UI test costs roughly fifty times an API test on every run, forever, and it's more fragile. Getting the level right up front does more for suite speed and stability than any optimization afterwards.
>
> **What gets covered first:** revenue and trust paths. Login, checkout, whatever moves money or data. Those get automated first and maintained best.
>
> **What I'd explicitly not automate yet:** anything under active redesign. Automating a moving target means rewriting weekly, and the noise teaches the team to ignore red."

---

## STEP 3: The Framework

> "Playwright with TypeScript, given the team's JavaScript comfort and a modern SPA. Built-in API testing means one framework rather than a separate HTTP client.
>
> **Structure:**
>
> ```
> tests/          feature-based — auth/, checkout/, payments/
> pages/          page objects, one per page
> fixtures/       auth, data seeding, API client
> utils/          factories, helpers
> ```
>
> **The four decisions that shape it:**
>
> **Page objects** — centralize locators so a UI change is one edit rather than fifty. Cost: an indirection layer that can hide what a test actually does. I manage that by keeping page objects to user actions only, not utility grab-bags.
>
> **Fixtures over hooks** — setup reusable across files, lazily evaluated, automatic teardown. Cost: a learning curve; a test signature doing a lot of invisible work.
>
> **Storage state for auth** — login once per run rather than per test. At a few hundred tests that's minutes saved. Cost, and this is the one people miss: most tests then never exercise login, which is the most-used flow in the product. So login gets explicit dedicated coverage, or you've optimized a blind spot into your most critical path.
>
> **API seeding for data** — setup drops from ~15 seconds to under one, and failures attribute correctly. Cost: coupling to API contracts, so I'd put it behind a thin client layer.
>
> **Locator strategy:** role-based first, `data-testid` second, CSS as genuine last resort. And I'd want `data-testid` treated as a contract with the dev team — added when the feature is built, removal understood as a breaking change. That's the organizational half, and it's the half that determines whether the convention survives."

---

## STEP 4: Sequence It

This is where many candidates lose the thread — they describe an end state without saying how to get there.

> "I wouldn't build all of that before shipping anything.
>
> **Week 1:** Playwright installed, CI running, one real test on the critical path — login. The goal is a working pipeline end to end, however thin. That proves the plumbing and gives the team something visible.
>
> **Weeks 2-4:** The critical journeys. Page objects and fixtures emerge from real usage rather than being designed upfront — I'd rather see the same thing three times before abstracting it, because abstracting on the first repetition means guessing at a shape you haven't seen.
>
> **Month 2:** API test layer, storage state, data seeding. By now there's enough volume that the performance work pays back.
>
> **Month 3:** Tiered execution — smoke on every push, full regression on merge, slower suites nightly. Metrics: first-attempt pass rate, execution time trend, escape rate.
>
> **Ongoing:** Production bugs converted into tests. That's evidenced risk rather than guessed risk, and it's the highest-return test-writing activity available."

**Why sequencing matters:** it shows you deliver value incrementally rather than disappearing for a quarter. Every engineering manager has been burned by someone who built a beautiful framework nobody asked for.

---

## STEP 5: Name the Risks

> "Three things I'd watch:
>
> **Adoption.** If only I write tests, this fails the moment I'm on leave. I'd pair early and hand over ownership of an area fast.
>
> **Over-engineering.** The design I described is right at 300 tests and genuinely wrong at 20. Building for scale you don't have is just overhead.
>
> **The test-id conversation.** If developers see it as a QA imposition, the convention erodes. The framing that works is that brittle tests mean noisy pipelines, which block their merges too — it's a shared velocity problem, not a favour."

---

## Variations You Should Be Ready For

**"The team has no automation experience."**
→ Shift heavily toward adoption. Simpler framework, more documentation, one exemplary test to copy, pairing built into the plan. The best framework nobody can maintain is worthless.

**"You have one month, not six."**
→ Critical paths only, smoke suite, CI running. Explicitly state what's deferred — an unstated gap is worse than a known one.

**"There's an existing broken suite."**
→ Don't rewrite by default. Assess first: is it broken architecture or accumulated debt? Rewrites are expensive and usually recreate the same problems. Stabilize the critical paths, quarantine the noise, migrate incrementally.

**"Budget is tight — justify the spend."**
→ Escapes prevented, feedback speed, manual regression displaced. And state the maintenance cost honestly, because a proposal that sounds free doesn't survive contact with reality.

---

## What They're Scoring

| Signal | Weak | Strong |
|--------|------|--------|
| **Requirements** | Starts designing immediately | Asks 4-5, then states assumptions |
| **Strategy** | Jumps to tools | Level decision before framework |
| **Trade-offs** | Design has no downsides | Names cost of each choice |
| **Sequencing** | Describes end state only | Week 1 → month 3 delivery |
| **Risk** | None mentioned | Adoption, over-engineering, org friction |
| **Judgement** | One right answer | "Right at 300 tests, wrong at 20" |

---

## Common Failure Modes

❌ **Designing before asking.** The most common and most costly.

❌ **Listing folder structure as the answer.** That describes a framework; it doesn't demonstrate you designed one.

❌ **No sequencing.** An end state with no path is an unfunded plan.

❌ **No trade-offs.** Instantly reads as theoretical.

❌ **Over-engineering the small case.** Proposing sharding and a full metrics dashboard for a team of three.

❌ **Ignoring people.** Adoption, ownership, and the dev-team relationship are half of whether this succeeds.

---

## Key Takeaway

The design question scores **process, not architecture**.

The habit that matters most:

> **Ask before you design. Then state your assumptions out loud and move — that's the real-world skill they're testing.**

And the line that shows genuine judgement:

> **"This design is right at 300 tests and wrong at 20. Building for scale you don't have is just overhead."**

---

## Design Question Checklist

✅ 4-5 clarifying questions before designing
✅ Assumptions stated explicitly, then move forward
✅ Test-level strategy before framework structure
✅ Each design decision paired with its cost
✅ Delivery sequenced: week 1 → month 3
✅ Adoption and ownership addressed
✅ Risks named unprompted
✅ Scale-appropriateness acknowledged

---

**Remember:** They're not buying your architecture. They're buying how you got to it. 🎯
