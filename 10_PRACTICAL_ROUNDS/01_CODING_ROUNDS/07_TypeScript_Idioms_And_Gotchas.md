# TypeScript / JS Idioms & Gotchas — Coding Round Reference

## What it is

The JavaScript/TypeScript-specific things that come up in coding rounds and that a modern-stack interviewer uses as depth probes: `==` vs `===`, `??` vs `||`, array methods, async/await correctness, and the type-system features. This is your Playwright-stack language — fluency here is expected when you're leading with Playwright.

---

## Why it matters

When you position Playwright-first, the coding round may well be in TypeScript, and the interviewer will read your idioms as a proxy for how much real TS you've written. The async gotchas in particular are where automation engineers get caught — because the *whole* of Playwright is async, and the forgotten `await` is the #1 bug in the entire ecosystem.

---

## The gotchas that get people

### `==` vs `===` (always `===`)
```javascript
0 == ""      // true  (coerces!)
0 == "0"     // true
null == undefined  // true
0 === ""     // false (no coercion)
```
`==` does type coercion with baffling rules. **Always use `===`.** The only defensible `==` is `x == null` (catches both null and undefined) — and even that, be explicit.

### `??` vs `||` (the one that bites automation)
```javascript
const count = input || 10;   // if input is 0, count becomes 10 — BUG
const count = input ?? 10;   // if input is 0, count stays 0 — correct
```
`||` falls back on any *falsy* value (`0`, `""`, `false`, `NaN`). `??` falls back only on `null`/`undefined`. When 0 or empty string are valid values (they usually are in test data), `??` is what you want. This is a real, subtle bug source.

### The forgotten `await` — THE automation bug
```typescript
async function login(page) {
  page.click('#submit');          // MISSING await — floating promise, test races ahead
  await expect(page).toHaveURL(/dashboard/);  // may run before click completes
}
```
Every Playwright action returns a promise. Miss an `await` and the test continues before the action finishes → flaky, order-dependent failures. **The lint rule `no-floating-promises` catches it — always enabled in a real config.** In a coding round, narrate every await. See [Async/Await](../../02_PLAYWRIGHT/04_TYPESCRIPT/04_Async_Await.md).

### `for...of` vs `for...in`
```javascript
for (const x of [10,20,30]) // x = 10,20,30 (values) — what you want for arrays
for (const i in [10,20,30]) // i = "0","1","2" (indices as strings!) — gotcha
```
`for...in` is for object keys and gives string indices on arrays. Use `for...of` for array values.

### Reference vs value
Objects and arrays are passed by reference. `const b = a; b.push(1)` mutates `a`. Spread to copy: `const b = [...a]` (shallow). Nested needs a deep clone.

---

## The array methods worth knowing cold

```typescript
arr.map(x => x * 2)                      // transform
arr.filter(x => x > 5)                   // keep matching
arr.reduce((acc, x) => acc + x, 0)       // fold to single value
arr.find(x => x.id === 3)                // first match (or undefined)
arr.some(x => x > 5)  / arr.every(...)   // boolean checks
arr.includes(5)                          // membership
arr.sort((a, b) => a - b)                // numeric sort (default is STRING sort!)
[...new Set(arr)]                        // dedup
Object.entries(obj) / Object.keys / Object.values
```

### The sort gotcha
```javascript
[10, 2, 1].sort()          // [1, 10, 2] — sorts as STRINGS by default!
[10, 2, 1].sort((a,b)=>a-b) // [1, 2, 10] — numeric
```
Default `.sort()` converts to strings and compares lexically. Always pass a comparator for numbers. A classic catch.

---

## The counting idioms (from the problem bank)

```typescript
// count with a Map
const freq = new Map<string, number>();
for (const c of s) freq.set(c, (freq.get(c) ?? 0) + 1);

// get-or-create a list (object)
(groups[key] ??= []).push(value);

// most frequent
[...freq.entries()].reduce((a, b) => (b[1] > a[1] ? b : a))[0];
```
`??=` (nullish assignment) is the clean "initialize if missing" — the TS equivalent of Java's `computeIfAbsent`.

---

## TypeScript type-system things they probe

- **`interface` / `type`** for shaping test data: `interface User { id: number; name: string; }`.
- **Utility types:** `Partial<T>` (all optional — for update payloads), `Pick`/`Omit` (derive a subset), `Record<K,V>` (typed map). **Derive, don't duplicate** — see [Advanced Interfaces](../../02_PLAYWRIGHT/04_TYPESCRIPT/02_Advanced_Interfaces.md).
- **Generics:** `function first<T>(arr: T[]): T` — reusable and type-safe.
- **`unknown` vs `any`:** `any` disables checking (avoid); `unknown` forces you to narrow before use (safe).
- **`as T` is a cast, NOT validation** — it tells the compiler to trust you; it checks nothing at runtime. For real API responses, validate with a schema/type-guard, don't just cast. This is a genuine senior distinction.

---

## Async idioms

```typescript
// sequential
const a = await stepOne();
const b = await stepTwo(a);

// parallel (independent) — faster
const [x, y] = await Promise.all([fetchX(), fetchY()]);

// in a loop — DON'T forget await; DON'T use forEach with async (it won't wait)
for (const item of items) await process(item);   // works
items.forEach(async i => await process(i));       // BUG — forEach ignores the promises
```
*The `forEach` + async trap is real: `forEach` doesn't await, so the loop finishes before the work does. Use `for...of` or `Promise.all(items.map(...))`.*

---

## Common interview Q&A

**Q: "Why `===` over `==`?"**
`==` coerces types with confusing rules (`0 == ""` is true). `===` compares without coercion — predictable.

**Q: "`||` gave the wrong default — why?"**
`||` falls back on any falsy value including 0 and "". Use `??` when those are valid.

**Q: "What's wrong with this async loop?"**
Missing `await`, or `forEach` with async (doesn't wait). Use `for...of` with `await`, or `Promise.all`.

**Q: "Does `as User` validate the response?"**
No — it's a compile-time assertion, zero runtime check. Validate with a schema if the data is external.

---

## Weak areas to watch

- **`===` always; `??` when 0/"" are valid.** The two most impactful JS gotchas.
- **Narrate every `await`** — the forgotten await is the ecosystem's #1 bug and interviewers know it.
- **`.sort()` needs a comparator for numbers** — default is string sort.
- **`for...of` for array values**, not `for...in`.
- **`as T` isn't validation** — know the difference; it's a senior signal.
- **`forEach` + async doesn't wait** — use `for...of` or `Promise.all`.

---

## Key takeaway

> **In a TS coding round the async correctness is the whole game — narrate every `await`, because the forgotten one is the most famous bug in the Playwright world.** Pair that with `===` over `==`, `??` over `||` when 0/"" are valid, a numeric comparator on `.sort()`, and knowing that `as T` casts but never validates. These mark you as someone who's written real TypeScript, not someone who learned it for the interview.
