# AI In Testing - Context Management For QA Agents

## What it is

Context management means deciding what information an agent needs, where that information
comes from, how much of it to load, and how to keep it relevant while the agent works.

For QA agents, context can include:

- Jira story or acceptance criteria
- PR diff
- changed files
- existing test files
- framework conventions
- locator strategy
- API contracts
- test data rules
- previous defects
- review gates

The goal is not to give the agent everything. The goal is to give it the right context for
the current task.

## Why it matters

Interviewers ask this because "agentic workflow" sounds nice, but without context management
the agent gives generic or wrong answers.

The strong answer is:

> "I separate long-term instructions from task-specific context. Skills hold stable rules,
> while the agent loads only the files, PR changes, story details, and notes needed for the
> current task."

## 30-second answer

> "I manage agent context in layers. Skills contain stable instructions like QA standards,
> Playwright conventions, locator strategy, and review gates. For each task, the agent loads
> only the relevant Jira story, PR diff, changed files, existing tests, and framework files.
> Then it produces a focused output like test plan, regression impact, or automation draft.
> This keeps the agent specific instead of generic and avoids flooding it with unnecessary
> context."

## 2-minute answer

> "For my QA Knowledge Agent and QA Automation Agent, I do not depend on one giant prompt.
> I manage context in layers.
>
> First is stable context: project rules, naming conventions, folder structure, locator
> strategy, test-data rules, and red lines. Those live in skills or instruction files because
> they don't change every task.
>
> Second is task context: Jira story, acceptance criteria, PR diff, changed files, existing
> tests, API contracts, or the manual flow. This is loaded only when needed.
>
> Third is working context: the agent's current plan, assumptions, generated cases, and review
> notes. This should stay small and connected to the task.
>
> The practical rule is: QA Knowledge Agent needs story/PR/testing context. QA Automation
> Agent needs framework/code/locator/context. They should share the result, not blindly share
> every file."

## QA Knowledge Agent context

Needs:

- story summary
- acceptance criteria
- PR diff or changed areas
- product behavior notes
- existing manual/regression cases
- bug history if available
- QA test design rules

Produces:

- impacted areas
- open questions
- functional cases
- regression cases
- automation candidates
- bug-drafting inputs

## QA Automation Agent context

Needs:

- reviewed manual flow
- existing Playwright folder structure
- page objects
- fixture patterns
- locator conventions
- API helper patterns
- test data strategy
- review gates

Produces:

- Playwright draft script
- page object method suggestions
- fixture suggestions
- smoke test draft
- code review notes

## Skills vs task context

| Type | What it contains | Example |
|---|---|---|
| Skill context | Stable rules | prefer semantic locators, do not hardcode secrets |
| Task context | Current work item | Jira story, PR diff, manual flow |
| Code context | Existing implementation | page object, fixture, helper files |
| Review context | Quality gate | assertion, locator, data, cleanup, test level |

Interview line:

> "Skills tell the agent how to work. Task context tells it what to work on."

## Context handoff between agents

Good handoff:

```text
QA Knowledge Agent
  -> reviewed manual flows
  -> automation candidates
  -> priority and risk

QA Automation Agent
  -> Playwright draft
  -> page object methods
  -> fixture/data suggestions
  -> review checklist
```

Do not hand off:

- raw large unrelated files
- entire workspace
- sensitive internal details not needed
- unreviewed hallucinated assumptions

Strong line:

> "The Knowledge Agent should hand off a reviewed flow and risk priority, not a dump of
> everything it read."

## How to avoid context problems

- load only relevant files
- state assumptions explicitly
- keep acceptance criteria visible
- compare PR change against story intent
- ask for missing requirement only when needed
- summarize before generating
- review generated output against source context
- keep secrets and internal sensitive data out

## Common interview Q&A

### Q1: "How do you manage context between agents and skills?"

**A:** "Skills hold stable instructions, like QA rules and framework conventions. Agents load
task-specific context, like Jira story, PR diff, existing tests, or page objects. The output
of one agent becomes a reviewed handoff to the next, not a raw dump."

### Q2: "How do you stop the agent from giving generic answers?"

**A:** "I provide grounded context: acceptance criteria, changed files, existing tests, and
framework rules. Then I ask it to map output back to those sources. Generic output usually
happens when the agent has no grounded project context."

### Q3: "How do two agents collaborate?"

**A:** "QA Knowledge Agent analyzes story/PR and produces reviewed flows and automation
candidates. QA Automation Agent takes those reviewed flows plus framework context and drafts
Playwright automation. I review before anything becomes final."

### Q4: "What is the biggest risk in context management?"

**A:** "Too much irrelevant context or missing critical context. Too much context makes output
noisy; missing context makes it confidently wrong. So I keep context scoped to the task."

## Weak areas to watch

- Do not say "I give all files to the agent."
- Do not say skills and agents are the same.
- Do not let one agent's unreviewed assumptions become another agent's facts.
- Do not include secrets or private internal material unnecessarily.

## Key takeaway

> "Good agent workflow is mostly good context management: stable rules in skills,
> task-specific files for the current work, and reviewed handoffs between agents."
