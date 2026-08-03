# SDET MASTER NOTES — MASTER INDEX

**84 notes** · Last updated: 2026-08-03

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
| **08** | QUICK_REVISION | Revision layer (planned, after learning pass) |

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
| Technical — Playwright | `02_PLAYWRIGHT` |
| Technical — Selenium/Java | `03_SELENIUM_JAVA` |
| Scenario / problem-solving | `04_TEST_ENGINEERING` |
| Process / ways of working | `05_PROCESS_AND_TOOLING` |
| Manager / senior | `06_LEADERSHIP_AND_ARCHITECTURE` |
| HR / behavioral | `07_INTERVIEW_PREP` |

---

## 01_TESTING_FUNDAMENTALS (8) ✅

Foundation. These are the **warm-up questions** — asked first in an interview, cheap to
answer, expensive to fumble. ISTQB CTFL on the resume makes them fair game.

| # | Note | Covers |
|---|------|--------|
| 01 | Types & Levels Of Testing | **Smoke vs sanity**, **regression vs retesting**, test pyramid |
| 02 | Principles & Terminology | 7 ISTQB principles, error/defect/failure, verification vs validation |
| 03 | Test Design Techniques | EP, BVA, decision tables, state transition |
| 04 | STLC & Test Artifacts | Phases, strategy vs plan, scenario vs case, RTM |
| 05 | Defect Lifecycle & Reporting | **Severity vs priority**, actionable bug reports |
| 06 | White-Box Testing | Coverage criteria, why 100% statement coverage is weak |
| 07 | Performance Testing | **Load vs stress vs spike**, percentiles, saturation curve |
| 08 | Security, Usability & Accessibility | **IDOR**, what automates vs needs humans |

---

## 02_PLAYWRIGHT — Track 1 (31) ✅

### 01_BASICS (8)
| # | Note | Covers |
|---|------|--------|
| 01 | What Is Playwright | Why Playwright, vs Selenium, architecture contrast |
| 02 | Browser, Context, Page | Three-level model, isolation, parallel foundation |
| 03 | Locators | Role → testid → text → CSS, why CSS breaks |
| 04 | Actions | High vs low-level, auto-wait behaviour |
| 05 | Assertions | Web-first assertions, retry, what to assert |
| 06 | Debugging | Trace viewer, video, inspector, replaying CI failures |
| 07 | **Waiting Strategies** | 5 actionability checks, **where auto-waiting stops** |
| 08 | Handling Complex UI | iframes, dialogs, tabs, uploads, downloads, shadow DOM |

### 02_FRAMEWORK (11)
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

### 03_API_TESTING (3)
| # | Note | Covers |
|---|------|--------|
| 01 | APIRequestContext | Built-in client, **request vs page.request** |
| 02 | HTTP Methods | REST semantics, idempotency, status codes |
| 03 | Response Validation | toMatchObject over toEqual, schema validation |

### 04_TYPESCRIPT (4)
| # | Note | Covers |
|---|------|--------|
| 01 | TypeScript Basics | Types, unions for test data, unknown vs any |
| 02 | Advanced Interfaces | Partial/Pick/Omit/Record, derive don't duplicate |
| 03 | Generics | Typed API client, constraints, `as T` isn't validation |
| 04 | Async/Await | **The forgotten `await`**, no-floating-promises |

### 05_CICD_AND_EXECUTION (5)
| # | Note | Covers |
|---|------|--------|
| 01 | GitHub Actions | Workflow, caching, sharding, `if: always()` |
| 02 | Azure DevOps | Pipeline YAML, **CI=true gotcha**, JUnit publishing |
| 03 | Docker & Environments | Image/version matching, env config, fail-fast |
| 04 | Reporting & Artifacts | Reporters per audience, trace-on-first-retry |
| 05 | Tooling & AI Productivity | Codegen, UI mode, honest AI positioning |

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

## 07_INTERVIEW_PREP (3 built, 4 pending)

| Folder | Status | Covers |
|--------|:---:|--------|
| 01_SELF_INTRO | ❌ Empty | The 60-second opener |
| 02_PROJECT_STORIES | ❌ Empty | Two-track walkthrough, deeper than behavioral |
| 03_HR_AND_SALARY | ❌ Empty | CTC negotiation, notice period, why leaving |
| 04_COMPANY_SPECIFIC | ❌ Empty | Per-company research approach |
| 05_COMMUNICATION_SKILLS | ✅ | Claim → Reason → Example → Trade-off |
| 06_DESIGN_QUESTIONS | ✅ | Worked example, 5-step process |
| 07_BEHAVIORAL_STORIES | ✅ | The six stories, STAR structure |

---

## 08_QUICK_REVISION — Empty (planned)

Revision layer, deliberately deferred until the learning pass is done.
Planned as round-based cheat sheets + a living weak-spots file.

---

## What's NOT Built Yet

| Gap | Priority | Notes needed |
|-----|:---:|:---:|
| **Practical / live rounds** | 🔴 High | ~6-8 |
| ├ Coding problems (Java + JS/TS) | | problem bank |
| ├ SQL query practice | | problem bank |
| ├ Code review exercise | | worked examples |
| ├ Live UI test writing | | approach + examples |
| ├ Live API testing | | approach |
| └ Take-home assignment | | scoping guide |
| **Interview prep gaps** | 🟡 Medium | 4 |
| └ Self-intro, project stories, HR/salary, company-specific | | |
| **Revision layer** (`08_QUICK_REVISION`) | 🟡 Medium | ~7 |
| └ After the learning pass | | |
| **Gauge, Python** | ⚪ Deferred | — |
| └ On the resume; Sagar's call to revisit later | | |

---

## Verified Metrics — Use Only These

| Metric | Value | Belongs to |
|--------|-------|-----------|
| Total experience | 3.6+ years | — |
| Automated scenarios | 300+ | CIP_LEGACY |
| Regression coverage | 90% | NielsenIQ overall |
| CI/CD feedback loop | **40 → 12 min** | BASES_PLAYWRIGHT |
| Flaky rate | **under 2%** (via Fluent Waits) | CIP_LEGACY |
| AI efficiency gain | 30% | BASES_PLAYWRIGHT |
| Awards | 2x Quarterly Award | NielsenIQ |

Source of truth: `Resume & Coverletters/Resume.tex`.
Metrics appear in ~11 notes only — never scatter them further.

---

## Note Format

Every note follows:
1. What it is · 2. Why it matters · 3. 30-second answer · 4. 2-minute answer
5. Real project example · 6. Interview Q&A · 7. Weak areas · 8. Key takeaway

Plus comparison tables, checklists, and code patterns where useful.
Template: `00_INDEX/NOTE_TEMPLATE.txt`
