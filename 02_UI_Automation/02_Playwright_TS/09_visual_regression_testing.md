# Playwright & TypeScript: Visual Regression Testing

### 1. 🚀 Tutorial Overview
Functional assertions (`expect(locator).toBeVisible()`) verify the DOM state, but they cannot verify if a CSS change caused a button to overlap with text, or if a global color variable broke the styling. **Visual Regression Testing (VRT)** takes screenshots of elements or whole pages and compares them pixel-by-pixel against a baseline image.
Within the automation lifecycle, VRT acts as the ultimate safety net for frontend styling and layout changes. Playwright includes this natively using the `expect(page).toHaveScreenshot()` matcher.
**Prerequisites:** Understanding of how browser rendering differs across OS platforms (macOS, Linux, Windows), and basic knowledge of Playwright assertions.

### 2. 📌 Core Theoretical Pillars
- **Baseline vs. Actual:** A visual test requires a "Baseline" (the golden image). The first time a test runs, it generates this baseline. On subsequent runs, it generates an "Actual" image, compares it to the baseline, and if differences exceed a threshold, generates a "Diff" image.
- **Platform Dependency:** Rendering engines (Chromium, WebKit) draw pixels differently on Linux than on macOS (font anti-aliasing, scrollbars). Baselines must be generated on the exact OS that will run the tests (usually the CI Linux container).
- **Masking and Flakiness:** Dynamic content (timestamps, ad banners, randomized IDs) will fail a pixel comparison every time. Playwright provides mechanisms to "mask" or hide these dynamic areas before capturing the screenshot.

### 3. 🛠️ Step-by-Step Chronological Implementation

#### Phase 1: The Foundations (Entry Level)

**1. Basic Page Screenshot Comparison**
Comparing the entire viewport.
```typescript
import { test, expect } from '@playwright/test';

test('Full page visual comparison', async ({ page }) => {
    await page.goto('https://example.com');
    
    // The first run creates: example-com-snapshot.png
    // The second run compares against it.
    await expect(page).toHaveScreenshot('example-com-snapshot.png');
});
```

**2. Component/Element Screenshot Comparison**
Instead of the whole page, capture just a specific locator. This is much less flaky and highly recommended.
```typescript
test('Component visual comparison', async ({ page }) => {
    await page.goto('https://example.com/dashboard');
    const header = page.locator('.site-header');
    
    // Compares only the header component
    await expect(header).toHaveScreenshot('site-header.png');
});
```

#### Phase 2: Intermediate Configurations

**1. Handling Dynamic Content (Masking)**
If a page contains a live clock or a dynamic chart, mask it. Playwright overlays a solid pink box over the masked elements.
```typescript
test('Visual test with masked elements', async ({ page }) => {
    await page.goto('/dashboard');
    
    await expect(page).toHaveScreenshot('dashboard.png', {
        mask: [
            page.locator('.live-clock'),
            page.locator('.dynamic-user-avatar')
        ],
        maskColor: '#FF00FF' // Optional: default is pink
    });
});
```

**2. Adjusting Thresholds**
Sometimes anti-aliasing causes a 1% pixel difference. You can loosen the strictness.
```typescript
test('Loose visual comparison', async ({ page }) => {
    const banner = page.locator('.hero-banner');
    
    await expect(banner).toHaveScreenshot('banner.png', {
        maxDiffPixels: 150,           // Allow up to 150 total pixels to be different
        maxDiffPixelRatio: 0.05,      // Or allow 5% of the total pixels to differ
        threshold: 0.2                // Per-pixel color threshold (0 strict, 1 loose)
    });
});
```

#### Phase 3: Professional Optimization (Senior Level)

**1. Full Page Scroll Screenshots**
By default, Playwright only captures the visible viewport. If you need the entire scrolling page, enable `fullPage`.
```typescript
test('Capture entire scrolling page', async ({ page }) => {
    await page.goto('/long-article');
    
    await expect(page).toHaveScreenshot('full-article.png', { 
        fullPage: true,
        // Wait for lazy-loaded images by disabling animations and scrolling down manually if needed
        animations: 'disabled' 
    });
});
```

**2. Updating Snapshots**
When the UI intentionally changes (e.g., a rebrand), tests will fail. You must update the baselines.
You do this via the CLI, not in the code:
```bash
npx playwright test --update-snapshots
# or
npx playwright test -u
```

### 4. 📊 Technical Component Encyclopedia

| Component / Matcher | Target Type | Method Signature | Code Example |
| :--- | :--- | :--- | :--- |
| **`toHaveScreenshot()`** | `Page\|Locator`| `toHaveScreenshot(name?: string, options?: {})` | `expect(page).toHaveScreenshot('img.png');` |
| **`maxDiffPixels`** | `Option` | `number` | `{ maxDiffPixels: 100 }` |
| **`maxDiffPixelRatio`**| `Option` | `number` | `{ maxDiffPixelRatio: 0.1 }` |
| **`mask`** | `Option` | `Locator[]` | `{ mask: [page.locator('.ad')] }` |
| **`animations`** | `Option` | `'allow' \| 'disabled'` | `{ animations: 'disabled' }` |
| **`caret`** | `Option` | `'hide' \| 'initial'` | `{ caret: 'hide' }` (hides blinking text cursor) |
| **`fullPage`** | `Option` | `boolean` | `{ fullPage: true }` |
| **`page.screenshot()`**| `Page` | `screenshot(options?: { path?: string })`| `await page.screenshot({ path: 'debug.png' });` (Manual capture, no comparison) |

### 5. 🔄 Visual Workflow (Mermaid)

```mermaid
flowchart TD
    A[Run Test: expect.toHaveScreenshot] --> B{Does Baseline Exist?}
    B -- No --> C[Capture & Save as Baseline]
    C --> D[Fail Test: 'New snapshot created']
    B -- Yes --> E[Capture Actual Image]
    E --> F[Compare Actual vs Baseline (pixelmatch)]
    F --> G{Difference > Threshold?}
    G -- Yes --> H[Generate Diff Image]
    H --> I[Fail Test: Visual Regression Detected]
    G -- No --> J[Pass Test]
```

### 6. 🏭 Real-World Industrial Patterns

**CI/CD Pipeline Snapshot Generation Pattern**
Never generate baselines on your local Windows/Mac machine if CI runs in Linux Docker. The pixel rendering will differ, causing instant failure in CI.

*Industrial Workflow:*
1. Make code changes locally.
2. Push to GitHub.
3. CI runs and fails because snapshots don't match.
4. Download the artifacts (Diffs and Actuals) from the CI run.
5. If the changes are expected, trigger a special GitHub Action workflow: "Update Visual Baselines".
6. This workflow runs `npx playwright test --update-snapshots` inside the Linux Docker container and automatically creates a Pull Request with the updated `.png` files.

*Global Config Tuning (playwright.config.ts):*
Instead of configuring thresholds per test, set a global standard to ensure consistency across the enterprise.
```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  expect: {
    // Global configuration for toHaveScreenshot
    toHaveScreenshot: {
      maxDiffPixelRatio: 0.05,
      animations: 'disabled', // Automatically stop CSS animations before screenshot
    },
  },
});
```
