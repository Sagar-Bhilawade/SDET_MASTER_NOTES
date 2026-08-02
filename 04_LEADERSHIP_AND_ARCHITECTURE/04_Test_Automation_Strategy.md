# Leadership - Test Automation Strategy: What to Automate & Why

## What This Topic Covers

Deciding **what deserves a test** — and being able to defend that to an engineering manager who's asking why automation costs what it costs.

Most SDETs can automate a given scenario. Fewer can explain why that scenario is worth automating and another isn't. That judgement is what leadership roles are actually buying.

---

## Why It Matters (Interview Context)

The questions:

> "How do you decide what to automate?"
> "You have limited time. What gets tested?"
> "How do you know your automation is worth the investment?"

The wrong instinct is "automate everything." It sounds thorough and it's strategically naive — every test is a liability as well as an asset, because every test has to be maintained forever.

---

## Your 30-Second Interview Answer

> "I automate based on risk and repetition — what's expensive if it breaks, and what gets verified over and over. High-value, stable, frequently-run scenarios are the clear yes. What I deliberately don't automate is anything highly volatile, one-off, or subjective — exploratory testing and visual judgement are still better done by a person. And I push validation to the cheapest level that can catch the problem, because a UI test that could have been an API test costs ten times more forever."

---

## Your 2-Minute Deep Answer

> "The frame I use is that a test is an **asset and a liability**. It catches bugs, and it also has to be maintained for as long as it exists. So the question isn't 'can this be automated' — almost anything can — it's whether the return justifies the ongoing cost.
>
> ---
>
> **WHAT I AUTOMATE FIRST**
>
> **Revenue and trust paths.** Login, checkout, payment, data submission. Anything where a failure costs money or credibility. These get automated first and maintained best, regardless of how stable they are.
>
> **High-repetition regression.** Anything a human would otherwise verify every release. That's where automation compounds — the same test running two hundred times.
>
> **Historically broken areas.** If a module has broken three times, it'll break again. Bug-driven test writing is one of the highest-return activities available, because you have evidence of actual risk rather than a guess.
>
> **Things humans are bad at.** Precise numeric validation, large data comparison, cross-browser consistency. Machines are simply better here.
>
> ---
>
> **WHAT I DELIBERATELY DON'T AUTOMATE**
>
> **Volatile UI.** A feature being actively redesigned will break tests weekly and each break is noise. Wait for it to settle. Automating a moving target is a commitment to rewriting.
>
> **One-time verifications.** A migration check that runs once doesn't earn the automation cost.
>
> **Subjective judgement.** Does this layout look right, does this flow feel confusing. Automation checks what you told it to check — it can't notice that something is wrong in a way you didn't anticipate.
>
> **Exploratory testing.** Not automatable by definition, and it finds a category of bug automation structurally cannot. This is worth saying out loud in an interview, because it shows you see automation as part of a strategy rather than the whole of it.
>
> ---
>
> **THE TEST LEVEL DECISION — the highest-leverage call**
>
> Before automating anything, I ask what's the cheapest level that catches this.
>
> ```
> Unit test:        ~10ms    cheapest, most stable
> API test:        ~200ms    fast, stable, no browser
> UI test:          ~10s     slow, more fragile, but tests the real thing
> ```
>
> A UI test costs roughly fifty times what an API test costs, every single run, forever — plus it's more fragile to maintain.
>
> So: business logic, validation rules, calculations, permissions → API level. Reserve UI tests for what genuinely needs a browser: rendering, interaction, integration across screens, the critical journeys end to end.
>
> This one decision affects suite speed, stability, and maintenance cost more than almost anything else, and it's the answer most candidates don't give.
>
> ---
>
> **HOW I'D EXPLAIN THE INVESTMENT TO A MANAGER**
>
> Not in test counts or coverage percentages — neither means anything outside QA. In their terms:
>
> - **Escapes prevented** — bugs caught before production
> - **Feedback speed** — how fast a developer learns they broke something
> - **Manual effort displaced** — regression cycles no longer done by hand
> - **Maintenance cost** — what the suite costs to keep, stated honestly
>
> That last one matters for credibility. A pitch that presents automation as free doesn't survive contact with reality, and presenting the cost yourself is what makes the rest of the numbers believable.
>
> ---
>
> **The strategic point:** automation isn't a goal, it's a means of managing risk at a cost. Two hundred well-chosen tests beat a thousand indiscriminate ones — fewer to maintain, faster to run, and every failure means something."

---

## The Automation Decision Matrix

| Scenario | Automate? | Why |
|----------|:---:|-----|
| Login / checkout / payment | ✅ First | High cost of failure |
| Regression run every release | ✅ Yes | Repetition compounds return |
| Area that's broken repeatedly | ✅ Yes | Evidenced risk |
| Numeric / data-heavy validation | ✅ Yes | Humans are worse at it |
| Feature under active redesign | ❌ Wait | Constant rewrite cost |
| One-time migration check | ❌ No | No repetition to amortize |
| "Does this look right?" | ❌ No | Subjective judgement |
| Exploratory testing | ❌ No | Finds what you didn't predict |

---

## The Level Decision

| Validation | Right Level | Cost per run |
|-----------|------------|---|
| Field validation rules | API | ~200ms |
| Calculation correctness | API / Unit | ~10-200ms |
| Permission logic | API | ~200ms |
| Page renders correctly | UI | ~10s |
| Multi-screen user journey | UI | ~10s |
| Cross-browser consistency | UI | ~10s |

**The rule:** push it down until it can't go lower.

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Our suite covers 300+ scenarios with 90%+ regression coverage, and the thing I'd emphasize is that the number is a result rather than a target. Coverage as a goal produces low-value tests that inflate maintenance cost without reducing risk.
>
> The prioritization is risk and repetition. Critical business flows first — those get automated first and maintained best. Then anything verified every release, because that's where automation compounds. And I treat production bugs as direct input: if something broke once, it's evidenced risk rather than speculation, and it earns a test.
>
> The decision I'd call out as highest-leverage is test level. A meaningful share of what teams write as UI tests are validating logic with no UI dependency — validation rules, calculations, permissions. Those run in a fraction of the time as API tests and are far more stable. A UI test costs roughly fifty times an API test on every run, forever. Choosing the level well does more for suite speed and stability than any amount of optimization afterwards.
>
> And I'd be clear that automation isn't the whole strategy. Exploratory testing finds a category of problem automation structurally can't, because automation only checks what you thought to tell it."

---

## Common Interview Questions & Answers

### Q1: "How do you decide what to automate?"

**A:** "Risk and repetition. What's expensive if it breaks, and what gets verified repeatedly — that's where automation compounds. I also treat production bugs as direct input, since a module that's broken before is evidenced risk rather than a guess. And before automating anything I ask what's the cheapest level that catches it, because a UI test that could have been an API test costs fifty times more on every run, forever."

### Q2: "Why not automate everything?"

**A:** "Because every test is a liability as well as an asset — it has to be maintained for as long as it exists. Two hundred well-chosen tests beat a thousand indiscriminate ones: fewer to maintain, faster to run, and every failure means something. Automating a volatile feature is worse than not automating it, because you get constant noise and the team learns to ignore red."

### Q3: "Your manager wants 100% automation coverage. How do you respond?"

**A:** "I'd want to understand what problem they're solving — usually it's 'bugs are reaching production' or 'releases feel risky', and coverage percentage is a proxy they've reached for. Those have better answers. I'd redirect to escape rate, since that measures whether we're catching what matters, and point out that chasing a coverage number produces low-value tests that increase maintenance cost without reducing risk. I'd also be honest that some things shouldn't be automated at all — exploratory and subjective checks."

### Q4: "How do you justify automation spend?"

**A:** "In their terms, not QA terms. Escapes prevented, feedback speed, manual regression displaced — and the maintenance cost, stated honestly. Including the cost is what makes the rest credible. A pitch that presents automation as free falls apart the first time someone asks why the team spent a sprint fixing tests."

### Q5: "Limited time before a release. What do you test?"

**A:** "Revenue and trust paths first — login, checkout, payment, whatever moves money or data. Then whatever changed in this release, plus anything historically fragile near it. What I wouldn't do is run the full suite for completeness if time is short; I'd rather have high confidence in the critical paths than shallow confidence across everything. And I'd say plainly what wasn't covered, because an unstated gap is worse than a known one."

---

## Weak Areas to Watch

❌ **Don't say "automate everything"**
→ Strategically naive. Every test is a maintenance liability.

❌ **Don't lead with coverage percentage**
→ It's a proxy metric that drives the wrong behaviour.

❌ **Don't skip the test-level question**
→ It's the highest-leverage decision and most candidates miss it.

❌ **Don't dismiss manual and exploratory testing**
→ It finds what automation structurally cannot.

❌ **Don't present automation as free**
→ Naming the cost is what makes your numbers credible.

---

## Key Takeaway for Interview

Strategy is about **what not to build** as much as what to build.

The framing that lands:

> **"Every test is an asset and a liability. It catches bugs and it has to be maintained forever. Two hundred well-chosen tests beat a thousand indiscriminate ones."**

And the highest-leverage insight:

> **"Before automating anything, I ask what's the cheapest level that catches it. A UI test costs about fifty times an API test on every run, forever — that one decision does more for suite speed and stability than any optimization afterwards."**

---

## Strategy Checklist

✅ Prioritized by risk and repetition, not by coverage target
✅ Critical business paths automated first, maintained best
✅ Production bugs converted into tests
✅ Volatile features deliberately left alone until stable
✅ Every test pushed to the cheapest level that catches the problem
✅ Exploratory testing retained as part of the strategy
✅ Value expressed as escapes, feedback speed, and effort displaced
✅ Maintenance cost stated honestly

---

**Remember:** The tests you choose not to write are as much a decision as the ones you do. 🎯
