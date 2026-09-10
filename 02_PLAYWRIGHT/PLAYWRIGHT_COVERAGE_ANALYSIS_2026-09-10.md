# Playwright Coverage Analysis - 2026-09-10

## Purpose

This report is an approval plan before expanding the Playwright notes.

The current Playwright library is useful, but it was mostly written in the older
interview-answer style. The manual testing fundamentals have now moved to a stronger
topic-first format. Playwright should get the same treatment: every important topic
should explain the concept, interview risk, breakdown, exact syntax, real BASES_PLAYWRIGHT
example, trade-offs, common follow-ups, and quick revision points.

## Sources Checked

- Existing index: `00_INDEX/INDEX.md`
- Existing Playwright notes: `02_PLAYWRIGHT/`
- Official Playwright docs checked on 2026-09-10:
  - https://playwright.dev/docs/intro
  - https://playwright.dev/docs/writing-tests
  - https://playwright.dev/docs/locators
  - https://playwright.dev/docs/actionability
  - https://playwright.dev/docs/test-configuration
  - https://playwright.dev/docs/test-projects
  - https://playwright.dev/docs/test-fixtures
  - https://playwright.dev/docs/auth
  - https://playwright.dev/docs/test-timeouts
  - https://playwright.dev/docs/network
  - https://playwright.dev/docs/pages
  - https://playwright.dev/docs/events
  - https://playwright.dev/docs/best-practices
  - https://playwright.dev/docs/emulation
  - https://playwright.dev/docs/clock
  - https://playwright.dev/docs/accessibility-testing

## Current Playwright Library

Current count: 36 notes.

Current structure:

- `01_BASICS` - 9 notes
- `02_FRAMEWORK` - 12 notes
- `03_API_TESTING` - 5 notes
- `04_TYPESCRIPT` - 4 notes
- `05_CICD_AND_EXECUTION` - 6 notes

Strong areas already present:

- What Playwright is and why it is different from Selenium
- Browser / Context / Page architecture
- Locators and locator strategy
- Actions and auto-waiting
- Web-first assertions
- Debugging with trace/video/report
- Complex UI: iframe, dialog, tabs, upload/download, shadow DOM
- Error-message classification
- Framework design, POM, fixtures, hooks, auth storage state
- APIRequestContext and `request` vs `page.request`
- CI with GitHub Actions and Azure DevOps
- Reporting and artifacts
- TypeScript basics needed for test automation

Main weakness:

The section is broad, but many notes are not yet in the new 12-part self-contained
format. They answer interview questions, but some do not teach the full decision model
behind the feature. For 20-24 LPA rounds, that is where follow-up questions can hurt.

Structural scan result:

- Existing Playwright notes in the new 12-section fundamentals format: `0`
- Existing Playwright notes in the older interview/Q&A-heavy format: `36`

So this is not a small touch-up. The Playwright section needs a full format upgrade.

## Must-Fix Coverage Gaps

These should be added or expanded before we call the Playwright library complete.

| Priority | Topic | Why it matters | Recommendation |
|---|---|---|---|
| P0 | Playwright projects and test matrix | Interviewers ask how one framework runs Chromium/Firefox/WebKit, mobile viewports, smoke/regression, setup projects, and multiple environments. Current coverage is spread across config/parallel/auth, not one clean answer. | Add a separate framework note: `Projects_And_Test_Matrix` |
| P0 | Timeouts model | This is different from waiting. Test timeout, expect timeout, action timeout, navigation timeout, fixture timeout, and global timeout are common practical probes. | Add or expand as separate note: `Timeouts_And_Retries_Policy` |
| P0 | Global setup, teardown, and project dependencies | Auth setup is partly covered, but the official setup-project pattern needs a clean explanation: setup project creates state, dependent projects consume it, teardown/cleanup boundaries. | Expand auth practical note or add: `Global_Setup_Teardown_And_Dependencies` |
| P0 | Navigation, load states, and events | Current notes cover actions and waiting, but not enough on `page.goto`, `waitForURL`, `waitForLoadState`, popup/download/request events, and `Promise.all`/pre-wait pattern. | Add basics note: `Navigation_And_Events` |
| P0 | Emulation and browser/device coverage | Mobile viewport, device presets, permissions, geolocation, timezone, locale, color scheme, and browser channels are interview-friendly and map to compatibility testing. | Add basics/framework note: `Emulation_Devices_And_Permissions` |
| P0 | Network monitoring, mocking, HAR, and service-worker gotchas | Existing network mocking note is useful but should go deeper: monitor requests/responses, mock third-party only, modify response, HAR replay, service-worker interference. | Expand network note or split into `Network_Mocking_And_HAR` |
| P0 | Accessibility testing with Playwright | Manual fundamentals cover accessibility generally, but Playwright-specific answer should connect role locators, accessible names, keyboard flow, and `axe` scans. | Add framework/basics note: `Accessibility_Testing_With_Playwright` |
| P0 | TestInfo, attachments, and failure evidence | Reporting exists, but the practical code surface for attaching screenshots/logs, using `testInfo`, and reading artifacts should be stronger. | Expand reporting/artifacts note |
| P0 | Complete assertion matcher coverage | Current assertion note covers common assertions, but not the full Playwright matcher surface. Interviewer can ask exact differences: `toBeVisible` vs `toBeAttached`, `toHaveText` vs `toContainText`, `toBeOK`, `expect.poll`, soft assertions, custom messages. | Expand `05_Assertions` into a complete assertion guide |

## High-Value Expansions To Existing Notes

These are not brand-new gaps, but existing notes should be rewritten in the new manual-note
style.

| Existing note | Expansion needed |
|---|---|
| `01_What_Is_Playwright` | Include test runner bundle, browser support, auto-wait, isolation, parallelism, tracing, API support, and honest Selenium/Cypress trade-offs |
| `02_Browser_Context_Page` | Add lifecycle, isolation, context as browser profile, multi-user testing, resource cleanup, browser/context/page fixture mapping |
| `03_Locators` | Add accessible locator hierarchy, strictness, chaining, `filter`, `has`, `hasText`, `first/last/nth` risk, CSS/XPath fallback policy |
| `04_Actions` | Add actionability per action, `force` risk, hover/press/select/upload/download patterns, trial actions if useful |
| `05_Assertions` | Add auto-retrying vs non-retrying assertions, locator/page/API/snapshot assertions, soft assertions, anti-patterns like `isVisible()` inside normal expect |
| `06_Debugging` | Add local debug vs CI debug, inspector, UI mode, trace viewer, actionability logs, console/network inspection, artifact sharing |
| `07_Waiting_Strategies` | Separate auto-wait, web-first assertions, navigation waits, event waits, polling, and hard-wait red lines |
| `08_Handling_Complex_UI` | Expand each surface with exact syntax and failure traps: iframes, popups, dialogs, downloads, uploads, shadow DOM |
| `09_Error_Messages_And_Exceptions` | Keep as a practical rescue map, but align to 12-part format and include timeout family distinctions |
| `05_Config_Files` | Add top-level vs `use` split, `testDir`, `testMatch`, `testIgnore`, `outputDir`, `webServer`, `expect`, environment config |
| `07_Authentication_State` | Add setup project, role states, expiry, logout/login coverage gap, `.auth` gitignore, API auth boundary |
| `09_Parallel_Execution` | Add workers vs fullyParallel vs sharding vs projects; explain data isolation and cost trade-off |
| `10_Test_Runner_Features` | Add annotations, tags, grep, serial risk, parameterization, `test.describe.configure`, `test.step` and reporting value |
| `11_Visual_Testing` | Add screenshot config, snapshot path, masking dynamic elements, baseline review discipline |
| `05_CICD_AND_EXECUTION/06_Project_Setup` | Add version check/update commands, browser install choices, `npm ci`, `npx playwright install chromium --with-deps`, report/trace commands |

## Proposed New Notes

Recommended target: add 8 new notes and expand the existing 36.

This keeps the library strong without bloating it into documentation.

## Assertion Coverage Requirement

The assertion note must not stay as a short list of common matchers. It should become a
complete interview-safe assertion library.

It should cover these groups:

### Locator assertions

- `toBeAttached`
- `toBeChecked`
- `toBeDisabled`
- `toBeEditable`
- `toBeEmpty`
- `toBeEnabled`
- `toBeFocused`
- `toBeHidden`
- `toBeInViewport`
- `toBeVisible`
- `toContainText`
- `toContainClass`
- `toHaveAccessibleDescription`
- `toHaveAccessibleName`
- `toHaveAttribute`
- `toHaveClass`
- `toHaveCount`
- `toHaveCSS`
- `toHaveId`
- `toHaveJSProperty`
- `toHaveRole`
- `toHaveScreenshot`
- `toHaveText`
- `toHaveValue`
- `toHaveValues`
- `toMatchAriaSnapshot`

### Page assertions

- `toHaveScreenshot`
- `toHaveTitle`
- `toHaveURL`
- `toMatchAriaSnapshot`

### API response assertion

- `toBeOK`

### Generic non-retrying assertions

- `toBe`
- `toEqual`
- `toStrictEqual`
- `toMatchObject`
- `toContain`
- `toContainEqual`
- `toHaveLength`
- `toHaveProperty`
- `toMatch`
- `toBeTruthy`
- `toBeFalsy`
- `toBeDefined`
- `toBeUndefined`
- `toBeNull`
- `toBeNaN`
- `toBeGreaterThan`
- `toBeGreaterThanOrEqual`
- `toBeLessThan`
- `toBeLessThanOrEqual`
- `toBeCloseTo`
- `toBeInstanceOf`
- `toThrow`

### Assertion control features

- `.not`
- `expect.soft`
- custom assertion message
- `expect.configure`
- `expect.poll`
- `expect.toPass`
- `expect.extend`
- asymmetric matchers: `expect.any`, `expect.anything`, `expect.objectContaining`,
  `expect.arrayContaining`, `expect.stringContaining`, `expect.stringMatching`

Important interview distinction:

- Locator/page/API assertions are usually auto-retrying and should be preferred for UI
  state.
- Generic assertions are non-retrying and can create flaky tests if used against
  asynchronous UI state.
- `expect.poll` and `expect.toPass` are for retrying conditions that are not simple
  locator assertions.

This same "complete surface" rule should be applied to actions, locators, config, runner
features, fixtures, auth, network, and CI commands. We do not need every rare API as a
separate note, but every common interview surface should be covered with exact syntax and
trade-offs.

### `01_BASICS`

Add:

1. `10_Navigation_And_Events.md`
   - `page.goto`
   - `waitForURL`
   - `waitForLoadState`
   - `page.waitForEvent`
   - request/response/popup/download event pre-wait pattern
   - why event wait must start before the action

2. `11_Emulation_Devices_And_Permissions.md`
   - device presets
   - viewport
   - geolocation
   - permissions
   - timezone/locale
   - color scheme
   - browser channel
   - compatibility testing strategy

3. `12_Clock_And_Time_Dependent_Testing.md`
   - testing dates/timers without real waiting
   - when clock mocking is useful
   - risk of hiding real integration behavior
   - examples: token expiry, countdowns, date filters

### `02_FRAMEWORK`

Add:

1. `13_Projects_And_Test_Matrix.md`
   - browser projects
   - mobile projects
   - smoke vs regression project split
   - staging vs production config
   - setup project dependencies
   - `--project`
   - when not to run every project on every commit

2. `14_Timeouts_And_Retries_Policy.md`
   - test timeout
   - expect timeout
   - action timeout
   - navigation timeout
   - fixture timeout
   - global timeout
   - `test.slow`
   - retry policy and why retries are diagnostic, not a fix

3. `15_Global_Setup_Teardown_And_Project_Dependencies.md`
   - setup project vs global setup
   - auth setup
   - cleanup ownership
   - dependency ordering
   - failure behavior
   - what belongs in fixture vs setup project

4. `16_Accessibility_Testing_With_Playwright.md`
   - role locators and accessible names
   - keyboard navigation
   - focus checks
   - `axe` scan boundary
   - what automation cannot prove

### `03_API_TESTING`

Add:

1. `06_Network_Monitoring_Mocking_And_HAR.md`
   - `page.route`
   - `context.route`
   - `route.fulfill`
   - `route.abort`
   - wait for request/response
   - HAR replay
   - service worker issue
   - mock third-party, not your own backend by default

## Optional / Do Not Prioritize Now

These exist in official docs, but should not become core notes unless a JD asks.

| Topic | Decision |
|---|---|
| Component testing | Optional. Good to know exists, but not part of Sagar's resume/project positioning. |
| Chrome extensions | Skip unless a JD mentions extension testing. |
| WebView2 | Skip unless a JD mentions desktop/WebView apps. |
| Playwright library mode | Skip for now; Sagar is positioned around Playwright Test, not using Playwright as a raw browser automation library. |
| Playwright Agents / MCP style claims | Do not add as production experience. Keep Sagar's wording as Playwright CLI agent-style workflow and GitHub Copilot Chat. |

## Proposed Final Playwright Shape

After approval, update the Playwright section to about 44 notes:

- `01_BASICS` - 12 notes
- `02_FRAMEWORK` - 16 notes
- `03_API_TESTING` - 6 notes
- `04_TYPESCRIPT` - 4 notes
- `05_CICD_AND_EXECUTION` - 6 notes

This gives complete interview coverage while keeping the library readable.

## Work Plan After Approval

1. Rewrite existing Playwright notes into the 12-section topic-first format.
2. Add the 8 proposed missing notes.
3. Keep every example tied to BASES_PLAYWRIGHT.
4. Keep AI wording honest: GitHub Copilot Chat and Playwright CLI workflows, not self-healing or production agent architecture.
5. Add exact syntax where needed: config keys, CLI commands, Playwright APIs, common error wording.
6. Update `00_INDEX/INDEX.md` in the same pass with the new count and coverage.
7. Run a structure check so every Playwright note has the complete format.
8. Commit only after the expanded Playwright library is coherent.

## Recommendation

Do not start rewriting immediately without approval, because the new notes change the
Playwright section count and reading load.

My recommended approval path:

- Approve all P0 additions.
- Keep optional topics out for now.
- Rewrite all existing Playwright notes to match the new fundamentals format.
- Then commit Playwright as one strong milestone.
