# Emulation, Devices, And Permissions

## 1. What It Is

Emulation means running tests with controlled browser/device conditions: viewport, mobile
device profile, touch, user agent, geolocation, permissions, locale, timezone, color
scheme, and browser channel.

## 2. Why It Matters In Interviews

This connects Playwright to compatibility and responsive testing. Interviewers may ask
how you test mobile width, multiple browsers, location permissions, or timezone/date
behavior.

## 3. Complete Topic Breakdown

| Area | Config/use option |
|---|---|
| Device profile | `...devices['iPhone 13']` |
| Viewport | `viewport: { width, height }` |
| Touch/mobile | `isMobile`, `hasTouch` via device |
| Locale | `locale: 'en-GB'` |
| Timezone | `timezoneId: 'Asia/Kolkata'` |
| Permissions | `permissions: ['geolocation']` |
| Geolocation | `geolocation: { latitude, longitude }` |
| Color scheme | `colorScheme: 'dark'` |
| Browser channel | `channel: 'chrome'` |

## 4. Manual Testing Angle

Manual compatibility testing checks real risk: supported browser matrix, mobile layout,
timezone/date formats, language/locale, permissions, and responsive tables/modals.
Playwright can automate the repeatable slice.

## 5. Automation Angle

Use projects for repeated device/browser matrix and `test.use` for a focused override.

Trade-off: running every test across every device/browser is expensive. Run core smoke
widely, full regression selectively.

## 6. Exact Surface To Remember

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'Mobile Safari', use: { ...devices['iPhone 13'] } }
  ]
});
```

```typescript
test.use({
  viewport: { width: 390, height: 844 },
  locale: 'en-GB',
  timezoneId: 'Asia/Kolkata',
  colorScheme: 'dark'
});
```

```typescript
await context.grantPermissions(['geolocation']);
await context.setGeolocation({ latitude: 18.5204, longitude: 73.8567 });
await context.clearPermissions();
```

## 7. Real Project Example

> "For BASES_PLAYWRIGHT, I would not run every regression test on every viewport. I would
> keep the main browser in the merge pipeline and run selected critical flows across
> other browser/device projects nightly or before release."

## 8. Common Mistakes / Weak Areas

- Calling device emulation the same as real-device testing.
- Running the full suite on every browser/device every commit.
- Forgetting device presets include user agent and viewport.
- Setting geolocation without granting permission.
- Confusing browser timezone with test runner timezone.
- Ignoring responsive tables/modals.

## 9. 30-Second Answer

Playwright can emulate device/browser conditions through config projects or `test.use`:
viewport, device profiles, locale, timezone, permissions, geolocation, color scheme, and
browser channel. I use it for compatibility smoke and responsive checks, but I do not run
every test on every device every commit.

## 10. 2-Minute Answer

Emulation helps automate the repeatable part of compatibility testing. With projects, I
can run the same test suite against Chromium, Firefox, WebKit, or mobile device profiles.
For a specific test block, I can override viewport, locale, timezone, permissions, or
geolocation with `test.use`.

The trade-off is realism and cost. Emulation is not the same as real-device testing, and
wide matrices are expensive. So I choose a small cross-browser/mobile smoke set and keep
deeper runs for nightly or pre-release gates.

## 11. Interview Q&A

### Q: How do you test mobile viewport?

Use device projects such as `devices['iPhone 13']` or set `viewport` with `test.use`.

### Q: Is emulation equal to real device testing?

No. It simulates browser/device conditions, but final confidence for device-specific
issues may still need real devices.

### Q: How test geolocation?

Grant geolocation permission and set latitude/longitude on context or config.

### Q: How handle browser matrix?

Use Playwright projects and run selected smoke/regression based on risk and CI cost.

## 12. Quick Revision Points

- Emulation = controlled browser conditions.
- Use projects for matrix.
- Use `test.use` for scoped override.
- Geolocation needs permission.
- Do not run everything everywhere.
