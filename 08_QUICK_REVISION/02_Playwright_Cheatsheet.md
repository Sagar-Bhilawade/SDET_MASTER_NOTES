# Playwright Cheat Sheet — Technical Round (Track 1)

> Dense recall for `02_PLAYWRIGHT`. Learn from the full notes; fire from this. Your lead
> track — this is the one you must have cold.

---

## Architecture (the 30-second answer)

**Browser → Context → Page.**
- **Browser:** one browser process.
- **Context:** an isolated session — own cookies, storage, cache. Like a fresh incognito window. **This is what makes parallel tests safe** — each gets its own context, zero bleed.
- **Page:** a tab within a context.

*Why it beats Selenium: talks to the browser over one WebSocket connection (DevTools protocol), not HTTP round-trips per command → faster, and it can wait on browser events natively → auto-waiting.*

**Three reasons over Selenium (have these ready):** (1) auto-waiting eliminates most flakiness, (2) true browser isolation via contexts for safe parallelism, (3) one framework for UI + API + network interception.

---

## Locators (priority order)

`getByRole` → `getByLabel`/`getByText` → `getByTestId` → CSS/XPath (last resort).

**Why role/label first:** resilient to styling and DOM refactors, accessible-by-default, match user intent. **Why CSS breaks:** tied to structure/classes that change on every restyle.

Locators are **lazy** — they resolve when acted on, not when declared. That's why they auto-retry.

---

## Auto-waiting & actionability (the depth question)

Before an action, Playwright checks the element is: **Attached · Visible · Stable · Enabled · Receives events** (not obscured).

**Where auto-waiting STOPS (this is the senior signal):** it waits for the element, **not for your app's state**. It won't wait for a spinner to finish, an API call to land, or data to populate — only for the element to be actionable. For app state you use **web-first assertions** (`toBeVisible`, `toHaveText`) which auto-retry, or `waitForResponse`. Never `waitForTimeout`.

---

## Assertions

**Web-first** (`await expect(locator).toBeVisible()`) auto-retry until timeout. **Non-retrying** (`expect(value).toBe()`) check once. Use web-first for anything the UI does asynchronously.

Assert **user-visible outcomes**, not implementation. A negative test must assert the bad thing *didn't* happen (stayed on page, error shown).

---

## Framework layer

| Concept | One-liner |
|---------|-----------|
| **POM** | Encapsulate page interactions; a selector change touches one file. |
| **Fixtures over hooks** | Compose, lazy-evaluate (test only pays for what it uses), typed. Cost: learning curve. |
| **Hooks** | `beforeEach/All`; better than fixtures for pure ordering/setup with no injected value. |
| **Config** | `playwright.config.ts` — `forbidOnly` (fail CI if `.only` left in), retries, reporters, projects. |
| **Storage state** | Save auth once, reuse → skip UI login every test. **Gap:** login flow no longer exercised → keep one dedicated UI login test. |
| **Network mocking** | `route`/`fulfill`/`abort`. Mock **external** deps, not your own app. |
| **Parallel** | Workers + sharding. Requires test isolation (own data, no shared state). |

---

## CI/CD (the 40→12 story)

- **Sharding** across workers = most of the speedup.
- Remove wasted `waitForTimeout`; web-first assertions instead.
- `trace: 'on-first-retry'` — traces only when needed, cheap.
- Azure DevOps: **`CI=true` gotcha** — CI mode changes defaults (retries, workers); test locally with it set.
- `if: always()` (GH Actions) so reports publish even on failure.

---

## API testing in Playwright

`request` fixture / `APIRequestContext`. `request` (no browser) for pure API; `page.request` shares the page's cookies. `toMatchObject` over `toEqual` (survives new fields). Read the resource back to prove persistence.

---

## TypeScript gotchas

- **The forgotten `await`** — #1 bug. `no-floating-promises` lint catches it.
- `??` (nullish) vs `||` (falsy) — `0 || x` bites.
- `Partial`/`Pick`/`Omit`/`Record` — derive test-data types, don't duplicate.
- `as T` is a cast, **not** validation — it doesn't check anything at runtime.

---

## AI positioning (say it exactly this way)

Codegen + Copilot for scaffolding and maintenance, ~30% efficiency gain. **Practical CLI workflows.** Never claim self-healing selectors or agent architectures — you didn't run those.

---

## Rapid-fire Q&A

- **"Playwright vs Cypress?"** → Multi-tab/context, true parallelism, cross-browser, UI+API in one. Cypress has nicer DX. See `06/01`.
- **"How do you handle iframes?"** → `frameLocator`. Shadow DOM → Playwright pierces it automatically with normal locators.
- **"Auth without logging in every test?"** → storage state (+ the coverage-gap caveat).
- **"How is it not flaky in parallel?"** → context isolation + own data per test + web-first waits.

---

## Key takeaway

> **Browser/Context/Page, locator hierarchy, and — the one most people miss — where
> auto-waiting stops.** This is your lead track; every answer here should be reflexive.
> The senior differentiator is naming the *limits* (auto-wait boundary, storage-state gap)
> and the *costs* (fixtures' learning curve), not just the features.
