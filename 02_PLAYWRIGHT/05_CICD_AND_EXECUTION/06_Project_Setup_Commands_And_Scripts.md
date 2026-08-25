# Playwright - Project Setup, Commands, And package.json Scripts

## What it is

This note is the practical command layer of a Playwright TypeScript framework.

It covers:

- creating a new project
- installing browsers
- running tests
- running by project, file, tag, headed/debug/UI mode
- viewing reports and traces
- common `package.json` scripts
- what each command is used for in day-to-day work

---

## Why it matters

In practical rounds, interviewers often ask commands because commands reveal whether
you have actually used the tool.

They may ask:

- "How do you create a Playwright project?"
- "How do you run only smoke tests?"
- "How do you run headed?"
- "How do you debug one test?"
- "What scripts do you keep in package.json?"
- "What is the difference between `npm install` and `npm ci`?"
- "What is `package-lock.json` for?"

You do not need to memorize every CLI flag. You need a confident daily command set.

---

## 30-second answer

> "For a new Playwright TypeScript project I start with `npm init playwright@latest`.
> That creates the config, sample tests, and installs Playwright. I run tests using
> `npx playwright test`, use `--ui` for local authoring, `--debug` for step-through
> debugging, `--headed` when I want to watch the browser, and `show-report` or
> `show-trace` for failures. In `package.json`, I keep scripts like `test`, `test:ui`,
> `test:headed`, `test:smoke`, `test:debug`, and `report` so the team runs the suite
> consistently."

---

## 2-minute answer

> "My project setup flow is simple.
>
> First I create the project using `npm init playwright@latest`, choose TypeScript, and
> create a `tests` folder. Then I configure `playwright.config.ts` with `baseURL`,
> browsers, retries, traces, reporters, and CI-specific options like `forbidOnly`.
>
> For daily execution, I do not run random long commands manually every time. I keep
> standard scripts in `package.json`. Locally I use `npm run test:ui` for authoring,
> `npm run test:headed` when I want to watch the browser, and `npm run test:debug` when
> I need the inspector. CI uses `npm ci` followed by `npx playwright install --with-deps`
> and `npm test`.
>
> The important part is consistency. Commands become part of the framework contract.
> A new team member should be able to run `npm test` or `npm run test:smoke` without
> knowing the whole config."

---

## Real project example

### Create a new Playwright TypeScript project

```bash
npm init playwright@latest
```

Typical selections:

```text
Language: TypeScript
Test folder: tests
GitHub Actions: yes, if the project needs CI scaffold
Install browsers: yes
```

If browsers were not installed:

```bash
npx playwright install
```

On Linux CI machines:

```bash
npx playwright install --with-deps
```

### Run the full suite

```bash
npx playwright test
```

This runs tests based on `playwright.config.ts`.

### Run one file

```bash
npx playwright test tests/login.spec.ts
```

### Run one test by title

```bash
npx playwright test -g "user can login"
```

### Run by tag

```bash
npx playwright test --grep @smoke
npx playwright test --grep-invert @slow
```

### Run a browser project

```bash
npx playwright test --project=chromium
```

### Run headed

```bash
npx playwright test --headed
```

Use this when you want to see the browser while the test runs.

### Debug one test

```bash
npx playwright test tests/login.spec.ts --debug
```

Use this for step-through debugging with Playwright Inspector.

### Use UI mode

```bash
npx playwright test --ui
```

Use this for local authoring and debugging with watch mode, snapshots, locator picker,
network view, and timeline.

### Record a flow with codegen

```bash
npx playwright codegen https://staging.example.com
```

Use codegen for discovery, not final framework code. It is useful to find good locators,
then refactor into page objects and meaningful assertions.

### Show HTML report

```bash
npx playwright show-report
```

### Show trace

```bash
npx playwright show-trace trace.zip
```

Use trace viewer for failures that already happened, especially CI failures.

### Reproduce flakiness

```bash
npx playwright test tests/payment.spec.ts --repeat-each=20 --workers=4
```

If a test passes once, that does not prove it is stable. Repeating with workers helps
surface timing and isolation issues.

---

## Common package.json scripts

```json
{
  "scripts": {
    "test": "playwright test",
    "test:ui": "playwright test --ui",
    "test:headed": "playwright test --headed",
    "test:debug": "playwright test --debug",
    "test:smoke": "playwright test --grep @smoke",
    "test:regression": "playwright test --grep @regression",
    "test:chromium": "playwright test --project=chromium",
    "test:ci": "playwright test",
    "report": "playwright show-report",
    "trace": "playwright show-trace"
  }
}
```

How to explain:

| Script | Use |
|---|---|
| `npm test` | Default full suite |
| `npm run test:ui` | Local authoring/debugging |
| `npm run test:headed` | Watch browser execution |
| `npm run test:debug` | Inspector step-through |
| `npm run test:smoke` | Fast confidence suite |
| `npm run test:regression` | Broader release suite |
| `npm run test:chromium` | One browser project |
| `npm run report` | Open latest HTML report |

---

## npm, npx, package.json, and package-lock.json

### `npm`

`npm` is the Node package manager. It installs dependencies and runs scripts.

```bash
npm install
npm run test:smoke
```

### `npx`

`npx` runs a package command without needing a global install. In Playwright projects,
prefer:

```bash
npx playwright test
```

Instead of depending on:

```bash
npm install -g playwright
```

### `package.json`

`package.json` defines project metadata, dependencies, and scripts.

### `package-lock.json`

`package-lock.json` locks exact dependency versions. It makes local and CI installs
repeatable.

Interview line:

> "`package.json` says what dependencies the project needs. `package-lock.json` records
> the exact resolved versions, so CI and team machines install the same dependency tree."

### `npm install` vs `npm ci`

| Command | Use |
|---|---|
| `npm install` | Local development; can update lockfile |
| `npm ci` | CI; clean, repeatable install from lockfile |

CI answer:

> "In CI I prefer `npm ci` because it installs exactly from `package-lock.json` and fails
> if the lockfile is inconsistent. That makes pipeline runs more deterministic."

---

## Common interview Q&A

### Q1: How do you create a Playwright project?

**Answer:** `npm init playwright@latest`, choose TypeScript, configure the test folder,
install browsers, then update `playwright.config.ts` for baseURL, browsers, retries,
trace, reporters, and CI behavior.

### Q2: How do you run tests?

**Answer:** `npx playwright test` or `npm test` if the script is configured.

### Q3: How do you run only one test?

**Answer:** By file path or title grep.

```bash
npx playwright test tests/login.spec.ts
npx playwright test -g "user can login"
```

### Q4: How do you run smoke tests?

**Answer:** Tag smoke tests and run with grep.

```typescript
test('user can login @smoke', async ({ page }) => {
  // test
});
```

```bash
npx playwright test --grep @smoke
```

### Q5: How do you debug a failing test?

**Answer:** Locally I use UI mode or debug mode. For CI, I use trace viewer from the
artifact.

```bash
npx playwright test --ui
npx playwright test tests/login.spec.ts --debug
npx playwright show-trace trace.zip
```

### Q6: Why keep scripts in package.json?

**Answer:** It standardizes execution. Developers and CI use the same named commands,
so no one has to remember long CLI flags.

### Q7: Should Playwright be installed globally?

**Answer:** Usually no. Keep Playwright as a project dependency and run it through `npx`
or package scripts. Global installs can drift from the version used by the project.

### Q8: What command installs browsers?

**Answer:** `npx playwright install`. On Linux CI, use `npx playwright install --with-deps`.

### Q9: What command opens the report?

**Answer:** `npx playwright show-report`.

### Q10: What command opens a trace?

**Answer:** `npx playwright show-trace trace.zip`.

---

## Weak areas to watch

- Do not say `npm install -g @playwright/cli` as the main setup answer. The normal modern
  project answer is `npm init playwright@latest` and project-local execution through
  `npx` or package scripts.
- Do not confuse UI mode and trace viewer. UI mode is for local authoring; trace viewer
  is for replaying an existing failure.
- Do not say codegen produces final framework code. It gives a draft and locator
  discovery.
- Do not run tests only with manual long commands. Good frameworks expose scripts.
- Do not forget `npm ci` for CI pipelines.
- Do not commit secrets in scripts or config. Use environment variables.

---

## Key takeaway

> Commands are the operating manual of the framework. `playwright.config.ts` defines the
> behavior, and `package.json` scripts make that behavior easy and consistent for the
> team.

The clean interview line:

> "I create with `npm init playwright@latest`, run with `npx playwright test`, debug with
> UI mode or `--debug`, inspect CI failures with trace viewer, and keep common flows in
> package scripts so the team runs tests consistently."
