# Framework Architecture: Config and Test Runner

### 1. 🚀 Tutorial Overview
The `playwright.config.ts` file is the central nervous system of your framework. It controls browser engines, parallelism, network settings, test routing (sharding), retries, and reporting.
Within the automation lifecycle, mastering the runner configuration is what allows an SDET to scale a suite from 10 tests running on a laptop to 10,000 tests running across a distributed CI/CD pipeline in minutes.
**Prerequisites:** Understanding of JSON/Object configurations, Node.js process environments, and basic CI/CD concepts.

### 2. 📌 Core Theoretical Pillars
- **Projects:** Playwright uses "Projects" to run the exact same tests across different browsers (Chromium, Firefox, WebKit), specific devices (Mobile Safari), or different environments (QA, Staging).
- **Parallelism vs Sharding:** Playwright natively runs tests in parallel using multiple Node worker processes on a single machine. Sharding distributes tests across *multiple machines* in a CI pipeline.
- **Fully Isolated Environments:** Every test gets a fresh `BrowserContext` and `Page`. The config dictates how these are spun up (e.g., standard viewport, network intercepts).

### 3. 🛠️ Step-by-Step Chronological Implementation

#### Phase 1: The Foundations (Entry Level)

**1. Basic Configuration**
A standard `playwright.config.ts` file.
```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,    // Run all tests in parallel
  retries: 1,             // Retry failing tests once
  workers: 4,             // Use 4 parallel workers locally
  reporter: 'html',       // Generate HTML report

  use: {
    baseURL: 'https://staging.example.com',
    trace: 'on-first-retry', // Record trace only when failing
    screenshot: 'only-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    }
  ],
});
```

#### Phase 2: Intermediate Configurations

**1. Multi-Environment & Multi-Browser**
Using environment variables to switch targets, and configuring multiple browsers.
```typescript
export default defineConfig({
  // Use cross-env in package.json: cross-env ENV=qa npx playwright test
  use: {
    baseURL: process.env.ENV === 'prod' 
        ? 'https://example.com' 
        : 'https://qa.example.com',
  },

  projects: [
    {
      name: 'Desktop Chrome',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 13'] },
    },
    {
      name: 'API Tests',
      testMatch: /.*api.spec.ts/, // Only run API tests in this project
    }
  ],
});
```

**2. Custom Timeout Strategies**
Controlling how long things wait before failing.
```typescript
export default defineConfig({
  timeout: 30000, // Total time per test (30s)
  expect: {
    timeout: 5000 // Time for a single expect() to pass (5s)
  },
  use: {
    actionTimeout: 10000, // Time for click(), fill(), etc (10s)
    navigationTimeout: 15000 // Time for page.goto() (15s)
  }
});
```

#### Phase 3: Professional Optimization (Senior Level)

**1. Dynamic Worker Allocation (CI vs Local)**
Don't overload a local laptop, but utilize maximum cores in CI.
```typescript
export default defineConfig({
  // process.env.CI is automatically true in GitHub Actions, GitLab, etc.
  workers: process.env.CI ? 8 : undefined, // undefined uses 50% of local logical CPU cores
  retries: process.env.CI ? 2 : 0,         // Don't retry locally to speed up debugging
});
```

**2. Multiple Reporters**
Enterprise frameworks need JUnit for CI parsing, HTML for developers, and Allure for management.
```typescript
export default defineConfig({
  reporter: [
    ['list'],                                // Console output
    ['html', { open: 'never' }],             // Playwright HTML
    ['junit', { outputFile: 'results.xml' }], // For CI parsing
    ['allure-playwright']                    // Advanced Analytics
  ],
});
```

### 4. 📊 Technical Component Encyclopedia

| Configuration Key | Type | Description |
| :--- | :--- | :--- |
| **`testDir`** | `string` | Path to the directory containing test files. |
| **`fullyParallel`** | `boolean` | If true, all tests run in parallel. If false, tests *within a single file* run sequentially. |
| **`workers`** | `number` | Maximum number of concurrent worker processes. |
| **`retries`** | `number` | Number of times to retry a failed test. |
| **`use.baseURL`** | `string` | Base URL used for all `page.goto('/path')` and `request.get('/path')`. |
| **`use.trace`** | `'on'\|'off'\|'retain-on-failure'\|'on-first-retry'` | Controls the powerful Playwright Trace Viewer generation. |
| **`use.video`** | `'on'\|'off'\|'retain-on-failure'\|'on-first-retry'` | Records a video of the test execution. |
| **`globalSetup`** | `string` | Path to a script that runs exactly once before all tests. |
| **`globalTeardown`** | `string` | Path to a script that runs exactly once after all tests complete. |

### 5. 🔄 Visual Workflow (Mermaid)

```mermaid
flowchart TD
    A[Playwright CLI Start] --> B[Parse playwright.config.ts]
    B --> C[Run Global Setup]
    C --> D[Initialize Worker 1]
    C --> E[Initialize Worker 2]
    D --> F[Run Test A (Chromium)]
    E --> G[Run Test B (Firefox)]
    F --> H[Test Fails]
    H --> I[Retry Test A]
    I --> J[Capture Trace & Screenshot]
    G --> K[Test Passes]
    J --> L[Run Global Teardown]
    K --> L
    L --> M[Generate Reporters (HTML, JUnit)]
```

### 6. 🏭 Real-World Industrial Patterns

**The Dependency Project Pattern (Playwright v1.31+)**
A massive problem in E2E was ensuring `setup` tests (like logging in and saving state) ran *before* the main tests. Playwright introduced Project Dependencies to solve this elegantly.

```typescript
// playwright.config.ts
export default defineConfig({
  projects: [
    {
      name: 'setup',
      testMatch: /.*\.setup\.ts/, // Runs login.setup.ts
    },
    {
      name: 'e2e-tests',
      testMatch: /.*\.spec\.ts/,
      dependencies: ['setup'], // GUARANTEES setup finishes before these start
      use: {
        storageState: 'playwright/.auth/user.json',
      },
    },
  ],
});
```
*Benefits:* 
- Unlike `globalSetup`, the `setup` project runs inside the standard Playwright runner, meaning you get traces, videos, and full reporting if the login itself fails!
