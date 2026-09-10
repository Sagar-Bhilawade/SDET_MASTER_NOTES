# What Is Playwright

## 1. What It Is

Playwright is a modern web automation and test framework. For Sagar's current track, it
means Playwright Test with TypeScript: UI E2E automation, API-assisted setup/cleanup,
fixtures, assertions, reports, traces, and CI execution in one stack.

It is not only a Selenium replacement. It is a complete testing toolchain: browser
automation, test runner, fixtures, projects, web-first assertions, API client, and
debugging artifacts.

## 2. Why It Matters In Interviews

Interviewers ask this to check whether you know why Playwright fits the project, not just
whether you can write `page.click()`.

Strong answer:

- Explain browser/context/page.
- Explain locators and auto-waiting.
- Explain web-first assertions.
- Explain trace debugging.
- Compare honestly with Selenium/Cypress.
- Connect it to BASES_PLAYWRIGHT and CI speed.

## 3. Complete Topic Breakdown

| Area | Playwright gives | Interview signal |
|---|---|---|
| Browser support | Chromium, Firefox, WebKit | Cross-browser testing from one API |
| Test runner | `test`, fixtures, hooks, projects | Framework-level design |
| Isolation | Browser contexts | Parallel and multi-user testing |
| Locators | Role, label, text, test id, CSS/XPath | Stable element strategy |
| Auto-waiting | Actionability checks before actions | Less timing code |
| Assertions | Auto-retrying web assertions | Stable validation |
| Debugging | Trace, video, screenshots, UI mode | CI failures are explainable |
| API support | `request`, `page.request` | API setup/cleanup with UI tests |
| CI support | reporters, artifacts, sharding | Faster feedback loop |

### Best fit

- Dynamic modern web apps.
- Critical E2E journeys.
- Regression smoke suites.
- Multi-browser checks.
- Multi-user role flows.
- UI tests that need API data setup.

### Not the best fit

- Mobile native testing.
- Heavy load/performance testing.
- Dedicated Java-only API suites where RestAssured is already the standard.
- Pure unit tests owned by developers.

## 4. Manual Testing Angle

Playwright does not replace manual QA thinking. Manual testing decides risk, scenarios,
data, exploratory areas, usability judgement, and release confidence. Playwright is the
automation layer for stable, repeatable checks.

## 5. Automation Angle

Use Playwright for repeated behavior that must be protected:

- login smoke,
- dashboard/load smoke,
- create/edit/delete critical flows,
- role-based access,
- cross-browser smoke,
- regression checks,
- API-assisted setup and cleanup before UI verification.

The trade-off is cost. UI E2E tests are slower and more fragile than API/unit tests, so
do not automate every manual case at UI level.

## 6. Exact Surface To Remember

```typescript
import { test, expect } from '@playwright/test';

test('dashboard opens', async ({ page }) => {
  await page.goto('/dashboard');
  await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
});
```

```bash
npm init playwright@latest
npx playwright test
npx playwright test --headed
npx playwright test --debug
npx playwright show-report
npx playwright show-trace trace.zip
```

Key terms:

- Browser = browser process.
- Context = isolated browser profile/session.
- Page = tab inside a context.
- Locator = lazy element query.
- Web-first assertion = assertion that auto-retries.

## 7. Real Project Example

> "In BASES_PLAYWRIGHT, I use Playwright with TypeScript for UI E2E automation. The
> framework uses fixtures, API-assisted setup/cleanup, role/test-id locators, trace
> artifacts, and CI execution through Azure DevOps/GitHub Actions. One result was reducing
> the CI feedback loop from 40 minutes to 12."

## 8. Common Mistakes / Weak Areas

- Saying Playwright is only Selenium with new syntax.
- Saying auto-waiting means no waits are ever needed.
- Treating UI automation as a replacement for API/unit/manual testing.
- Claiming Playwright removes all flakiness.
- Overclaiming mobile native or load testing.
- Saying Playwright MCP as project experience. Use: Playwright CLI workflows and GitHub
  Copilot Chat.

## 9. 30-Second Answer

Playwright is a modern web testing framework. I use Playwright with TypeScript for UI E2E
automation, API-assisted setup/cleanup, fixtures, locators, web-first assertions, trace
debugging, and CI execution. Its main advantages over Selenium are context isolation,
better auto-waiting, modern locators, parallel execution, and strong debugging artifacts.

## 10. 2-Minute Answer

I use Playwright as a complete test framework, not only as a browser driver. It gives the
runner, fixtures, projects, assertions, API request context, traces, reports, and browser
automation. The core architecture is browser, context, and page. Contexts isolate cookies,
storage, permissions, viewport, and auth state, which makes parallel and multi-user tests
cleaner.

The main practical wins are stable locators, actionability auto-waiting, auto-retrying
assertions, trace viewer, and API setup before UI verification. I still choose the right
test level: API for service/data validation, UI for user journeys, and manual testing for
exploration and judgement.

## 11. Interview Q&A

### Q: Why Playwright over Selenium?

Stronger auto-waiting, context isolation, modern locators, built-in trace debugging,
parallel execution, and API support. Selenium still has a large ecosystem and is valid
for legacy Java/grid setups.

### Q: Is Playwright only for UI?

No. It is mainly used for web UI E2E, but it also has `APIRequestContext`. In my project,
API calls help setup and cleanup data for UI tests.

### Q: Can Playwright remove all flaky tests?

No. It reduces timing flakiness, but failures can still come from data, environment,
network, bad selectors, unstable requirements, or weak assertions.

### Q: When would you not choose Playwright?

For mobile native apps, heavy load testing, or a dedicated Java-only API suite where
RestAssured is already mature.

## 12. Quick Revision Points

- Playwright = runner + browser automation + assertions + artifacts.
- Browser/context/page is the base architecture.
- Context isolation supports parallel and multi-user tests.
- Auto-waiting helps timing, not data/environment issues.
- Say UI E2E with API-assisted setup/cleanup.
