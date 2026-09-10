# Project Stories — The Two-Track Walkthrough

## What it is

The detailed version of your two projects — what you'd say when an interviewer stops the small talk and says *"walk me through a project you've worked on."* This goes deeper than the behavioral stories: it's architecture, decisions, trade-offs, and your specific ownership, not just a 2-minute STAR anecdote.

You need each project fluent at three depths: a 30-second summary, a 2-minute walkthrough, and a "go as deep as they want" version where every layer has a decision behind it.

---

## Why it matters

Behavioral rounds test *how you operate*. Project deep-dives test *whether you actually did the work you claim*. A senior interviewer will pick one thread — "why fixtures over hooks?", "how did you handle test data?", "what broke and how did you fix it?" — and follow it until you either demonstrate real ownership or run out of depth. The candidates who get the 20-24 LPA offer are the ones where every layer has a reason, and every reason names a cost.

---

## Project 1 — BASES_PLAYWRIGHT (Track 1, Jan 2025 – present)

### 30-second frame

> "The current project is a modern Playwright and TypeScript UI E2E automation framework for a data-heavy analytics product. I use API-assisted setup and cleanup for test data, with reusable fixtures, config, and CI integration through Azure DevOps/GitHub Actions. The result I'm proudest of is taking the CI feedback loop from about 40 minutes to 12."

### 2-minute walkthrough

**The context:** a Playwright + TypeScript suite for a BASES analytics product. The main scope is UI E2E automation, with Playwright request/API capabilities used for setup, cleanup, and test data control so the UI tests start from a reliable state.

**What I own:** the framework architecture — the fixture layer, API-assisted setup/cleanup, `playwright.config.ts`, the Page Object structure, and CI execution in Azure DevOps/GitHub Actions.

**The design decisions that matter:**
- **Fixtures over raw hooks** for setup, because they compose and evaluate lazily — a test only pays for the fixtures it actually uses. Cost: a steeper learning curve for someone new to Playwright.
- **Storage-state auth** so tests don't log in through the UI every run. Cost: it creates a coverage gap — the login flow itself is no longer exercised by most tests, so I keep one dedicated UI login test to cover it.
- **Sharding across workers** in CI, which is most of the 40→12 win, plus removing wasted `waitForTimeout` calls in favour of web-first assertions.

**AI angle, honestly framed:** I use GitHub Copilot Chat and Playwright CLI workflows to draft and maintain tests — roughly a 30% efficiency gain on boilerplate and maintenance support. Not self-healing selectors, not Playwright MCP as a resume skill, not an agent architecture.

### If they go deeper

- **"How do you keep it non-flaky in parallel?"** → Every test owns its data through a fixture; no shared mutable state; web-first assertions instead of sleeps. See [Waiting Strategies](../../02_PLAYWRIGHT/01_BASICS/07_Waiting_Strategies.md).
- **"Why Playwright over Cypress here?"** → Multi-tab/context support, strong auto-waiting, parallel execution, and built-in request context for setup/cleanup. See [Tool Selection](../../06_LEADERSHIP_AND_ARCHITECTURE/01_Tool_Selection_Framework.md).
- **"Kubernetes — why does that matter to tests?"** → Rolling deploys can swap the target mid-run; readiness checks and retrying on the right conditions matter. See [Docker & Kubernetes](../../05_PROCESS_AND_TOOLING/03_Docker_And_Kubernetes.md).

---

## Project 2 — CIP_LEGACY (Track 2, Nov 2022 – Dec 2024)

### 30-second frame

> "My first two years were an enterprise Selenium and Java framework — over 300 automated scenarios across UI, API with RestAssured, and database validation with SQL, running on Jenkins. That's where I did the serious flakiness work and got the suite under 2% flaky on first attempt."

### 2-minute walkthrough

**The context:** a large regression suite for a CIP product. Selenium WebDriver for UI, RestAssured for the API layer, SQL against Oracle/MySQL for backend validation. TestNG as the runner, Maven for the build, Jenkins for CI.

**What I own:** contributed heavily to framework architecture, owned the parallel-execution stability and the flakiness reduction.

**The design decisions that matter:**
- **ThreadLocal WebDriver** for parallel runs, because a shared driver across threads is the classic race condition. Cost: you *must* call `remove()` in teardown or you leak drivers and poison the thread pool. See [Parallel Execution & ThreadLocal](../../03_SELENIUM_JAVA/02_TESTNG/03_Parallel_Execution_And_ThreadLocal.md).
- **Explicit waits only, never mixed with implicit** — mixing them produces unpredictable timeouts. Replaced hard sleeps with Fluent Waits that poll on the real condition. This is what drove flakiness under 2%.
- **POJO serialization with Jackson** for API payloads, using `@JsonIgnoreProperties(ignoreUnknown = true)` so a new field in the response doesn't break every test.

### If they go deeper

- **"Walk me through your framework."** → The full answer lives in [Framework Architecture](../../03_SELENIUM_JAVA/06_FRAMEWORK_AND_BUILD/01_Framework_Architecture.md) — describe it as decisions with costs, not a layer diagram.
- **"How did you validate the database?"** → JDBC queries checking that the UI action actually persisted correctly, orphan detection with joins. See [SQL For Testers](../../03_SELENIUM_JAVA/05_SQL_AND_DB/01_SQL_For_Testers.md).
- **"How did you know flakiness was actually fixed?"** → Measured first-attempt pass rate, not after-retry. Reproduced under stress before claiming a fix. See [Test Flakiness Mastery](../../04_TEST_ENGINEERING/01_Test_Flakiness_Mastery.md).

---

## The bridge between them (the senior signal)

The single most valuable thing you can say connects the two projects:

> "Having built in both a legacy Selenium stack and a modern Playwright one, I don't see Playwright as universally better — I see it as better for *my* context: a greenfield TypeScript product with a team that could adopt it. If I walked into a shop with a mature, stable Selenium grid and a Java team, I'd tell them not to migrate. That's a decision you can only make credibly if you've lived in both."

That sentence is worth more than any single technical detail. It's the difference between "I know a tool" and "I make tooling decisions."

---

## Common interview Q&A

**Q: "Which project are you most proud of?"**
Playwright — framework ownership and the 40→12 result. But mention the flakiness work on legacy as the harder engineering problem. Pride in the modern, respect for the hard.

**Q: "What would you do differently?"**
Have a real answer. *"On the legacy side I abstracted too early — built page objects before I knew the right shape. I'd wait until the third repetition now."* (Ties to your failure story.)

**Q: "What was the hardest bug you found?"**
The data-collision flakiness: intermittent reds with no code change, root-caused to tests sharing data under parallel scheduling. Fixed structurally, not with retries.

**Q: "How big was the team? What was your role in it?"**
Be precise about "I" vs "we". You *contributed to* the legacy framework; you *own* the Playwright framework design. Never inflate — the resume dates make role claims checkable.

---

## Weak areas to watch

- **Don't blur the two stacks together.** Keep them cleanly separated by date and tool. The interviewer has your resume timeline open.
- **Don't claim sole ownership of the legacy framework.** You contributed and owned specific areas (parallel stability, flakiness). "We" for the framework, "I" for your slice.
- **Every decision needs a cost.** "I used fixtures" is junior. "I used fixtures over hooks, and the cost is a steeper learning curve" is senior.
- **Keep AI honest.** Test drafting, locator refinement, maintenance support, 30% gain. Never self-healing selectors, Playwright MCP as a resume skill, or agent architectures — you didn't run those in production.
- **Metrics stay pinned to their track.** 40→12 is Playwright. Under 2% flaky and 300+ scenarios are legacy. 90% coverage is NielsenIQ overall.

---

## Key takeaway

> **Two projects, told as decisions with costs — and the bridge sentence that says you can choose between them.** The deep-dive isn't testing whether you can list what you used; it's testing whether every layer had a reason. The one line that seals the senior title: *"I've built in both, so I know where each actually fits."*
