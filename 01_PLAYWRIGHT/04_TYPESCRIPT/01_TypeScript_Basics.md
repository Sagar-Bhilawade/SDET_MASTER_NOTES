# TypeScript - Basics for Test Automation

## What This Covers

Enough TypeScript to explain your framework code confidently in an interview. Not deep language theory — the subset that shows up in automation and that interviewers actually probe.

---

## Why It Matters (Interview Context)

Interviewers use TypeScript questions as a **proxy for whether you write real code or just scripts.**

The distinction they're listening for: someone who copies test code versus someone who designs framework code. Types are where that shows, because typing is what you do when other people will use your code.

You don't need to be a TypeScript expert. You need to explain your own framework's types without hesitating.

---

## Your 30-Second Interview Answer

> "TypeScript gives us type safety at compile time, which in a test framework mostly means catching mistakes before a pipeline run rather than during one. The practical value is in fixtures and page objects — a typed fixture means your editor tells you what's available, and a typo becomes a compile error instead of a `undefined is not a function` at minute eight of a CI job."

---

## Your 2-Minute Deep Answer

> "The types that actually matter in automation:
>
> **Basic annotations**
> ```typescript
> const email: string = 'test@example.com';
> const retries: number = 3;
> const isAdmin: boolean = false;
> const tags: string[] = ['smoke', 'regression'];
> ```
>
> In practice you annotate less than you'd think — TypeScript infers most of it. Where you *do* annotate is function signatures and shared structures, because that's the contract other code depends on.
>
> **Interfaces — describing shape**
> ```typescript
> interface User {
>   id: number;
>   email: string;
>   role: 'admin' | 'user' | 'viewer';
>   name?: string;              // optional
>   readonly createdAt: string; // can't be reassigned
> }
> ```
>
> Three things worth calling out there:
>
> `role` uses a **union of literals** rather than `string`. That means `role: 'admni'` is a compile error rather than a mystery 403 at runtime. This is one of the highest-value typing patterns in test code, because test data typos are otherwise silent.
>
> `name?` marks it **optional** — present or absent, both valid.
>
> `readonly` prevents reassignment, useful for things a test shouldn't mutate.
>
> **Functions**
> ```typescript
> function buildUser(email: string, role: 'admin' | 'user' = 'user'): User {
>   return { id: Date.now(), email, role, createdAt: new Date().toISOString() };
> }
> ```
>
> Parameter types, a default value, and a return type. The return type is the one worth always writing on shared helpers — it's the contract, and it makes the function self-documenting.
>
> **`type` vs `interface`**
>
> Common interview question. Practically:
> - `interface` for object shapes, and it can be extended and merged
> - `type` for unions, intersections, and aliases
>
> ```typescript
> interface Page { url: string; }
> interface AdminPage extends Page { permissions: string[]; }
>
> type Status = 'pass' | 'fail' | 'skip';        // interface can't do this
> type Result = TestCase & { duration: number }; // intersection
> ```
>
> My rule: interface for objects, type for everything else. Teams argue about this more than it deserves — consistency matters more than which you pick.
>
> **`any` vs `unknown`**
>
> Worth knowing because it's a real depth question.
>
> `any` turns off type checking entirely. It's an escape hatch and it's contagious — one `any` propagates through everything it touches.
>
> `unknown` is the safe version: you can hold anything, but you must narrow it before use.
>
> ```typescript
> const body: unknown = await response.json();
>
> // Can't use it directly — must narrow first
> if (typeof body === 'object' && body !== null && 'email' in body) {
>   // now it's safe
> }
> ```
>
> In test code, API responses are the classic case. `response.json()` genuinely returns unknown data, so typing it properly — either narrowing or validating against a schema — is where type safety earns its keep.
>
> **Null safety**
> ```typescript
> const text = await page.getByTestId('total').textContent();  // string | null
>
> expect(text).not.toBeNull();
> expect(text!.trim()).toBe('$99');    // ! asserts non-null
> ```
>
> Playwright's `textContent()` returns `string | null` because the element might not have text. TypeScript forces you to handle that. The `!` operator tells the compiler you're certain — use it sparingly, and only after you've actually asserted.
>
> **Where this pays off in a framework:**
>
> ```typescript
> // Untyped — mistakes surface at runtime, in CI
> async function login(page, credentials) {
>   await page.fill('#email', credentials.emial);   // typo, silent
> }
>
> // Typed — this is a compile error
> async function login(page: Page, credentials: Credentials): Promise<void> {
>   await page.getByTestId('email').fill(credentials.emial);
>   //                                               ^^^^^ caught immediately
> }
> ```
>
> That's the whole argument in one example. In a suite that takes ten minutes to run, catching a typo at compile time instead of minute eight is worth real money over a year."

---

## Quick Reference

| Concept | Syntax | Use in automation |
|---------|--------|-------------------|
| Union of literals | `'admin' \| 'user'` | Prevents test data typos |
| Optional | `name?: string` | Fields not always present |
| Readonly | `readonly id: number` | Immutable test data |
| Interface extension | `extends BasePage` | Page object hierarchies |
| Type alias | `type Status = ...` | Unions, intersections |
| `unknown` | `const b: unknown` | API response bodies |
| Non-null assert | `text!.trim()` | After asserting not-null |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Coming from Java on the legacy stack, TypeScript felt familiar — static typing, interfaces, generics. The difference is that TypeScript's types are compile-time only; they're erased at runtime, so they're a development-time safety net rather than a runtime guarantee. That distinction matters when you're validating API responses, because a type annotation doesn't actually check the data.
>
> Where it earns its keep in our framework is fixtures and page objects. A typed fixture means the editor tells you what's available and a typo is a compile error rather than a failure eight minutes into a pipeline run.
>
> The pattern I use most is union types for test data — `role: 'admin' | 'user' | 'viewer'` instead of `role: string`. A typo in a role name would otherwise be a silent 403 that looks like a permissions bug. With a union, it doesn't compile. That's a small thing that saves a surprising amount of debugging."

---

## Common Interview Questions & Answers

### Q1: "Why TypeScript over JavaScript for test automation?"

**A:** "Catching mistakes at compile time rather than during a pipeline run. In a suite that takes ten minutes, a typo caught by the compiler instead of at minute eight is a meaningful difference over a year. It's also better for framework code specifically — typed fixtures and page objects mean the editor documents the API for whoever uses them next."

### Q2: "What's the difference between `type` and `interface`?"

**A:** "Interfaces are for object shapes and can be extended and merged. Types can express unions and intersections, which interfaces can't. My rule is interface for objects, type for everything else — but honestly, consistency within a codebase matters more than which convention you pick."

### Q3: "What's the difference between `any` and `unknown`?"

**A:** "`any` disables type checking and is contagious — it spreads through everything it touches. `unknown` accepts anything but forces you to narrow before using it, so you keep the safety. In test code the classic case is `response.json()`, which genuinely returns unknown data. Typing it as `any` gives up exactly where you most need the checking."

### Q4: "Does TypeScript guarantee your API responses match your types?"

**A:** "No, and that's an important gap. Types are erased at compile time — annotating a response as `User` tells the compiler what you expect, not what actually arrived. If the API changes, the type lies and you get a runtime failure. That's why for critical contracts I'd validate with a schema library rather than relying on the annotation."

### Q5: "How much TypeScript does an SDET actually need?"

**A:** "Enough to design framework code that other people use — interfaces, generics, unions, and understanding async typing. Not deep type-level programming. The bar is being able to write a fixture or page object that's self-documenting through its types, and explain your own framework's code without hesitating."

---

## Weak Areas to Watch

❌ **Don't use `string` where a union works**
→ `'admin' | 'user'` catches typos; `string` doesn't.

❌ **Don't reach for `any`**
→ It's contagious. `unknown` plus narrowing keeps the safety.

❌ **Don't claim types validate runtime data**
→ They're erased at compile time. Knowing this is a depth signal.

❌ **Don't over-annotate**
→ TypeScript infers well. Annotate signatures and shared contracts.

---

## Key Takeaway for Interview

TypeScript in automation is about **catching mistakes before the pipeline does**.

The highest-value small pattern:

> **"Union types for test data. `role: 'admin' | 'user'` instead of `role: string` — a typo becomes a compile error instead of a silent 403 that looks like a permissions bug."**

And the depth answer most candidates miss:

> **"Types are erased at compile time. Annotating an API response as `User` says what you expect, not what arrived — so for critical contracts you need schema validation, not just a type annotation."**

---

## Checklist

✅ Union literals used for constrained values
✅ Return types on shared helpers
✅ `unknown` over `any` for external data
✅ Null handled explicitly, `!` used only after asserting
✅ Interfaces for objects, types for unions
✅ Understand types are compile-time only

---

**Remember:** Types are documentation the compiler enforces. 🎯
