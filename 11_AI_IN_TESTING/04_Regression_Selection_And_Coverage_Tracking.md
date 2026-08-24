# AI In Testing - Regression Case Selection And Coverage Tracking

## What it is

Two related problems AI can help with:

1. **Regression selection** — given a change, which existing tests should run? Running
   the whole suite every time is slow; running too few misses regressions. AI helps map
   a change to its likely blast radius so the selection is risk-based, not guesswork.
2. **Coverage tracking** — keeping an honest, current map of what's automated, what's
   still manual, and what should be automated next. AI helps compare requirements and
   manual flows against the existing automated suite and surface the gaps.

The honest framing: AI helps you *reason about* coverage and impact faster. It does not
give you real code-coverage numbers or a guaranteed-correct impact analysis — those
come from instrumentation and from understanding the system.

## Why it matters

Regression selection is a senior conversation. A junior runs everything or runs a fixed
smoke set; an engineer selects based on what changed and can defend the selection. AI
lowers the cost of that reasoning, so interviewers want to know you use it to *sharpen*
selection, not to blindly trust a tool's "these tests are affected" list.

Coverage tracking is where teams quietly rot. Manual flows never get automated, the
tracker drifts, and nobody knows the real state. Showing you keep an honest coverage map
— and use AI to keep it current — is a maturity signal.

## 30-second answer

> "For regression, I use AI to map a change to its likely blast radius so I select tests
> by risk instead of running everything or running a fixed set. For coverage, I use it to
> compare requirements and manual flows against what's already automated and surface the
> gaps. It sharpens the reasoning; the final selection and the priority are still mine,
> because a wrong impact call means a missed regression."

## 2-minute answer

> "I split this into impact and inventory.
>
> On impact — regression selection — the question is always 'what could this change
> break?'. AI helps by reading the change and suggesting the affected flows: if a shared
> utility or an API contract changed, it flags the wider surface. That turns selection
> from a guess into a risk-based short list. But I never run *only* what the tool
> suggests for a risky change — an AI impact call can miss an indirect dependency, and a
> missed regression is the most expensive kind of miss. So for high-risk changes I widen
> the net deliberately; AI narrows my thinking, it doesn't get the final word.
>
> On inventory — coverage tracking — I keep a map of automated vs manual vs
> to-be-automated. AI helps maintain it by comparing new requirements and manual flows
> against the existing suite and listing what's not covered yet. That gives a
> prioritized backlog of automation candidates instead of a stale spreadsheet.
>
> The trade-off I'm honest about: this is *reasoning support*, not real coverage
> measurement. True code coverage needs instrumentation; AI gives me a fast, defensible
> map of risk and gaps, which is what actually drives day-to-day selection — but I don't
> present its output as a hard coverage number."

## Real project example

> "On the Playwright track the regression suite is large enough that running all of it
> on every change is wasteful, so selection matters. When a change comes in, I use an
> AI-assisted read of the diff to flag the impacted flows, then pick the regression set
> around that — plus a fixed smoke core that always runs. For anything touching shared
> code I widen it on purpose. Alongside that I keep a coverage map: what's automated,
> what's still manual, what's next. AI helps me keep that list current by comparing new
> manual flows against the suite, so the automation backlog reflects reality instead of
> drifting. The CI feedback loop went from 40 to 12 minutes partly because selection got
> tighter — you run the right tests, not all of them."

## Common interview Q&A

### Q1: How do you decide what regression to run?
**A:** "By risk. I use an AI-assisted read of the change to flag the likely blast
radius, run a fixed smoke core plus the impacted flows, and widen the net for anything
touching shared code. Selection follows the change, not a fixed script."

### Q2: Can you trust AI's impact analysis?
**A:** "As a narrowing aid, yes; as the final word, no. It can miss an indirect
dependency, and a missed regression is the costliest miss. So for risky changes I
deliberately test wider than it suggests."

### Q3: How do you track coverage?
**A:** "A living map of automated vs manual vs to-be-automated. AI helps keep it current
by comparing new requirements and manual flows against the existing suite and surfacing
gaps, which gives a prioritized automation backlog instead of a stale tracker."

### Q4: Is that real code coverage?
**A:** "No, and I'm careful not to claim it is. Real coverage needs instrumentation.
This is risk-and-gap reasoning — it drives day-to-day selection and the automation
backlog, but it's not a hard percentage."

## Weak areas to watch

- Don't call AI impact analysis "code coverage" — they're different, and the distinction
  is a maturity tell.
- Don't run only what the tool flags on a risky change; widen deliberately.
- Don't let the coverage map drift — its value is in being current.
- Don't cite a coverage number you can't defend; use the resume's 90% regression figure
  only as it's stated.

## Key takeaway

> "AI turns regression selection from guessing into risk-based reasoning and keeps the
> coverage map honest — but for a risky change I widen the net myself, because a missed
> regression is the miss that costs the most."
