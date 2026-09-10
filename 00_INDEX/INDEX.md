# SDET MASTER NOTES — MASTER INDEX

**140 notes** · Last updated: 2026-09-10

> **This file is the single source of truth for what exists and what's missing.**
> Update it whenever a note is added, moved, renamed, or removed.

---

## Reading Order

Sections are numbered as a **learning arc**, not by importance:

```
foundation  →  tools  →  application  →  practice  →  seniority  →  delivery
```

| # | Section | Role in the arc |
|---|---------|-----------------|
| **01** | TESTING_FUNDAMENTALS | What testing *is* — before any tool |
| **02** | PLAYWRIGHT | Primary track (TypeScript lives inside) |
| **03** | SELENIUM_JAVA | Secondary track (Java lives inside) |
| **04** | TEST_ENGINEERING | Running a suite at scale — theory paired with practice |
| **05** | PROCESS_AND_TOOLING | How you work — Agile, Git, Docker, tools |
| **06** | LEADERSHIP_AND_ARCHITECTURE | Above the tools — decisions and trade-offs |
| **07** | INTERVIEW_PREP | Packaging it for delivery |
| **08** | QUICK_REVISION | Revision layer — round-based cheat sheets + living weak-spots file |
| **09** | EXTERNAL_REFS | Borrowed PDFs — read from, never copy. Gitignored |
| **10** | PRACTICAL_ROUNDS | Live/practical rounds — coding, SQL, locators, code review, live UI/API, take-home |
| **11** | AI_IN_TESTING | AI-assisted QA workflows, agents, story/PR analysis, automation drafting |

**Placement rule:** if the answer changes on a different tool → inside a track folder (02/03).
If the answer would be identical in a Selenium interview → top level.

**Reporting spans three notes** — this is deliberate, since each is tool-locked except Allure:

| Note | Works with |
|---|---|
| `02_PLAYWRIGHT/05_CICD_AND_EXECUTION/04_Reporting_And_Artifacts` | Playwright only |
| `03_SELENIUM_JAVA/06_FRAMEWORK_AND_BUILD/04_Reporting_ExtentReports` | Java only |
| `05_PROCESS_AND_TOOLING/05_Allure_Reporting` | **Both stacks** |

Each cross-references the other two.

**Which section maps to which interview round:**

| Round | Read |
|---|---|
| QA fundamentals / warm-ups | `01_TESTING_FUNDAMENTALS` |
| Manual QA depth / release confidence | `01_TESTING_FUNDAMENTALS/09` to `01_TESTING_FUNDAMENTALS/13` |
| Technical — Playwright | `02_PLAYWRIGHT` |
| Technical — Selenium/Java | `03_SELENIUM_JAVA` |
| Scenario / problem-solving | `04_TEST_ENGINEERING` |
| Process / ways of working | `05_PROCESS_AND_TOOLING` |
| Manager / senior | `06_LEADERSHIP_AND_ARCHITECTURE` |
| HR / behavioral | `07_INTERVIEW_PREP` |
| Live coding / SQL / locators / code review / take-home | `10_PRACTICAL_ROUNDS` |
| AI in testing / agentic QA / AI-assisted automation | `11_AI_IN_TESTING` |
| Final revision (any round) | `08_QUICK_REVISION` |

---

## 01_TESTING_FUNDAMENTALS (13) ✅

Foundation. These are the **manual + automation QA base notes** — asked first in an
interview, cheap to answer, expensive to fumble. ISTQB CTFL and 3.8+ years across
manual + automation testing make them fair game.

| # | Note | Covers |
|---|------|--------|
| 01 | Types & Levels Of Testing | **Smoke vs sanity**, **regression vs retesting**, test pyramid, manual vs automation selection |
| 02 | Principles & Terminology | 7 ISTQB principles, QA/QC/testing, error/defect/failure, verification vs validation |
| 03 | Test Design Techniques | EP, BVA, decision tables, state transition, exploratory charters, automation conversion |
| 04 | STLC & Test Artifacts | Phases, strategy vs plan, scenario vs case, RTM, Agile QA artifacts |
| 05 | Defect Lifecycle & Reporting | **Severity vs priority**, actionable bug reports, evidence, automation-failure triage |
| 06 | White-Box Testing | Coverage criteria, black/white/grey-box framing, why 100% coverage is weak |
| 07 | Performance Testing | **Load vs stress vs spike**, percentiles, manual performance risk observation |
| 08 | Security, Usability & Accessibility | **IDOR**, auth/authz, usability/accessibility checks, what automates vs needs humans |
| 09 | Requirement Analysis & Testability | AC clarity, ambiguity, testability questions, automation-ready expected results |
| 10 | Test Scenarios, Cases & Checklists | Scenario vs case vs checklist, case fields, Agile documentation depth |
| 11 | Test Data & Environment | Valid/invalid/boundary data, roles, environment checks, automation data stability |
| 12 | Exploratory & Risk-Based Testing | Charters, time boxes, risk = impact x likelihood, automation candidates |
| 13 | Release Readiness & Sign-Off | Test summary, evidence, known risks, blockers, ready/ready-with-risk/not-ready recommendation |

---

## 02_PLAYWRIGHT — Track 1 (39) 🚧

### 01_BASICS (12) ✅
| # | Note | Covers |
|---|------|--------|
| 01 | What Is Playwright | Runner + browser automation + assertions + artifacts; honest Selenium/Cypress contrast |
| 02 | Browser, Context, Page | Process/session/tab model, isolation, storage state, multi-user testing |
| 03 | Locators | Role/label/text/test-id hierarchy, strict mode, filters, CSS/XPath fallback |
| 04 | Actions | click/fill/check/select/upload/download, actionability, force-click risk |
| 05 | Assertions | Locator/page/API/generic assertions, auto-retry vs non-retry, soft/poll/toPass |
| 06 | Debugging | Trace viewer, report, video, UI mode, console/network evidence, CI failures |
| 07 | **Waiting Strategies** | Actionability checks, web-first assertions, event waits, response waits, hard-wait red lines |
| 08 | Handling Complex UI | iframes, dialogs, popups, uploads, downloads, shadow DOM, drag/drop |
| 09 | Error Messages And Exceptions | strict mode, timeout, actionability, closed context, auth/setup errors |
| 10 | Navigation And Events | `goto`, URL/load-state waits, popup/download/request/response event pre-wait |
| 11 | Emulation, Devices, And Permissions | device presets, viewport, locale/timezone, permissions, geolocation, browser matrix |
| 12 | Clock And Time-Dependent Testing | browser clock control, date/timer testing, browser-time vs server-time boundary |

### 02_FRAMEWORK (12)
| # | Note | Covers |
|---|------|--------|
| 01 | Framework Design | Architecture as decisions, separation of concerns |
| 02 | Page Object Model | Encapsulation, why change propagates |
| 03 | Fixtures | Reusable setup, scopes, lazy evaluation |
| 04 | Hooks | beforeEach/All, when hooks beat fixtures |
| 05 | Config Files | playwright.config.ts, **forbidOnly**, retries, reporters |
| 06 | Test Organization | Feature-based folders, naming conventions |
| 07 | Authentication State | Storage state, **the coverage gap it creates** |
| 08 | Network Mocking | route/fulfill/abort, mock external not internal |
| 09 | Parallel Execution | Workers, sharding, isolation requirements |
| 10 | Test Runner Features | test.step, tags, annotations, soft assertions |
| 11 | Visual Testing | Screenshot comparison, baseline environment problem |
| 12 | Fixtures, Hooks, And Auth Setup Practical | `base.extend`, `use()`, scopes, storage state, API token fixture |

### 03_API_TESTING (5)
| # | Note | Covers |
|---|------|--------|
| 01 | APIRequestContext | Built-in client, **request vs page.request** |
| 02 | HTTP Methods | REST semantics, idempotency, status codes |
| 03 | Response Validation | toMatchObject over toEqual, schema validation |
| 04 | API Non-Functional Testing | Response time, auth/security, error quality, schema stability, invalid type handling |
| 05 | API Auth Token Management | Bearer token fixture, expiry refresh, `request` vs `page.request`, storage-state boundary |

### 04_TYPESCRIPT (4)
| # | Note | Covers |
|---|------|--------|
| 01 | TypeScript Basics | Types, unions for test data, unknown vs any |
| 02 | Advanced Interfaces | Partial/Pick/Omit/Record, derive don't duplicate |
| 03 | Generics | Typed API client, constraints, `as T` isn't validation |
| 04 | Async/Await | **The forgotten `await`**, no-floating-promises |

### 05_CICD_AND_EXECUTION (6)
| # | Note | Covers |
|---|------|--------|
| 01 | GitHub Actions | Workflow, caching, sharding, `if: always()` |
| 02 | Azure DevOps | Pipeline YAML, **CI=true gotcha**, JUnit publishing |
| 03 | Docker & Environments | Image/version matching, env config, fail-fast |
| 04 | Reporting & Artifacts | Reporters per audience, trace-on-first-retry |
| 05 | Tooling & AI Productivity | Codegen, UI mode, honest AI positioning |
| 06 | Project Setup, Commands, And Scripts | `npm init playwright@latest`, execution commands, `package.json`, `npm ci` |

---

## 03_SELENIUM_JAVA — Track 2 (24) ✅

### 01_SELENIUM_UI (6)
| # | Note | Covers |
|---|------|--------|
| 01 | WebDriver Architecture | W3C protocol, why stale elements exist |
| 02 | Locators & XPath | Priority, axes, **the table-row XPath** |
| 03 | Waits & Synchronization | **Implicit/explicit mixing problem** |
| 04 | Common Exceptions | Stale, intercepted, not-interactable + real fixes |
| 05 | Advanced Interactions | Frames, windows, alerts, Actions, JS executor |
| 06 | Page Object Model (Java) | **By vs WebElement**, PageFactory trade-offs |

### 02_TESTNG (4)
| # | Note | Covers |
|---|------|--------|
| 01 | TestNG Fundamentals | Annotation order, @BeforeTest trap, groups |
| 02 | DataProviders & Data-Driven | @DataProvider vs @Parameters, POI cell types |
| 03 | **Parallel Execution & ThreadLocal** | Shared driver bug, **why remove() matters** |
| 04 | Listeners & Retry | ITestListener, screenshots, retry philosophy |

### 03_JAVA_FOR_SDET (4)
| # | Note | Covers |
|---|------|--------|
| 01 | OOP For Frameworks | Four pillars **via your framework**, not definitions |
| 02 | Collections & Streams | ConcurrentHashMap for parallel, stream cost |
| 03 | **Java 8 Features** | Lambdas, **why `ExpectedCondition` takes one**, Optional |
| 04 | Exceptions & Generics | **Catching hides failures**, type erasure |

### 04_RESTASSURED_API (3)
| # | Note | Covers |
|---|------|--------|
| 01 | RestAssured Basics | given/when/then, specifications, GPath |
| 02 | **POJO Serialization** | Jackson, ObjectMapper, `@JsonIgnoreProperties` |
| 03 | Auth & Validation | Token caching, 401 vs 403, JSON schema |

### 05_SQL_AND_DB (2)
| # | Note | Covers |
|---|------|--------|
| 01 | SQL For Testers | Joins, orphan detection, DENSE_RANK, NOT IN trap |
| 02 | Database Testing Strategy | JDBC, what to validate where, schema coupling |

### 06_FRAMEWORK_AND_BUILD (5)
| # | Note | Covers |
|---|------|--------|
| 01 | **Framework Architecture** | The "explain your framework" answer |
| 02 | Maven & Build | Scopes, surefire, profiles, dependency:tree |
| 03 | Jenkins CI | Pipeline, `post { always }`, headless viewport |
| 04 | Reporting (ExtentReports) | **ExtentTest ThreadLocal trap**, flush(), base64 |
| 05 | Selenium Grid | Selenium 4 components, session queue, Grid vs cloud |

---

## 04_TEST_ENGINEERING — Scenario Round (9) ✅

Tool-agnostic. **Theory paired directly with its practice** — read in order.

| # | Note | Pairing |
|---|------|---------|
| 01 | **Test Flakiness Mastery** | ┐ root causes, Hard Sleeps → Fluent Waits |
| 02 | Flaky Test Debugging | ┘ the hands-on workflow |
| 03 | Performance Optimization | ┐ the optimization ladder |
| 04 | Slow Test Suite | ┘ the diagnostic |
| 05 | Passes Locally, Fails In CI | "Walk me through debugging that" |
| 06 | Selector Breakage | "Tests break every UI change" |
| 07 | Test Data Management | Isolation, factories, API seeding, cleanup |
| 08 | Real World Problems | Six erosion patterns, silent test debt |
| 09 | Test Suite Metrics | **First-attempt pass rate**, escape rate |

---

## 05_PROCESS_AND_TOOLING (5) ✅

| # | Note | Covers |
|---|------|--------|
| 01 | Agile, Scrum & Jira | Ceremonies, **the mini-waterfall problem**, DoD |
| 02 | Git Workflow | Branching, conflicts in test code, committed secrets |
| 03 | Docker & Kubernetes | **Rolling deploys break tests**, readiness checks |
| 04 | Postman & API Tooling | Exploration vs regression, Newman's limits |
| 05 | Allure Reporting | **History is the differentiator**, failed vs broken |

---

## 06_LEADERSHIP_AND_ARCHITECTURE — Senior Round (4) ✅

| # | Note | The question it answers |
|---|------|------------------------|
| 01 | Tool Selection Framework | "Playwright vs Cypress vs Selenium?" |
| 02 | Architecture Decisions | "What trade-offs did you make?" |
| 03 | Mentoring & Code Review | "How would you onboard a junior SDET?" |
| 04 | Test Automation Strategy | "How do you decide what to automate?" |

---

## 07_INTERVIEW_PREP (7) ✅

| Folder | Status | Covers |
|--------|:---:|--------|
| 01_SELF_INTRO | ✅ | The 60-second opener — present-first, two-track, one metric |
| 02_PROJECT_STORIES | ✅ | Two-track walkthrough as decisions-with-costs, deeper than behavioral |
| 03_HR_AND_SALARY | ✅ | CTC negotiation, notice, why leaving, the pivot story, BGV reality |
| 04_COMPANY_SPECIFIC | ✅ | 30-min research pass, positioning by company type |
| 05_COMMUNICATION_SKILLS | ✅ | Claim → Reason → Example → Trade-off |
| 06_DESIGN_QUESTIONS | ✅ | Worked example, 5-step process |
| 07_BEHAVIORAL_STORIES | ✅ | The six stories, STAR structure |

---

## 08_QUICK_REVISION (8) ✅

Revision layer — dense, scannable recall sheets for the day of an interview. Learn from
the full section notes; fire from these.

| # | Note | Covers |
|---|------|--------|
| 00 | **Weak Spots** (living file) | Running log of hesitations + post-interview capture; update every session |
| 01 | Fundamentals Cheat Sheet | Smoke/sanity, severity/priority, ISTQB, design techniques |
| 02 | Playwright Cheat Sheet | Browser/Context/Page, locators, **where auto-wait stops**, CI 40→12 |
| 03 | Selenium + Java Cheat Sheet | WebDriver arch, wait mixing, **ThreadLocal remove()**, RestAssured, SQL |
| 04 | Scenario Round Cheat Sheet | 5 flakiness causes, debug walks, optimization ladder, metrics |
| 05 | Process & Leadership Cheat Sheet | Agile, Git, Docker/K8s, tool selection, trade-offs |
| 06 | Behavioral Cheat Sheet | STAR ratio, 6 stories, self-intro, HR/salary triggers |
| 07 | AI In Testing Cheat Sheet | Two-agent model, story-vs-PR gap, **deletion is the work**, review gate, red lines |

---

## 09_EXTERNAL_REFS — Borrowed material (not counted as notes)

Moved here from the old workspace-root `Prep/` folder. **Gitignored** — copyrighted
third-party PDFs stay on disk, out of version control.

| Folder | What it is |
|--------|-----------|
| `JavaScript/` | Reference books, snippet collections |
| `Typescript/` | TypeScript notes PDF |
| `Playwright/` | Udemy course material, framework-explanation deck |
| `Playwright/PlaywrightLearning.md` | **Sagar's own raw course log** (Feb 2026) — mine it for topics |
| `Office_Projects/PlaywrightTS/` | Internal NielsenIQ repo guide — never leaves the machine |

Read from it, never copy from it. Anything worth keeping gets rewritten into the
topic-first format in his own words. See `09_EXTERNAL_REFS/README.md`.

---

## 10_PRACTICAL_ROUNDS — Live / Practical Rounds (22) ✅

Tool-agnostic where possible; both stacks where it matters. The "do it in front of me"
rounds — the ones you can't bluff. Organized into subfolders per round type.

### 01_CODING_ROUNDS (7)
| # | Note | Covers |
|---|------|--------|
| 01 | Approach & the Meta-Skill | The 5-step method; **test your own solution** out loud |
| 02 | String Problems | Reverse, palindrome, first-unique, anagram, compression + edge cases |
| 03 | Array & Collection Problems | Two-sum, dedup, missing number, group-by; brute-force→map |
| 04 | Counting & HashMap Problems | The count-with-a-map template; the highest-value pattern |
| 05 | Logic & Number Problems | FizzBuzz, primes (√n), Fibonacci, digit manipulation, overflow |
| 06 | Java Idioms & Gotchas | `==` vs `.equals()`, Integer cache, streams, map idioms |
| 07 | TypeScript/JS Idioms & Gotchas | `===`, `??` vs `\|\|`, the forgotten `await`, sort comparator |

### 02_SQL_ROUNDS (5)
| # | Note | Covers |
|---|------|--------|
| 01 | Joins & Filtering | All join types, orphan pattern, `NOT IN`+NULL, WHERE vs HAVING |
| 02 | Aggregation & Grouping | `COUNT(*)` vs `COUNT(col)`, execution order, conditional aggregation |
| 03 | Window Functions | **The senior differentiator** — ROW_NUMBER/RANK/DENSE_RANK, top-N-per-group |
| 04 | SDET Validation Patterns | Persistence checks, integrity, reconciliation, JDBC, cross-table |
| 05 | Problem Bank & Gotchas | Rapid-fire prompts + the complete gotcha table |

### 03_CODE_REVIEW (1)
| # | Note | Covers |
|---|------|--------|
| 01 | Code Review Exercise | Worked flawed examples; correctness > flakiness > style |

### 04_LIVE_UI_TESTING (1)
| # | Note | Covers |
|---|------|--------|
| 01 | Live UI Test Writing | Design cases out loud first, then clean code; test-level trade-off |

### 05_LIVE_API_TESTING (1)
| # | Note | Covers |
|---|------|--------|
| 01 | Live API Testing | Status+body+schema+negatives; CRUD lifecycle; 401 vs 403 |

### 06_TAKE_HOME (1)
| # | Note | Covers |
|---|------|--------|
| 01 | Take-Home Assignment | Scope tight, build well; README is half the grade |

### 07_LOCATORS_MASTERY (6)
The in-depth locator guide — CSS, XPath, and finding elements in practice. Interview-tested.

| # | Note | Covers |
|---|------|--------|
| 01 | Locator Strategy | Priority hierarchy, good vs bad locators, `data-testid`, CSS vs XPath |
| 02 | CSS Selectors — Complete | Attribute operators, combinators, pseudo-classes, `:has()`, limits |
| 03 | XPath — Complete | Axes, functions, `text()` vs `.`, **the table-row XPath**, predicates |
| 04 | Playwright Locators | `getByRole`, filter/chaining, strictness, lazy = no stale elements |
| 05 | Selenium Locators | 8 `By` strategies, findElement vs findElements, **stale-element problem** |
| 06 | Finding & Debugging Locators | DevTools workflow, dynamic IDs, iframes, shadow DOM, tables |

---

## 11_AI_IN_TESTING (9) ✅

AI-assisted QA workflows across the whole testing lifecycle. Separate from Playwright
tooling because interviewers increasingly ask about AI end-to-end: Jira, PR review, test
design, regression selection, automation candidates, script drafting, and bug reporting.
Framing throughout is honest and bounded — AI drafts fast; QA judgment owns risk,
assertions, coverage, and release confidence.

| # | Note | Covers |
|---|------|--------|
| 01 | AI-Augmented QA Workflow | Two-agent model (Knowledge + Automation), where AI helps, what not to claim |
| 02 | PR And Jira Story Analysis | Story = intended behavior, PR = actual change; **test the gap**; blast radius |
| 03 | Test Plan And Case Generation | AI gives breadth, you supply depth+priority; **good QA is deletion** |
| 04 | Regression Selection And Coverage Tracking | Risk-based selection, coverage map, **not real code coverage** |
| 05 | Manual Flow To Playwright Drafts | Draft→harden; locators, waits, **visibility-only assertions**; green-but-wrong |
| 06 | Bug Drafting And Defect Quality | AI formats, you own facts; never invent repro; severity is yours |
| 07 | Review Gates For AI-Generated Tests | The 6-question gate; **"why does this exist?"**; fast green suite that proves nothing |
| 08 | Context Management For QA Agents | Skills vs task context, agent handoff, scoped context, avoiding generic output |
| 09 | Playwright CLI Agent Browser Exploration | CLI vs MCP, snapshots, refs, `find`, `generate-locator`, console/network commands |

---

## What's NOT Built Yet

| Gap | Priority | Notes needed |
|-----|:---:|:---:|
| **Gauge, Python** | ⚪ Deferred | — |
| └ On the resume; Sagar's call to revisit later | | |

> Learning arc, interview-prep, practical rounds, and the revision layer are all built.
> Remaining work is maintenance: keep `08_QUICK_REVISION/00_Weak_Spots.md` current after
> every session and interview, and add notes for any real-interview question the arc misses.

---

## Verified Metrics — Use Only These

| Metric | Value | Belongs to |
|--------|-------|-----------|
| Total experience | 3.8+ years across manual + automation testing | — |
| Automated scenarios | 300+ | CIP_LEGACY |
| Regression coverage | 90% | NielsenIQ overall |
| CI/CD feedback loop | **40 → 12 min** | BASES_PLAYWRIGHT |
| Flaky rate | **under 2%** (via Fluent Waits) | CIP_LEGACY |
| AI efficiency gain | 30% | BASES_PLAYWRIGHT |
| Awards | 2x Quarterly Award | NielsenIQ |

Source of truth: `JOB_PIPELINE/assets/Resume.tex`.
Metrics appear in ~11 notes only — never scatter them further.

---

## Note Format

Every note now follows topic-first learning:
1. What it is · 2. Why it matters in interviews · 3. Complete topic breakdown
4. Manual testing angle · 5. Automation angle · 6. Exact surface to remember
7. Real project example · 8. Common mistakes / weak areas · 9. 30-second answer
10. 2-minute answer · 11. Interview Q&A · 12. Quick revision points

Plus comparison tables, checklists, and code patterns where useful.
Template: `00_INDEX/NOTE_TEMPLATE.txt`
