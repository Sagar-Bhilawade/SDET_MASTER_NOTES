# AI In Testing - VS Code And GitHub Copilot For SDET

## What it is

GitHub Copilot in VS Code is an AI coding assistant inside the editor. For an SDET, it is
not only for writing code faster. It can help with test design, Playwright and Selenium
drafting, debugging, refactoring, explaining framework code, reviewing flaky tests, and
turning manual testing knowledge into automation-ready steps.

As of the latest official VS Code and GitHub documentation checked on 2026-09-10, the
important Copilot/VS Code surfaces are:

- inline suggestions while coding
- Chat/Ask style Q&A
- Edit style targeted code changes
- Agent mode for task execution with tools
- Plan style thinking before implementation
- custom agents using `.agent.md` files
- prompt files using `.prompt.md` files
- agent skills using a `SKILL.md` folder structure
- MCP servers for external tools and context
- repository instructions using `.github/copilot-instructions.md`
- path-specific instructions using `.github/instructions/**/*.instructions.md`

For Sagar's interviews, the value is not saying "I use Copilot". The value is explaining
how he gives context, chooses the right mode, reviews output, and keeps AI-generated test
code maintainable.

Sources checked:

- VS Code custom agents: https://code.visualstudio.com/docs/agent-customization/custom-agents
- VS Code prompt files: https://code.visualstudio.com/docs/agent-customization/prompt-files
- VS Code agent skills: https://code.visualstudio.com/docs/agent-customization/agent-skills
- GitHub Copilot customization: https://docs.github.com/en/copilot/reference/customization-cheat-sheet
- GitHub custom instructions support: https://docs.github.com/en/copilot/reference/custom-instructions-support
- GitHub MCP concept: https://docs.github.com/en/copilot/concepts/context/mcp

## Why it matters in interviews

Interviewers ask Copilot questions because many candidates only use autocomplete. A stronger
SDET shows that Copilot is part of a controlled QA workflow.

Weak answer:

> "I use Copilot to generate Playwright scripts."

Strong answer:

> "I use Copilot in VS Code in different modes. Ask mode is useful for understanding code or
> debugging an error. Edit mode is useful for targeted changes in one file. Agent mode is
> useful when the task needs repo search, file edits, test execution, and iteration. For QA,
> I give it story context, existing framework style, locator strategy, fixtures, test data
> rules, and expected assertions. I treat the output as a draft and review it before commit."

What they are checking:

| Question | What they really check |
|---|---|
| "How do you use Copilot?" | Do you use it intentionally or blindly accept suggestions? |
| "What is agent mode?" | Do you understand tool-assisted execution? |
| "How does Copilot get context?" | Can you control input quality? |
| "Can Copilot write tests?" | Do you review locators, assertions, data, and waits? |
| "What are MCPs?" | Do you understand tool/context extension without hype? |
| "How do you keep output consistent?" | Do you use instructions, examples, and framework patterns? |

## Complete topic breakdown

### 1. Inline suggestions

Inline suggestions appear while typing code. They are useful for small local completions:

- finishing a Playwright assertion
- completing a fixture name
- writing a simple helper
- filling repetitive test data
- completing imports
- writing small Java/TestNG boilerplate

Best use:

- accept small pieces only
- read before accepting
- prefer clear test names and meaningful assertions
- reject suggestions that introduce weak waits, brittle locators, or hidden assumptions

SDET example:

```ts
await expect(page.getByRole('button', { name: 'Submit' })).toBeEnabled();
```

If Copilot suggests only:

```ts
await expect(page.locator('.submit')).toBeVisible();
```

do not accept blindly. Visibility of a CSS button may not prove the business behavior.

### 2. Ask mode

Ask mode is best when Sagar wants explanation without edits.

Use Ask mode for:

- "Explain this Playwright fixture."
- "Why is this test flaky?"
- "What does this TypeScript error mean?"
- "Compare this locator with a role-based locator."
- "What are the risks in this test case?"
- "What follow-up question can an interviewer ask from this answer?"

Ask mode is safer during learning because it should not change files. It is also good for
interview prep because it trains explanation.

Prompt pattern:

```text
Explain this Playwright test like I am preparing for an SDET interview.
Focus on fixtures, locators, waits, assertions, test data, and maintainability.
Do not rewrite code unless I ask.
```

### 3. Edit mode

Edit mode is best for a targeted local change.

Use Edit mode for:

- improving one test file
- replacing brittle locators
- adding missing assertions
- extracting repeated code into a helper
- converting one manual scenario into one test
- cleaning names and comments
- adding negative cases in a controlled file

Good prompt:

```text
Update only this Playwright test.
Replace brittle CSS locators with role/label/test-id locators where possible.
Add one meaningful assertion after the save action.
Do not change framework structure.
Keep the existing fixture pattern.
```

Bad prompt:

```text
Make this framework better.
```

That is too broad. It may cause unrelated refactoring.

### 4. Agent mode

Agent mode is for a complete bounded task where Copilot may need to inspect the workspace,
edit files, run commands, read failures, and iterate.

Use Agent mode for:

- create a new Playwright test following existing framework style
- investigate one failing test from error plus trace/log
- add one helper and update affected tests
- migrate a small Selenium pattern to Playwright style
- build a test data utility
- run tests and fix only failures caused by the current change

Do not use Agent mode for:

- vague "automate this module"
- wide framework redesign without a plan
- production secrets or confidential data
- uncontrolled self-healing behavior
- release sign-off

Safe agent prompt:

```text
Goal: add one Playwright test for the valid coupon checkout flow.

Read:
- existing checkout tests
- fixture setup
- page object for checkout
- test data helper

Rules:
- follow current framework style
- use role/label/test-id locators
- no hard waits
- add assertions for order total and success message
- do not touch unrelated tests
- run only the relevant test first
- summarize changed files and remaining risk
```

### 5. Plan mode

Plan mode is useful before letting the agent edit files. It should gather context and propose
steps.

Use Plan mode when:

- the task is unclear
- multiple files may be affected
- a framework decision is involved
- the change could create flakiness
- the test data strategy is not obvious
- you want a review before implementation

QA planning prompt:

```text
Create an automation plan for this Jira story.
Do not write code yet.
Identify:
- functional scenarios
- regression impact
- automation candidates
- test data needs
- API setup possibilities
- Playwright files likely to change
- risks and assumptions
```

Interview wording:

> "For broad work, I prefer planning first. I ask Copilot to identify impacted areas,
> assumptions, and test strategy before it edits code. That prevents the agent from creating
> a large but shallow automation change."

### 6. Custom agents

VS Code supports custom agents using `.agent.md` files. Older material may call these
custom chat modes. The newer VS Code wording is custom agents.

Custom agents let you define:

- agent name and description
- tools available to that agent
- model preference
- body instructions
- handoffs to other agents
- optional MCP server access

Workspace location:

```text
.github/agents/<name>.agent.md
```

User-level location:

```text
~/.copilot/agents/<name>.agent.md
```

SDET agent examples:

| Agent | Purpose | Tool boundary |
|---|---|---|
| QA Planner | story analysis, scenarios, risks | mostly read/search |
| Automation Builder | create/update tests | edit + test tools |
| Test Reviewer | review generated tests | read/search, maybe no edit |
| Failure Analyst | inspect error, trace, logs | read/search/test output |
| Locator Coach | suggest locator strategy | read/browser tools |

Best practice:

- keep planner read-only where possible
- keep builder scoped to specific files
- keep reviewer independent from builder
- do not give every agent every tool

### 7. Prompt files

Prompt files are reusable slash commands stored as `.prompt.md` files. VS Code docs describe
them as useful for common tasks, but also note that prompt files are moving toward agent
skills for future agent-host flows. So Sagar should learn them, but not overbuild everything
around prompt files only.

Workspace location:

```text
.github/prompts/<name>.prompt.md
```

Good SDET prompt file ideas:

- `/review-playwright-test`
- `/generate-test-cases-from-story`
- `/analyze-flaky-test`
- `/convert-manual-flow-to-playwright`
- `/write-bug-report-from-evidence`

Prompt file frontmatter can include:

```yaml
---
description: Review a Playwright test for SDET quality
agent: ask
tools: ['search/codebase']
---
```

Important: prompt files are manually invoked. They are not the same as always-on custom
instructions.

### 8. Agent skills

Agent skills are folders of instructions, scripts, and resources. A skill is better than a
single prompt when the task has a repeatable workflow.

Skill shape:

```text
skill-name/
  SKILL.md
  references/
  scripts/
  assets/
```

SDET skill examples:

- Playwright test review skill
- Jira story to QA scenario skill
- flaky test debug skill
- Selenium exception analysis skill
- bug report quality skill
- API validation strategy skill

Use a skill when:

- the workflow is repeated often
- there are supporting examples or templates
- the agent needs step-by-step procedure
- the quality standard must stay consistent

Do not use a skill when:

- one simple prompt is enough
- the task changes completely each time
- the instructions are only personal preference

### 9. MCP servers

MCP means Model Context Protocol. It is a standard way to connect Copilot or another AI agent
to external tools and data sources.

For QA, MCP can connect the AI assistant to:

- GitHub issues/PRs
- Playwright browser automation
- test management tools
- databases or APIs, if safely configured
- documentation systems
- CI/CD systems

Important interview boundary:

> "MCP does not make AI intelligent by itself. It gives the model access to tools or context.
> The value depends on tool permissions, safe configuration, and review."

Playwright MCP vs Playwright CLI:

| Area | Playwright MCP | Playwright CLI for agents |
|---|---|---|
| Integration style | MCP server tools | command-line workflow |
| Main value | structured browser control for AI clients | token-efficient browser automation for coding agents |
| Good for | IDE agents with MCP support | coding agents balancing codebase + browser context |
| Risk | broad tool access if not scoped | command execution must still be controlled |

For Sagar's current positioning, say:

> "I am stronger with Playwright CLI agent-style workflow and Copilot-assisted drafting. I
> understand MCP as the emerging integration layer, but I do not overclaim production MCP
> usage unless I actually use it in the project."

### 10. Custom instructions

Custom instructions are always-on guidance for Copilot.

Common files:

```text
.github/copilot-instructions.md
.github/instructions/*.instructions.md
AGENTS.md
CLAUDE.md
GEMINI.md
```

For VS Code Copilot Chat, GitHub documentation lists support for:

- repository-wide instructions using `.github/copilot-instructions.md`
- path-specific instructions using `.github/instructions/**/*.instructions.md`
- agent instructions using `AGENTS.md`

For Copilot CLI and cloud agent, support is broader and can include additional agent
instruction files depending on the surface.

Good SDET instruction:

```text
For Playwright tests, prefer user-facing locators such as getByRole, getByLabel,
getByText, and getByTestId. Avoid hard waits. Add assertions that prove business behavior,
not only element visibility. Follow existing fixture and page object patterns.
```

Bad instruction:

```text
Always make tests pass.
```

That encourages hiding defects.

## Manual testing angle

In manual QA work, Copilot helps convert rough thinking into structured output.

Use it for:

- story review
- missing acceptance criteria questions
- positive/negative/boundary scenarios
- role-based scenarios
- exploratory charters
- test data combinations
- bug report wording
- test summary drafts

Manual QA prompt:

```text
Analyze this Jira story as an SDET.
Return:
- assumptions
- missing questions
- positive scenarios
- negative scenarios
- boundary cases
- regression impact
- automation candidates
Do not invent business rules. Mark unknowns clearly.
```

Review gate:

- remove duplicate cases
- prioritize by business risk
- confirm expected behavior with PO/BA/dev
- avoid copying AI output directly into test management tools

## Automation angle

In automation, Copilot helps most when the framework context is supplied.

Give Copilot:

- one similar existing test
- fixture or setup file
- page object or helper pattern
- locator strategy
- assertion style
- test data rules
- cleanup rule
- run command
- exact goal

Playwright prompt:

```text
Create a Playwright test draft for this scenario using the existing framework style.
Use the current fixtures and page object pattern.
Prefer getByRole/getByLabel/getByTestId.
No hard waits.
Add assertions for business outcome, not only visibility.
Mark assumptions as comments.
```

Selenium prompt:

```text
Review this Selenium TestNG test for flaky behavior.
Focus on waits, stale element risk, locator stability, Page Object boundaries,
ThreadLocal safety, and assertion quality.
Do not rewrite the whole framework.
```

## Exact surface to remember

Commands and file names:

```text
.github/copilot-instructions.md
.github/instructions/<name>.instructions.md
.github/agents/<name>.agent.md
.github/prompts/<name>.prompt.md
~/.copilot/agents/<name>.agent.md
~/.copilot/copilot-instructions.md
```

Terms:

- Ask mode: explain/analyze without edits
- Edit mode: targeted code change
- Agent mode: tool-assisted task execution
- Plan mode: context and implementation plan before edits
- custom agent: reusable agent persona/configuration
- prompt file: reusable slash command
- skill: reusable workflow folder with `SKILL.md`
- MCP: standard protocol for giving AI access to tools/context
- context window: how much information the model can consider
- human-in-the-loop: human review before trusting output

One-line memory:

> "Mode decides how Copilot behaves; context decides answer quality; review decides whether
> the output is safe."

## Real project example

BASES_PLAYWRIGHT:

Sagar can explain a real workflow like this:

1. Read the Jira story and acceptance criteria.
2. Use Ask or Plan mode to identify scenarios, risks, and regression impact.
3. Select one automation-worthy scenario.
4. Give Copilot a similar Playwright test, fixture pattern, locator rules, and expected
   assertions.
5. Use Edit or Agent mode to draft the test.
6. Review locator quality, waits, assertions, data setup, cleanup, and CI behavior.
7. Run the focused test and inspect failures.
8. Commit only after the test proves meaningful behavior.

CIP_LEGACY:

In Selenium/Java, Sagar can say:

- Copilot helps explain Selenium exceptions.
- It can review XPath and wait strategy.
- It can draft TestNG skeletons.
- It can refactor repeated Java utility code.
- It can suggest RestAssured validation checks.
- It does not replace framework understanding.

## Common mistakes / weak areas

- Saying "agent mode" but not explaining what the agent can access.
- Using Agent mode for vague tasks.
- Letting Copilot rewrite unrelated files.
- Accepting code that only checks visibility.
- Forgetting data setup and cleanup.
- Giving too much context instead of relevant context.
- Mixing confidential office code or internal data into prompts.
- Assuming MCP means fully autonomous testing.
- Creating many prompt files but never reviewing output quality.
- Using "make test pass" prompts that hide product defects.

## 30-second answer

I use GitHub Copilot in VS Code as an SDET assistant, not as an autopilot. Ask mode helps
me understand code, errors, or testing risks. Edit mode is useful for targeted changes.
Agent mode is useful for bounded tasks where it can inspect files, make changes, run tests,
and iterate. For test automation, I give it the existing framework style, locator strategy,
fixture pattern, test data rules, and expected assertions. Then I review the output for
stability and real coverage before using it.

## 2-minute answer

In VS Code, I use Copilot differently depending on the task. If I am learning or debugging,
I use Ask mode because I want explanation without file changes. If I need a small change in
one test, I use Edit mode with clear boundaries. If I need Copilot to inspect the repo,
create a test, run it, and fix only related failures, I use Agent mode. For broader work, I
prefer Plan mode first so it identifies impacted files, assumptions, test data needs, and
risks before touching code.

For QA, context is the main skill. I do not just ask "write Playwright test". I provide the
manual scenario, expected behavior, existing test style, fixtures, page object pattern,
locator preference, data rules, and assertion expectation. Copilot can draft faster, but the
review gate is still mine: stable locators, no hard waits, meaningful assertions, isolated
data, cleanup, and CI suitability.

The trade-off is that Copilot can create code that looks polished but proves very little.
So I use it for speed and breadth, but not for final judgment. The SDET still owns risk,
coverage, defect quality, and release confidence.

## Interview Q&A

### Q1: How do you use GitHub Copilot in VS Code?

I use it in different modes. Ask mode for explanation and analysis, Edit mode for targeted
file changes, Agent mode for bounded tasks that need repo search and test execution, and
Plan mode before wider changes. For automation, I give it framework context and review the
generated code before accepting it.

### Q2: What is the difference between Ask, Edit, Agent, and Plan mode?

Ask is for explanation without changes. Edit is for controlled local changes. Agent mode can
use tools to inspect files, edit, run commands, and iterate. Plan mode is for thinking
through the approach before implementation.

### Q3: When would you not use Agent mode?

I avoid Agent mode for vague or risky work: broad framework redesigns, confidential data,
release decisions, or "fix everything" prompts. I use it only when the goal and boundaries
are clear.

### Q4: What context do you give Copilot for Playwright?

I give the manual scenario, expected result, similar existing test, fixture pattern, page
object/helper style, locator strategy, test data rules, cleanup expectation, and assertion
standard.

### Q5: What is the risk of Copilot-generated tests?

False confidence. A generated test can pass while checking only visibility or the happy
path. It may also use brittle locators, hard waits, shared data, or skip cleanup.

### Q6: What are custom agents?

Custom agents are reusable AI configurations in VS Code, usually stored as `.agent.md`
files. They define the agent's role, instructions, available tools, model preference, and
sometimes handoffs. For QA, I could have a QA Planner, Automation Builder, and Test Reviewer.

### Q7: What are prompt files?

Prompt files are reusable slash commands stored as `.prompt.md` files. They are useful for
common tasks like reviewing a Playwright test or generating test cases from a story. They
are invoked manually and are different from always-on custom instructions.

### Q8: What are agent skills?

Agent skills are reusable folders with a `SKILL.md` and optional references, scripts, or
assets. They are better for repeatable workflows, like flaky test debugging or QA story
analysis.

### Q9: What is MCP?

MCP is a protocol that lets Copilot connect to external tools and context sources. For QA,
that could mean GitHub, Playwright browser automation, CI, or test-management tools. MCP
extends tool access; it does not remove the need for review.

### Q10: How do you keep Copilot from producing generic output?

I give scoped context, examples from the existing framework, explicit constraints, and a
clear output format. I also ask it to mark assumptions and then review the result manually.

## Quick revision points

- Ask = explain, Edit = targeted change, Agent = tool-assisted task, Plan = approach first.
- Copilot is strongest when context is scoped and examples are supplied.
- For Playwright, always give locator, fixture, data, and assertion rules.
- Custom agents use `.agent.md`.
- Prompt files use `.prompt.md`.
- Skills use `SKILL.md`.
- MCP gives tool/context access; it is not magic autonomy.
- Generated tests must pass the review gate before commit.

Recall drill:

1. Explain Ask vs Edit vs Agent vs Plan in 30 seconds.
2. Explain how you prompt Copilot for a Playwright test.
3. Explain what context Copilot needs.
4. Explain why generated tests can be dangerous.
5. Explain MCP without hype.
