# AI In Testing - PR And Jira Story Analysis

## What it is

Using AI to turn a change (a Jira story or a pull request) into a QA starting point:
a summary of what changed, the impacted areas, the risk, the open questions, and the
first list of scenarios worth testing.

The input is a requirement or a diff. The output is a QA analysis draft — not a
verdict. The tester still decides what actually matters.

Two distinct inputs, two slightly different jobs:

- **Jira story analysis** works from *intended behavior* — acceptance criteria, the
  business goal, what "done" means.
- **PR analysis** works from *actual change* — the diff, the files touched, the
  functions modified, what the code now does differently.

The strong tester uses both together: the story tells you what should happen, the PR
tells you what the developer actually changed, and the gap between them is where bugs
hide.

## Why it matters

This is the most common real use of AI in a QA day, so interviewers probe it to see
if your usage is genuine or buzzword-deep. A candidate who has actually done this can
describe the *shape* of the analysis; one who hasn't will stay vague.

It also signals seniority. Reading a PR for test impact — not just running the story's
happy path — is the difference between a scripted tester and an engineer who
understands the system. AI lowers the cost of that reading, so you can do it on every
change instead of only the big ones.

## 30-second answer

> "For a story or a PR, I use AI to draft the first analysis fast: what changed, which
> areas are impacted, the likely risk, and a starting scenario list. Then I correct it.
> The story tells me intended behavior, the PR tells me what actually changed, and I
> test the gap between them. AI gets me to that gap faster; it doesn't decide the risk
> for me."

## 2-minute answer

> "I treat the story and the PR as two different sources.
>
> From a Jira story, AI helps me expand acceptance criteria into scenarios — happy
> path, negative paths, boundary values, and the questions that aren't answered yet.
> That last part matters most: a good draft surfaces *ambiguity*, like 'what happens if
> this field is empty' or 'is this permission-gated', which I then take back to the BA
> or developer before I write a single test.
>
> From a PR, AI summarizes the diff: which files changed, which functions, and what the
> behavior change likely is. That points my regression thinking — if a shared utility or
> an API contract changed, I know the blast radius is wider than the story implies.
>
> The value is that I can do this on *every* change cheaply, so nothing ships without a
> QA read. But I keep a hard boundary: AI describes what the code appears to do, it
> doesn't confirm what it *actually* does. A diff summary can be confidently wrong about
> intent. So I validate the real behavior myself and I never let the summary replace
> reading the important parts of the diff. The analysis is a draft that saves me the
> blank-page cost, not a substitute for understanding the change."

## Real project example

> "On the Playwright track, work comes in as stories plus the developer's PR. Before I
> plan tests, I get a quick AI-assisted read: a summary of the change, the impacted
> flows, and a first scenario list. On one change to a shared component, the story only
> described one screen, but the PR touched a utility used in several places — the
> analysis flagged the wider surface, so I pulled in regression for the other screens
> instead of testing only the one the story named. That's the payoff: catching blast
> radius the story didn't mention. I still read the diff myself for anything risky; the
> AI read is the fast first pass, not the final call."

## Common interview Q&A

### Q1: How do you use AI to analyze a story or PR?
**A:** "I generate a first analysis — change summary, impacted areas, risk, and a
starting scenario list — then correct it. The story gives intended behavior, the PR
gives the actual change, and I focus my testing on the gap between them."

### Q2: What's the risk of trusting an AI PR summary?
**A:** "It can be confidently wrong about intent. A diff summary describes what the code
looks like it does, not what it actually does under real data and edge cases. So I use
it to point my attention, then validate the behavior myself."

### Q3: What's the single most useful output?
**A:** "The open questions. A good analysis surfaces ambiguity in the requirement —
empty states, permissions, error handling — that I take back to the developer or BA
before writing tests. Catching a spec gap early is cheaper than catching it in QA."

### Q4: How does PR analysis change your regression scope?
**A:** "If the diff touches a shared utility or an API contract, the blast radius is
wider than the story suggests. The PR read tells me which other flows to re-check, so
regression follows the actual change, not just the story title."

## Weak areas to watch

- Don't paste real Jira text, PR diffs, or internal code into an interview answer —
  describe the *workflow*, not the content.
- Don't claim the AI "understands requirements." It drafts; you confirm.
- Don't let a summary replace reading the risky part of a diff yourself.
- Don't treat the scenario list as complete — its job is to remove the blank page, not
  to define coverage.

## Key takeaway

> "The story says what should happen; the PR says what actually changed. AI gets me to
> the gap between them faster — but confirming the real behavior and the real risk is
> still mine."
