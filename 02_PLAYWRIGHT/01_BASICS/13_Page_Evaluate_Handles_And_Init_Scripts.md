# Page Evaluate, Handles, And Init Scripts

## 1. What It Is

Playwright test code runs in Node.js, but the application runs inside the browser. `page.evaluate()` is the bridge that lets us run JavaScript inside the browser page and bring a serializable result back to the test.

Handles are references to browser-side objects. Init scripts are scripts injected before the page's own JavaScript runs.

The key idea: **test code context and browser page context are separate worlds**. Strong Playwright engineers know when to cross that boundary, and when crossing it will make a test less realistic.

## 2. Why It Matters In Interviews

Interviewers ask this topic to check whether you only know simple locator syntax or whether you understand how Playwright actually controls a page.

They may ask:

- Why can a variable in test code not be used directly inside `page.evaluate()`?
- When would you use `page.evaluate()` instead of a locator assertion?
- What is a `JSHandle`?
- Why is clicking through DOM JavaScript weaker than `locator.click()`?
- What is `addInitScript()` used for?

A weak answer says: "evaluate executes JavaScript." A strong answer says: "evaluate runs inside the browser context, so I use it only for browser-only state, diagnostics, or controlled setup. For real user behavior, I still prefer locators, actions, and web-first assertions."

## 3. Complete Topic Breakdown

| Area                        | What It Does                                                         | Interview Point                                               |
| --------------------------- | -------------------------------------------------------------------- | ------------------------------------------------------------- |
| `page.evaluate()`         | Runs JavaScript in the browser page and returns a serializable value | Test context and browser context are separate                 |
| Passing arguments           | Sends explicit values from test code into browser-side function      | Variables are not magically shared                            |
| `locator.evaluate()`      | Runs JavaScript against one matched element                          | Useful for focused DOM inspection, not normal user assertions |
| `page.evaluateHandle()`   | Returns a handle to a browser-side object                            | Use when the returned object cannot be serialized directly    |
| `JSHandle`                | A reference to an object inside the browser                          | Must be used carefully and disposed when no longer needed     |
| `page.addInitScript()`    | Runs script before page scripts                                      | Useful for controlled browser setup                           |
| `context.addInitScript()` | Adds the same init behavior for every page in a context              | Useful when all pages need the same setup                     |

Important boundary:

- Test-side variables live in Node.js.
- Browser-side code lives inside the page.
- `window`, `document`, DOM nodes, and browser storage live in the browser.
- Values must be passed into `evaluate()` explicitly.
- Only serializable values come back directly.

## 4. Manual Testing Angle

Manual testers validate behavior as a user sees it. `page.evaluate()` is not a user action; it is a technical shortcut into the browser.

So for interview framing, do not present evaluate as proof that a feature works. Use it as supporting help:

- checking browser-side state during debugging
- confirming storage/cookie/local browser values
- setting up controlled browser conditions
- collecting diagnostics when the UI does not expose the state clearly

The final confidence should still come from visible behavior, business assertions, and realistic user flow.

## 5. Automation Angle

In automation, prefer Playwright's user-level APIs first:

- `locator.click()` instead of DOM `element.click()`
- `locator.fill()` instead of setting input value by JavaScript
- `expect(locator).toHaveText()` instead of reading text with JavaScript for normal UI checks
- `expect(page).toHaveURL()` instead of only reading `document.location.href`

Use evaluate when the information is truly browser-side or when controlled setup is cleaner than forcing the UI:

- reading `localStorage` for debugging or setup validation
- checking browser globals
- stubbing browser APIs before the application loads
- inspecting custom DOM state that is not user-visible

Trade-off: evaluate can bypass actionability, accessibility, and real user behavior. It can create tests that are green but do not prove the user journey.

## 6. Exact Surface To Remember

Page evaluate:

```typescript
const href = await page.evaluate(() => document.location.href);
```

Pass test-side data explicitly:

```typescript
const userId = '123';

const value = await page.evaluate(id => {
  return window.localStorage.getItem(`user-${id}`);
}, userId);
```

Locator evaluate:

```typescript
const buttonText = await page
  .getByRole('button', { name: 'Save' })
  .evaluate(button => button.textContent);
```

Evaluate handle:

```typescript
const buttonHandle = await page.evaluateHandle(() => {
  return document.querySelector('button');
});

const text = await buttonHandle.evaluate(button => button?.textContent);
await buttonHandle.dispose();
```

Page init script:

```typescript
await page.addInitScript(value => {
  Math.random = () => value;
}, 42);
```

Context init script:

```typescript
await context.addInitScript(() => {
  window.localStorage.setItem('featureFlag', 'enabled');
});
```

## 6A. Day-To-Day Browser-Side Toolkit (what `document.querySelector` and friends can do)

Inside `page.evaluate()` you have the full browser API, not just `querySelector`. `document.querySelector` is only the entry point.

**The DOM query family**

| Call                                  | Returns                                   | Watch out                                                                  |
| ------------------------------------- | ----------------------------------------- | -------------------------------------------------------------------------- |
| `document.querySelector(css)`       | First match or `null`                   | Only the first match; `null` crashes if you chain without `?.`        |
| `document.querySelectorAll(css)`    | `NodeList` (static)                     | Not an array: wrap with `Array.from(...)` before `.map/.filter`        |
| `document.getElementById(id)`       | One element or `null`                   | Fast, but ids are often dynamic                                            |
| `el.closest(css)`                   | Nearest ancestor that matches             | Great for "find the row of this cell"                                      |
| `el.matches(css)`                   | `true/false`                            | Test an element against a selector                                         |
| `el.getAttribute('x')`              | Attribute string                          | Differs from the live property (`el.value`, `el.checked`)              |
| `el.textContent` vs `el.innerText`  | All text vs rendered, visible text        | `innerText` respects CSS (hidden text excluded); `textContent` doesn't |
| `getComputedStyle(el).color`        | Final applied style                       | Returns resolved values (`rgb(...)`), not what the CSS file says         |
| `el.getBoundingClientRect()`        | Position and size                         | Relative to viewport, changes on scroll                                    |
| `document.activeElement`            | Currently focused element                 | Handy for keyboard/tab-order debugging                                     |
| `el.shadowRoot?.querySelector(css)` | Inside an open shadow DOM                 | `document.querySelector` does **not** pierce shadow DOM; Playwright locators do |

**Recipes I can actually use**

```typescript
// Row count (but prefer: await locator.count())
const rows = await page.evaluate(() => document.querySelectorAll('table tr').length);

// All link targets on a page (quick broken-link input)
const hrefs = await page.evaluate(() =>
  Array.from(document.querySelectorAll('a')).map(a => a.href));

// Broken images
const broken = await page.evaluate(() =>
  Array.from(document.images).filter(i => !i.complete || i.naturalWidth === 0).map(i => i.src));

// Duplicate ids (accessibility sanity)
const dupes = await page.evaluate(() => {
  const ids = Array.from(document.querySelectorAll('[id]')).map(e => e.id);
  return ids.filter((id, i) => ids.indexOf(id) !== i);
});

// Horizontal overflow on mobile viewport (responsive bug)
const overflow = await page.evaluate(() =>
  document.documentElement.scrollWidth > document.documentElement.clientWidth);

// Navigation timing / page weight hints
const nav = await page.evaluate(() => {
  const n = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming;
  return { domReady: n.domContentLoadedEventEnd, load: n.loadEventEnd };
});

// Read app state / analytics layer
const layer = await page.evaluate(() => (window as any).dataLayer);

// Reset browser state between debug runs
await page.evaluate(() => { localStorage.clear(); sessionStorage.clear(); });

// Element-level: computed style, scroll size
const color = await page.locator('.price').evaluate(el => getComputedStyle(el).color);
```

**Better Playwright-native equivalent exists for many of these:** `locator.count()`, `expect(locator).toHaveCSS()`, `expect(locator).toBeFocused()`, `locator.scrollIntoViewIfNeeded()`, `context.cookies()` (also returns httpOnly cookies, which `document.cookie` cannot see), `context.storageState()`. Reach for evaluate only when none of these fit.

**Init-script recipes**

```typescript
// Catch uncaught JS errors the app throws, then read them in the test
await page.addInitScript(() => {
  (window as any).__errors = [];
  window.addEventListener('error', e => (window as any).__errors.push(e.message));
});
// ... later
const errors = await page.evaluate(() => (window as any).__errors);

// Freeze randomness / force a feature flag before the app boots
await page.addInitScript(() => { Math.random = () => 0.5; });
await context.addInitScript(() => localStorage.setItem('featureFlag', 'on'));

// Kill animations (reduces flakiness) -- must wait for DOM, head does not exist yet at init time
await page.addInitScript(() => {
  document.addEventListener('DOMContentLoaded', () => {
    const s = document.createElement('style');
    s.textContent = '*{transition:none!important;animation:none!important}';
    document.head.appendChild(s);
  });
});
```

**Chrome DevTools console helpers (for locating and debugging by hand, not for test code):** `$('css')` = first match, `$$('css')` = all matches as an array, `$x('//xpath')` = XPath, `$0` = element currently selected in the Elements tab, `copy(obj)` = copy to clipboard. Running Playwright with `--debug` / `PWDEBUG=1` also exposes `playwright.locator('...')` and `playwright.selector($0)` in the console to test a locator live.

**Rules that keep this safe**

- Anything that *changes* the page from evaluate (`el.remove()`, `el.click()`, setting `.value`) is a diagnostic shortcut, not test proof.
- Return plain data (strings, numbers, arrays of those), never DOM nodes.
- A first `querySelector` can run before the element exists: evaluate does **not** auto-wait. Wait with a locator first, then evaluate.

## 7. Real Project Example

In the BASES_PLAYWRIGHT track, I would not use `page.evaluate()` to click buttons, fill forms, or validate normal page text. That would bypass the same behavior a real user depends on.

A good use would be a focused setup or diagnostic case. For example, if a feature depends on browser storage or a browser flag, I can use an init script or evaluate-based check to confirm the browser-side setup. But the actual test should still validate the visible page flow with locators and assertions.

That gives speed where it helps, without weakening the proof of user behavior.

## 8. Common Mistakes / Weak Areas

| Mistake                                                | Why It Is Weak                                                        |
| ------------------------------------------------------ | --------------------------------------------------------------------- |
| Using `page.evaluate()` for every action             | Bypasses Playwright actionability and user behavior                   |
| Forgetting to pass arguments                           | Test variables are not automatically available inside browser context |
| Returning DOM nodes directly                           | DOM nodes are browser objects, not normal serializable values         |
| Keeping handles forever                                | Handles can retain browser-side objects longer than needed            |
| Using init scripts to hide real product behavior       | Makes the test artificial                                             |
| Replacing accessibility/user assertions with DOM reads | Can miss broken user-facing behavior                                  |

## 9. 30-Second Answer

`page.evaluate()` runs JavaScript inside the browser page, while my test code runs in Node.js. I use it when I need browser-only state like `window`, `document`, or storage, or for controlled setup using init scripts. But for normal automation, I prefer locators, actions, and web-first assertions because they test the page like a user and include Playwright's waiting and actionability checks.

## 10. 2-Minute Answer

Playwright has two execution contexts. The test runs in Node.js, and the application runs inside the browser. `page.evaluate()` lets me execute a function in the browser context and return a serializable result to the test. If I need to pass test data into that function, I pass it as an explicit argument because variables are not shared automatically.

For element-specific browser-side checks, Playwright also has `locator.evaluate()`. For complex browser objects that cannot be serialized, `evaluateHandle()` gives a `JSHandle`, which is a reference to an object in the browser. I use handles carefully because they keep browser-side references alive.

I treat evaluate as a sharp tool. It is useful for browser storage, diagnostics, browser globals, or setup like `addInitScript()`. But I avoid using it for normal clicks, fills, and assertions because that bypasses realistic user behavior. For interview framing, my rule is: use evaluate for browser internals; use locators and assertions to prove product behavior.

## 11. Interview Q&A

**Q: Why can I not use a test variable directly inside `page.evaluate()`?**
Because the function passed to `page.evaluate()` runs inside the browser page, not in the Node.js test scope. I pass values explicitly as arguments.

**Q: When should you use `page.evaluate()`?**
When I need browser-side information such as `window`, `document`, local storage, or a controlled browser setup. I avoid it for normal user actions.

**Q: What is a `JSHandle`?**
It is a reference to an object inside the browser context. It is useful when the object cannot be returned as a simple serialized value.

**Q: Why is `locator.click()` better than `page.evaluate(() => element.click())`?**
`locator.click()` performs Playwright's actionability checks and behaves closer to a user action. A DOM click inside evaluate can bypass visibility, stability, event readiness, and accessibility signals.

**Q: What is `addInitScript()` used for?**
It injects JavaScript before the page's own scripts run. I can use it for controlled setup, such as overriding a browser API or setting browser-side state before the app loads.

## 12. Quick Revision Points

- Test code runs in Node.js; app code runs in the browser.
- `page.evaluate()` crosses into browser context.
- Pass data explicitly into evaluate.
- Prefer locators and assertions for user behavior.
- `locator.evaluate()` is for element-specific browser-side inspection.
- `evaluateHandle()` returns a browser-side reference.
- Dispose handles when they are no longer needed.
- `addInitScript()` runs before page JavaScript.
- `evaluate` does not auto-wait or retry; wait with a locator first.
- `document.querySelector` returns the first match only and does not pierce shadow DOM; Playwright locators do.
- Prefer native APIs (`count()`, `toHaveCSS`, `context.cookies()`) before reaching for evaluate.
- Init scripts run before `document.head` exists: touch the DOM inside `DOMContentLoaded`.
- Strong answer: evaluate is useful, but it can bypass real user proof.
