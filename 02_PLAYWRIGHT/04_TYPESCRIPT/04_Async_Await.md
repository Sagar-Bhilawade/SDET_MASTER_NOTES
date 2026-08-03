# TypeScript - Async/Await: Promises in Playwright

## What This Covers

How asynchronous code works in Playwright — and the single most common bug in all of Playwright test code.

Almost every Playwright call returns a Promise. Understanding what that means is the difference between tests that work and tests that fail mysteriously.

---

## Why It Matters (Interview Context)

This is a **high-yield interview topic** because there's one specific mistake that causes a large share of confusing Playwright failures, and being able to name it immediately marks you as someone who has actually debugged real suites.

Interviewers ask about async because:
1. Every Playwright API is async — you can't avoid it
2. The missing-`await` bug produces symptoms that look like something else entirely
3. It separates people who understand the model from people who copy `await` by pattern

---

## Your 30-Second Interview Answer

> "Nearly everything in Playwright returns a Promise, so almost every call needs `await`. The bug I'd flag first is a forgotten `await` — the test doesn't fail there, it carries on and fails somewhere later with a confusing error, or worse, passes while asserting nothing. Enabling the `no-floating-promises` lint rule catches it at compile time, which is the fix I'd put in place rather than relying on catching it in review."

---

## Your 2-Minute Deep Answer

> "The model is straightforward; the failure mode is what matters.
>
> **The basics:**
>
> ```typescript
> await page.goto('/login');
> await page.getByTestId('email').fill('test@example.com');
> await expect(page.getByTestId('welcome')).toBeVisible();
> ```
>
> Every one of those returns a Promise. `await` pauses until it resolves.
>
> ---
>
> **The bug that matters — the forgotten `await`:**
>
> ```typescript
> // ❌ Missing await
> test('User logs in', async ({ page }) => {
>   await page.goto('/login');
>   page.getByTestId('email').fill('test@example.com');    // ← no await
>   await page.getByRole('button', { name: 'Login' }).click();
>   await expect(page).toHaveURL('/dashboard');
> });
> ```
>
> What actually happens: the fill is *started* but not waited for. Execution moves straight to the click. Sometimes the fill completes first and the test passes. Sometimes it doesn't and the click submits an empty form.
>
> **That's an intermittent failure caused by a typo.** It looks exactly like flakiness. People spend hours investigating timing, adding waits, raising timeouts — and the actual cause is one missing keyword.
>
> **The worse version — on an assertion:**
>
> ```typescript
> expect(page.getByTestId('total')).toHaveText('$99');    // ← no await
> ```
>
> This creates a Promise and discards it. The assertion never actually runs. **The test passes regardless of what's on the page.** It's counted in your coverage and verifies nothing — which is the silent test debt problem in its purest form.
>
> **The fix isn't discipline, it's tooling:**
>
> ```json
> // .eslintrc — @typescript-eslint/no-floating-promises
> "@typescript-eslint/no-floating-promises": "error"
> ```
>
> This makes an un-awaited Promise a compile-time error. That's the right answer in an interview — not "I'm careful about it", because everyone forgets one eventually. Machines should enforce what machines can.
>
> ---
>
> **What does NOT need `await`:**
>
> ```typescript
> const emailInput = page.getByTestId('email');   // locator — no await
> await emailInput.fill('test@example.com');      // action — await
> ```
>
> Locators are lazy. Creating one does nothing; it's a description of how to find an element, resolved when you act on it. That's why you can define locators in a page object constructor without `await`, and it's a common point of confusion.
>
> ---
>
> **Running things in parallel:**
>
> ```typescript
> // Sequential — 3 seconds
> const users    = await api.get<User[]>('/api/users');
> const projects = await api.get<Project[]>('/api/projects');
> const settings = await api.get<Settings>('/api/settings');
>
> // Parallel — 1 second
> const [users, projects, settings] = await Promise.all([
>   api.get<User[]>('/api/users'),
>   api.get<Project[]>('/api/projects'),
>   api.get<Settings>('/api/settings'),
> ]);
> ```
>
> Worth using when setup makes several independent calls. `Promise.all` rejects as soon as any one fails; `Promise.allSettled` waits for all and reports each — useful in cleanup where you want every deletion attempted regardless of individual failures.
>
> ---
>
> **The waitFor + action race:**
>
> A genuinely tricky case worth knowing:
>
> ```typescript
> // ❌ Race — the response may arrive before you start listening
> await page.getByRole('button', { name: 'Save' }).click();
> await page.waitForResponse('**/api/save');
>
> // ✅ Start listening first, then act
> const responsePromise = page.waitForResponse('**/api/save');
> await page.getByRole('button', { name: 'Save' }).click();
> const response = await responsePromise;
> ```
>
> Set up the listener *before* triggering the action, then await it after. On a fast API the first version intermittently misses the response entirely — another failure that presents as flakiness.
>
> ---
>
> **Loops:**
>
> ```typescript
> // ❌ forEach doesn't await — the callback is async, forEach ignores it
> items.forEach(async (item) => { await api.delete(`/api/items/${item.id}`); });
>
> // ✅ for...of awaits properly
> for (const item of items) { await api.delete(`/api/items/${item.id}`); }
>
> // ✅ or parallel where order doesn't matter
> await Promise.all(items.map(item => api.delete(`/api/items/${item.id}`)));
> ```
>
> `forEach` with an async callback is a classic — it fires everything off and returns immediately, so the surrounding code continues before any deletion completes. Usually surfaces as cleanup that doesn't happen."

---

## The Common Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Missing `await` on action | Intermittent, looks like flakiness | `no-floating-promises` lint |
| Missing `await` on assertion | **Test passes, verifies nothing** | Same lint rule |
| `await` on a locator | Harmless but confused | Locators are lazy |
| `waitForResponse` after action | Intermittently misses response | Listen first, await after |
| `forEach` with async callback | Work doesn't complete | `for...of` or `Promise.all` |
| Sequential independent calls | Slow setup | `Promise.all` |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "The async issue I'd flag as most valuable to know is the forgotten `await`, because it doesn't fail where you'd expect. On an action it produces intermittent failures that look exactly like flakiness — people investigate timing and add waits when the actual cause is a missing keyword. On an assertion it's worse: the assertion never runs and the test passes unconditionally. It's counted in coverage and verifies nothing.
>
> The fix I'd argue for isn't care, it's the `no-floating-promises` lint rule. Everyone forgets an `await` eventually, so relying on review or discipline doesn't scale. Making it a compile error means it can't reach the suite. That's the general principle I'd apply — automate what machines can enforce rather than catching it repeatedly in review.
>
> The other one worth knowing is the `waitForResponse` ordering. Set the listener up before the click, then await it afterwards. Do it the intuitive way round and on a fast API you intermittently miss the response — another failure that presents as flakiness and isn't.
>
> Coming from Java, the mental model transferred reasonably — it's closer to CompletableFuture than to thread-blocking, but the single-threaded event loop took some adjusting to."

---

## Common Interview Questions & Answers

### Q1: "What happens if you forget an `await` in Playwright?"

**A:** "Depends where. On an action, the operation starts but isn't waited for, so execution races ahead — that produces intermittent failures that look like flakiness. On an assertion it's more dangerous: the assertion never runs and the test passes regardless of the page state. That's a test in your coverage count that verifies nothing."

### Q2: "How do you prevent that?"

**A:** "The `no-floating-promises` ESLint rule makes it a compile-time error. That's the right fix rather than relying on discipline, because everyone misses one eventually. It's the same principle I'd apply to any repeated review comment — if a machine can enforce it, it shouldn't be a human's job."

### Q3: "Do locators need `await`?"

**A:** "No — locators are lazy. `page.getByTestId('email')` just describes how to find an element; it doesn't touch the page. Resolution happens when you act on it or assert against it. That's why you can define locators in a page object constructor without await, which surprises people coming from Selenium where `findElement` executes immediately."

### Q4: "How would you speed up a test making several API calls?"

**A:** "`Promise.all` if they're independent — three sequential calls at a second each become one second. I'd use `allSettled` for cleanup instead, since I want every deletion attempted even if one fails, rather than short-circuiting on the first error."

### Q5: "You need to verify an API call fires when a button is clicked. How?"

**A:** "Set up `waitForResponse` before the click, then await the promise after. Doing it in the intuitive order — click then wait — races: on a fast API the response arrives before you start listening and the wait times out. That's a subtle one that presents as flakiness and is actually an ordering bug."

---

## Weak Areas to Watch

❌ **Don't say "I just remember to await"**
→ The lint rule is the real answer. Everyone forgets one.

❌ **Don't await locators**
→ Shows a shaky grasp of laziness.

❌ **Don't use `forEach` with async callbacks**
→ It doesn't wait. Classic and easy to miss.

❌ **Don't put `waitForResponse` after the action**
→ Intermittent by construction.

❌ **Don't run independent calls sequentially**
→ `Promise.all` is free speed.

---

## Key Takeaway for Interview

The forgotten `await` is the highest-value thing to know here, because of how it *presents*.

The line that lands:

> **"A missing `await` on an action creates intermittent failures that look exactly like flakiness — people investigate timing when the cause is one missing keyword. On an assertion it's worse: the assertion never runs and the test passes unconditionally."**

And the fix that shows systems thinking:

> **"The answer isn't being careful — it's the `no-floating-promises` lint rule. Everyone forgets one eventually, so make it a compile error rather than a review responsibility."**

---

## Checklist

✅ `@typescript-eslint/no-floating-promises` enabled as an error
✅ Every action and assertion awaited
✅ Locators not awaited (they're lazy)
✅ `waitForResponse` set up before the triggering action
✅ `for...of` or `Promise.all` instead of async `forEach`
✅ Independent calls parallelized with `Promise.all`
✅ `allSettled` for cleanup where failures shouldn't short-circuit

---

**Remember:** An un-awaited assertion is a test that can never fail. 🎯
