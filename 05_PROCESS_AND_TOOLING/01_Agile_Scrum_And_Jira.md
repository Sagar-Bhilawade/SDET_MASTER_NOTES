# Agile, Scrum & Jira

## What This Covers

How QA fits into a sprint — ceremonies, definition of done, and the Jira workflow around it.

---

## Why It Matters (Interview Context)

Agile/Scrum and Jira are both on your resume, and process questions appear in almost every loop — often from the hiring manager rather than the technical panel.

What they're really assessing: **do you work with the team, or after it?** The weak answer describes QA as a phase at the end of the sprint. The strong answer describes QA involved from refinement onward.

---

## Your 30-Second Interview Answer

> "In a two-week sprint I'm involved from refinement — that's where testability questions get asked, before the code exists. During the sprint I automate alongside development rather than waiting for a handoff, and I keep automation of a story in the same sprint where possible so it doesn't accumulate as debt. The thing I'd push for in definition of done is that a story isn't done until its automation exists, otherwise the automation backlog grows every sprint and never gets paid down."

---

## Your 2-Minute Deep Answer

> "**The ceremonies, and where QA adds value in each:**
>
> **Backlog refinement / grooming** — the most valuable one for QA, and the one people skip.
>
> This is where you ask the questions that are cheap now and expensive later:
> - What should happen when this fails?
> - What are the boundary values?
> - Is this testable as written, or is the acceptance criteria untestable?
>
> Building a decision table during refinement routinely surfaces a combination nobody specified. That's a defect prevented before any code exists — far cheaper than finding it in test.
>
> **Sprint planning** — sizing includes the testing effort, not just development. If automation isn't in the estimate, it doesn't happen.
>
> **Daily standup** — blockers surfaced early. "The staging environment has been down since yesterday" belongs here, not in a status report on day nine.
>
> **Sprint review / demo** — QA often demos, because you know the edge cases and can show the feature honestly rather than only the happy path.
>
> **Retrospective** — where process problems get raised. Flaky pipelines, environment instability, stories arriving on the last day — these are retro topics, and raising them with data rather than complaint is what makes them actionable.
>
> ---
>
> **The problem worth naming: the mini-waterfall**
>
> The common failure in "agile" teams is that development takes days 1-8 and testing gets days 9-10. That's a waterfall inside a sprint. The symptoms are familiar — testing is always compressed, defects found late, and either the sprint slips or quality does.
>
> What I'd push for:
> - Testing starts as soon as a slice is available, not when the story is complete
> - Automation written alongside development, not after
> - Definition of done includes automation, so it can't be deferred silently
>
> That last one matters most. If a story can be "done" without its automated test, the automation backlog grows every sprint and never gets paid down. Making it part of DoD is the structural fix.
>
> ---
>
> **Definition of Done — what I'd argue for:**
>
> ```
> ✅ Code complete and peer reviewed
> ✅ Unit tests written and passing
> ✅ Acceptance criteria verified
> ✅ Automated tests written and passing in CI
> ✅ No known open defects above agreed severity
> ✅ Documentation updated where relevant
> ```
>
> ---
>
> **Jira in practice:**
>
> Issue types — Epic → Story → Sub-task, plus Bug and Task.
>
> The workflow mirrors the defect lifecycle: To Do → In Progress → In Review → In Testing → Done.
>
> What I'd actually use it for:
> - **Linking tests to stories** so traceability exists — which story does this test cover
> - **Linking defects to the story that introduced them** — that's how you spot a module that keeps breaking
> - **JQL** for the queries that matter: open bugs by severity, defects found this sprint, reopened count
>
> A JQL example worth knowing:
> ```
> project = ABC AND issuetype = Bug AND status != Done AND priority in (Highest, High)
> ORDER BY created DESC
> ```
>
> ---
>
> **The metric I'd bring to a retro:**
>
> Defect escape rate — bugs found in production versus found before release. That's the number that tells you whether the process is working, and it's more useful than counting how many defects QA logged, which measures activity rather than outcome.
>
> Same principle as test suite metrics: measure the outcome, not the effort."

---

## Ceremonies & QA Value

| Ceremony | QA contribution |
|----------|----------------|
| **Refinement** | Testability questions, edge cases, decision tables — highest value |
| **Planning** | Testing effort in the estimate |
| **Standup** | Environment and blocker visibility |
| **Review** | Honest demo including edge cases |
| **Retro** | Process problems raised with data |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "The ceremony where I add most value is refinement, because that's where questions are cheapest. Asking what should happen on failure, what the boundary values are, and whether the acceptance criteria are actually testable — those find gaps before any code exists. Building a decision table during refinement routinely surfaces a combination nobody specified, and that's a defect prevented rather than found.
>
> The problem I'd name is the mini-waterfall — development takes most of the sprint and testing gets the last two days. It's technically agile and functionally waterfall, and the symptoms are always the same: compressed testing, late defects, and either the sprint slips or quality does.
>
> The structural fix I'd push for is automation in the definition of done. If a story can be marked done without its automated test, the automation backlog grows every sprint and never gets paid down — you end up with a permanent debt nobody scheduled. Making it part of DoD is what stops that, and it's a conversation with the team rather than something QA can impose.
>
> On Jira, the part I care about is linking defects back to the story that introduced them. That's how you spot a module that keeps breaking, which is evidenced risk worth automating against rather than a guess."

---

## Common Interview Questions & Answers

### Q1: "How does QA fit into your sprint?"

**A:** "From refinement onward, not at the end. Refinement is where I ask testability questions and surface edge cases before code exists. During the sprint I automate alongside development rather than waiting for a handoff. And I'd push for automation being part of the definition of done — otherwise it defers silently and the backlog grows every sprint."

### Q2: "What's the biggest problem you've seen with agile in practice?"

**A:** "The mini-waterfall — development takes days one to eight, testing gets nine and ten. It's agile in name and waterfall in shape. Testing is always compressed, defects arrive late, and either the sprint slips or quality does. The fix is testing slices as they become available rather than waiting for story completion, and putting automation in the DoD so it can't be quietly deferred."

### Q3: "What would you include in a definition of done?"

**A:** "Code complete and reviewed, unit tests passing, acceptance criteria verified, automated tests written and passing in CI, no open defects above agreed severity. The automation line is the one I'd argue hardest for, because without it automation becomes permanent debt that nobody schedules."

### Q4: "How do you use Jira?"

**A:** "Standard workflow — To Do, In Progress, In Review, In Testing, Done. What I actually get value from is linking defects to the story that introduced them, because that shows you which modules keep breaking, and that's evidenced risk worth targeting. Plus JQL queries for open bugs by severity and reopen counts."

### Q5: "What metric would you bring to a retrospective?"

**A:** "Defect escape rate — bugs found in production versus caught before release. That measures whether the process is working. Counting how many defects QA logged measures activity, not outcome, and it can even reward the wrong behaviour."

---

## Weak Areas to Watch

❌ **Don't describe QA as a phase at the end**
→ Signals you work after the team, not with it.

❌ **Don't skip refinement as the key ceremony**
→ It's where QA prevents rather than finds.

❌ **Don't accept automation outside the DoD**
→ That's how permanent automation debt forms.

❌ **Don't cite "defects logged" as a metric**
→ Measures activity. Escape rate measures outcome.

---

## Key Takeaway for Interview

Process questions test whether you work *with* the team.

The problem worth naming:

> **"The mini-waterfall — development takes eight days, testing gets two. It's agile in name and waterfall in shape, and testing is compressed every single sprint."**

And the structural fix:

> **"Automation belongs in the definition of done. If a story can be done without its automated test, the automation backlog grows every sprint and nobody ever schedules paying it down."**

---

## Checklist

✅ Refinement named as the highest-value ceremony
✅ Testability questions asked before code exists
✅ Testing effort included in estimates
✅ Automation part of the definition of done
✅ Mini-waterfall recognized and addressed
✅ Defects linked to originating stories in Jira
✅ Escape rate used as the process metric

---

**Remember:** Finding a defect is good. Preventing it at refinement is cheaper. 🎯
