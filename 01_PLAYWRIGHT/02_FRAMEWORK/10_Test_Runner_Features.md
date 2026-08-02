# Playwright - Test Runner Features: Steps, Tags, Annotations, Soft Assertions

## What This Covers

The test runner capabilities beyond writing test bodies — grouping, tagging, annotating, steps, and soft assertions.

These are the features that make a large suite navigable and reports readable. Most candidates know `test()` and `describe()` and nothing else.

---

## Why It Matters (Interview Context)

Two things get demonstrated here:

1. **Practical fluency** — you've worked with a suite large enough to need tagging and organization
2. **Report thinking** — `test.step` exists purely to make failures readable by someone who didn't write the test

The tagging question in particular connects to tiered CI execution, which is a strategy answer.

---

## Your 30-Second Interview Answer

> "The ones I use constantly are tags for tiered execution — smoke on PRs, full regression on merge — and `test.step` to structure long tests so a failure in the report shows which phase broke rather than just a line number. Annotations like `test.fixme` and `test.slow` handle known issues visibly instead of silently commenting tests out. And soft assertions for cases where I want to collect multiple failures in one run rather than stopping at the first."

---

## Your 2-Minute Deep Answer

> "Going through what actually earns its place:
>
> ---
>
> **TAGS — the highest-value one**
>
> ```typescript
> test('User completes checkout', { tag: ['@smoke', '@critical'] }, async ({ page }) => {
>   // ...
> });
> ```
>
> Run selectively:
> ```bash
> npx playwright test --grep @smoke
> npx playwright test --grep-invert @slow
> npx playwright test --grep "@smoke|@critical"
> ```
>
> This is what makes tiered CI possible: smoke on every PR, full regression on merge, slow suites nightly. A four-minute smoke suite on PRs plus full coverage on merge is a much better developer experience than forty minutes on everything — and it's free, unlike sharding.
>
> Tags I'd standardize on: `@smoke`, `@critical`, `@slow`, `@flaky` for quarantine.
>
> ---
>
> **`test.step` — structure inside long tests**
>
> ```typescript
> test('Complete purchase flow', async ({ page }) => {
>   await test.step('Add item to cart', async () => {
>     await page.getByRole('button', { name: 'Add to cart' }).click();
>     await expect(page.getByTestId('cart-count')).toHaveText('1');
>   });
>
>   await test.step('Enter shipping details', async () => {
>     await page.getByTestId('address').fill('123 Test St');
>     await page.getByRole('button', { name: 'Continue' }).click();
>   });
>
>   await test.step('Complete payment', async () => {
>     await page.getByTestId('card').fill('4242424242424242');
>     await page.getByRole('button', { name: 'Pay' }).click();
>     await expect(page.getByText('Order confirmed')).toBeVisible();
>   });
> });
> ```
>
> Steps appear as collapsible sections in the HTML report and in traces. When this fails, the report says *"failed in: Complete payment"* rather than pointing at a line number in a sixty-line test.
>
> That matters most for the person who *didn't* write the test — which after a few months is everyone, including you. It's documentation that the report renders.
>
> Steps are also useful inside page object methods, so a high-level action shows its internal phases in the trace.
>
> ---
>
> **ANNOTATIONS — handling known issues visibly**
>
> ```typescript
> test.skip('Not applicable on mobile', async ({ page }) => { });
> test.fixme('Broken by API change — JIRA-1234', async ({ page }) => { });
> test.slow();                    // triples the timeout for this test
> test.fail();                    // expects failure; fails if it passes
> ```
>
> The distinction between `skip` and `fixme` matters:
>
> - **`skip`** — intentionally not applicable here. Not a problem.
> - **`fixme`** — known broken, needs fixing. It's a debt marker.
>
> Using `fixme` rather than commenting a test out keeps the debt **visible in the report**. A commented-out test disappears entirely and nobody ever comes back to it. That's how suites quietly shrink.
>
> Conditional skipping:
> ```typescript
> test.skip(({ browserName }) => browserName === 'webkit', 'Safari-specific bug');
> ```
>
> `test.fail()` is the underused one — it asserts a test *should* fail, and errors if it starts passing. Useful for a known bug: when someone fixes it, the suite tells you, rather than the test quietly going green unnoticed.
>
> ---
>
> **SOFT ASSERTIONS — collect multiple failures**
>
> ```typescript
> await expect.soft(page.getByTestId('title')).toHaveText('Dashboard');
> await expect.soft(page.getByTestId('user')).toHaveText('Sagar');
> await expect.soft(page.getByTestId('count')).toHaveText('5');
> // test continues through all three; fails at the end reporting all failures
> ```
>
> Standard assertions stop at the first failure. Soft assertions record and continue, so you see every problem in one run rather than fixing one, re-running, finding the next.
>
> Good for verifying multiple independent things on a page. **Not** for sequential flows where a later step depends on an earlier one succeeding — there, failing fast is correct, and continuing produces a cascade of meaningless errors.
>
> ---
>
> **`test.describe` and `test.use`**
>
> ```typescript
> test.describe('Admin features', () => {
>   test.use({ storageState: '.auth/admin.json' });    // scoped config override
>
>   test('Can delete users', async ({ page }) => { });
>   test('Can view audit log', async ({ page }) => { });
> });
> ```
>
> `test.use` overrides config for a block — different auth state, viewport, locale. Useful for testing a specific role or a mobile viewport without a separate project.
>
> ```typescript
> test.describe.configure({ mode: 'serial' });   // run in order, stop on first failure
> ```
>
> Serial mode exists but I'd treat it as a smell. It creates order dependence, which breaks under sharding and makes tests non-independent. Occasionally justified for a genuinely sequential workflow; usually it means the tests should own their own setup."

---

## Feature Reference

| Feature | Purpose | Notes |
|---------|---------|-------|
| `tag: ['@smoke']` | Selective execution | Enables tiered CI |
| `test.step()` | Structure in reports | Names the failing phase |
| `test.skip()` | Not applicable | Not a debt marker |
| `test.fixme()` | Known broken | Keeps debt visible |
| `test.slow()` | Triples timeout | For genuinely slow ops |
| `test.fail()` | Expects failure | Alerts when a bug is fixed |
| `expect.soft()` | Collect failures | Not for dependent steps |
| `test.use()` | Scoped config | Auth state, viewport, locale |
| `describe.configure({mode:'serial'})` | Ordered | Treat as a smell |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Tags are the feature with the most leverage, because they enable tiered execution. Smoke on pull requests, full regression on merge, slower suites nightly. That gets most of the feedback-speed benefit of sharding without paying for parallel runners, so I'd reach for it first when a pipeline gets slow.
>
> `test.step` is the one I'd argue for on report quality. In a long end-to-end flow, a failure without steps points at a line number in a sixty-line test. With steps, the report says which phase failed — 'Complete payment' rather than 'line 47'. That's the difference between someone who didn't write the test being able to triage it or not, and after a few months that's everyone.
>
> On annotations, the distinction I care about is `fixme` versus commenting a test out. A commented test disappears — nobody comes back to it, and the suite quietly shrinks while the count in your head stays the same. `fixme` keeps the debt visible in every report. That ties directly to the silent test debt problem: green boards with less coverage than anyone realizes.
>
> And I'd flag serial mode as something I avoid. It creates order dependence, which breaks under sharding and makes tests non-independent. Usually it means the tests should be creating their own setup instead."

---

## Common Interview Questions & Answers

### Q1: "How do you organize which tests run when?"

**A:** "Tags plus `--grep`. Smoke on pull requests for fast feedback, full regression on merge, cross-browser and slow suites nightly. That's usually a better answer than adding runners when a pipeline gets slow, because tiering is free and sharding costs money."

### Q2: "What's `test.step` for?"

**A:** "Structuring long tests so the report shows which phase failed rather than a line number. In a checkout flow with five stages, a failure reporting 'Complete payment' is immediately actionable; 'line 47' means opening the file and reading. It matters most for whoever didn't write the test, which after a few months is everyone."

### Q3: "Difference between `skip` and `fixme`?"

**A:** "`skip` means intentionally not applicable — a mobile-only test on desktop, say. `fixme` means known broken and needs fixing. The important thing is using `fixme` rather than commenting a test out, because a commented test disappears from the report and nobody returns to it. That's how suites quietly lose coverage while the board stays green."

### Q4: "When would you use soft assertions?"

**A:** "Verifying several independent things on one page — you get all the failures in one run instead of fixing one and re-running. I wouldn't use them in a sequential flow where a later step depends on an earlier one, because then continuing after a failure produces a cascade of meaningless errors that obscure the real one."

### Q5: "Would you use serial mode?"

**A:** "Rarely, and I'd treat it as a smell. It creates order dependence, which breaks when you shard and means tests aren't independent. There are genuinely sequential workflows where it's justified, but usually it signals that tests should be creating their own preconditions rather than relying on a previous test's side effects."

---

## Weak Areas to Watch

❌ **Don't comment tests out**
→ Use `fixme`. Commented tests vanish and never come back.

❌ **Don't skip tagging**
→ It's what makes tiered CI possible.

❌ **Don't use soft assertions in dependent flows**
→ Produces cascading meaningless failures.

❌ **Don't reach for serial mode**
→ Order dependence breaks sharding.

❌ **Don't write long tests without steps**
→ Failures become line numbers instead of phases.

---

## Key Takeaway for Interview

These features are what make a large suite navigable rather than just large.

The one with the most strategic weight:

> **"Tags enable tiered execution — smoke on PRs, regression on merge, slow suites nightly. That's the free version of the speed win that sharding buys with money."**

And the one that shows you think about the next person:

> **"`test.fixme` instead of commenting a test out. A commented test disappears from the report and nobody comes back to it — that's how a suite quietly loses coverage while the board stays green."**

---

## Checklist

✅ Standard tag vocabulary agreed (`@smoke`, `@critical`, `@slow`)
✅ CI triggers tiered by tag
✅ Long tests structured with `test.step`
✅ `fixme` used instead of commenting out
✅ `test.fail()` on known bugs so fixes are noticed
✅ Soft assertions only for independent checks
✅ Serial mode avoided
✅ `test.use` for scoped config rather than duplicate projects

---

**Remember:** A report that names the failing phase saves someone twenty minutes, every time. 🎯
