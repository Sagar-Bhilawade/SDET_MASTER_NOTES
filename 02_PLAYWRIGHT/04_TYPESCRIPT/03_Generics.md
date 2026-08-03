# TypeScript - Generics: Reusable, Type-Safe Framework Code

## What Are Generics?

A way to write code that works with any type while **keeping** the type information.

Without generics, reusable code loses types:
```typescript
function first(items: any[]): any { return items[0]; }
const user = first(users);    // type: any — TypeScript now knows nothing
```

With generics, it doesn't:
```typescript
function first<T>(items: T[]): T { return items[0]; }
const user = first(users);    // type: User — preserved
```

`T` is a placeholder filled in at the call site. One function, full type safety for every type it's used with.

---

## Why It Matters (Interview Context)

**This is the TypeScript topic that separates senior from mid-level in an SDET interview.**

Most automation engineers use TypeScript's basics. Fewer write generic helpers. Being able to explain a generic API client or a typed fixture factory demonstrates you write *framework* code, not just test code — which is exactly the distinction a 20 LPA role is buying.

It's also the most common place candidates reach for `any` and lose the argument for using TypeScript at all.

---

## Your 30-Second Interview Answer

> "Generics let me write one helper that works with any type without losing type information. The clearest example in a test framework is a typed API client — one `get<T>` method where the caller specifies the response type, so `get<User>('/api/users/1')` returns a properly typed User rather than `any`. Without generics you'd either write a method per type or fall back to `any`, and `any` defeats the reason for using TypeScript."

---

## Your 2-Minute Deep Answer

> "The problem generics solve is reuse without losing types.
>
> **The typed API client — the clearest case in a test framework:**
>
> ```typescript
> class APIClient {
>   constructor(private request: APIRequestContext) {}
>
>   async get<T>(url: string): Promise<T> {
>     const response = await this.request.get(url);
>     return response.json() as T;
>   }
>
>   async post<TBody, TResponse>(url: string, data: TBody): Promise<TResponse> {
>     const response = await this.request.post(url, { data });
>     return response.json() as TResponse;
>   }
> }
> ```
>
> Used:
> ```typescript
> const user = await api.get<User>('/api/users/1');
> user.email;          // ✅ typed, autocompleted
> user.emial;          // ✅ compile error
>
> const created = await api.post<NewUser, User>('/api/users', newUserData);
> ```
>
> One client, every endpoint typed. The alternative is a method per resource, or `any` everywhere.
>
> **Worth being honest about a limitation here:** `as T` is an *assertion*, not a validation. If the API returns something different, TypeScript won't catch it — you get a runtime failure later, in a confusing place. For critical contracts I'd pair this with schema validation. Knowing that boundary is the difference between using generics and understanding them.
>
> ---
>
> **Constraints — limiting what T can be**
>
> ```typescript
> function getById<T extends { id: number }>(items: T[], id: number): T | undefined {
>   return items.find(item => item.id === id);
> }
> ```
>
> `T extends { id: number }` means "any type, as long as it has a numeric id". Now `item.id` is safe inside the function, and calling it with something lacking an id is a compile error.
>
> Constraints are what make generics genuinely useful rather than just permissive — unconstrained `T` means you can't do anything with it.
>
> ---
>
> **Generic factories**
>
> ```typescript
> function createFactory<T>(defaults: T) {
>   return (overrides: Partial<T> = {}): T => ({ ...defaults, ...overrides });
> }
>
> const buildUser = createFactory<User>({
>   id: 0, email: 'test@example.com', role: 'user', name: 'Test',
> });
>
> const admin = buildUser({ role: 'admin' });   // fully typed
> ```
>
> One factory generator, every entity gets a typed builder. This combines generics with `Partial<T>` from the utility types — that combination is the backbone of clean test data code.
>
> ---
>
> **Typed page object helpers**
>
> ```typescript
> function createPage<T extends BasePage>(
>   PageClass: new (page: Page) => T,
>   page: Page
> ): T {
>   return new PageClass(page);
> }
>
> const login = createPage(LoginPage, page);   // typed as LoginPage
> ```
>
> `new (page: Page) => T` is a **constructor signature** — "a class that takes a Page and produces a T". Useful in fixtures that instantiate page objects generically.
>
> ---
>
> **Multiple type parameters**
>
> ```typescript
> async function retry<T>(fn: () => Promise<T>, attempts = 3): Promise<T> {
>   for (let i = 0; i < attempts; i++) {
>     try { return await fn(); }
>     catch (e) { if (i === attempts - 1) throw e; }
>   }
>   throw new Error('unreachable');
> }
>
> const user = await retry(() => api.get<User>('/api/users/1'));  // typed User
> ```
>
> The return type flows through automatically — you don't annotate it at the call site. That's generics doing their job invisibly.
>
> ---
>
> **When I don't reach for generics:**
>
> They cost readability. A signature like `function f<T, K extends keyof T, V extends T[K]>(...)` is technically impressive and hard to read at 9am when a pipeline is red.
>
> My rule is the same as for any abstraction: introduce it on the third repetition, not the first. If two functions differ only by type, that's often fine. Three is a pattern worth generalizing. Premature generics are as costly as premature abstraction anywhere else."

---

## Generic Patterns Reference

| Pattern | Syntax | Use |
|---------|--------|-----|
| Basic | `<T>(x: T): T` | Preserve caller's type |
| Constrained | `<T extends { id: number }>` | Require certain fields |
| Multiple params | `<TBody, TResponse>` | Request and response types |
| Default | `<T = string>` | Sensible fallback |
| Constructor | `new (p: Page) => T` | Instantiate classes generically |
| With `Partial` | `(o: Partial<T>) => T` | Typed factories |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "The place generics earn their keep in our framework is the API client. One `get<T>` and `post<TBody, TResponse>` rather than a method per endpoint. A test writes `api.get<User>('/api/users/1')` and gets a properly typed User back with autocomplete — a typo in a field name is a compile error rather than an undefined at runtime.
>
> Second use is factories. A generic `createFactory<T>` combined with `Partial<T>` overrides means every entity gets a typed builder from one implementation. That's the pattern behind test data being both readable and type-safe.
>
> What I'd be honest about is the limitation: `response.json() as T` is an assertion, not validation. TypeScript trusts me. If the API returns a different shape, the type is a lie and the failure surfaces somewhere confusing later. So for the contracts that matter, generics get paired with schema validation rather than relied on alone.
>
> Coming from Java generics, the syntax was familiar. The difference is that TypeScript's are erased at compile time — there's no runtime type information, which is exactly why that `as T` gap exists."

---

## Common Interview Questions & Answers

### Q1: "What are generics and why use them?"

**A:** "Writing code that works with any type without losing type information. The clearest case in a test framework is a typed API client — one `get<T>` method where the caller specifies the response type. Without generics you'd write a method per resource or fall back to `any`, and `any` gives up the reason you're using TypeScript in the first place."

### Q2: "What does `T extends { id: number }` mean?"

**A:** "A constraint — T can be any type as long as it has a numeric `id`. That makes `item.id` safe to use inside the function, and calling it with something that lacks an id becomes a compile error. Constraints are what make generics useful; an unconstrained T is so permissive you can't actually do anything with it."

### Q3: "Does `response.json() as T` guarantee the response matches T?"

**A:** "No, and that's the important limitation. `as` is an assertion — I'm telling the compiler to trust me. If the API returns something else, TypeScript won't catch it and you get a runtime failure in a confusing place. TypeScript's types are erased at compile time, so there's no runtime check available. For contracts that matter I'd validate with a schema library alongside the generic."

### Q4: "When would you not use generics?"

**A:** "When they cost more readability than they buy. A signature with three type parameters and nested constraints is hard to read when you're debugging at speed. Same rule as any abstraction — introduce it on the third repetition, not the first. Two similar functions is often fine."

### Q5: "How do TypeScript generics compare to Java's?"

**A:** "Syntactically similar, and coming from Java they felt familiar. The key difference is erasure — both erase, but Java retains some runtime type information through reflection in ways TypeScript doesn't at all. TypeScript types simply don't exist at runtime, which is why type assertions on API responses are unverifiable and why you need schema validation for real guarantees."

---

## Weak Areas to Watch

❌ **Don't use `any` where a generic works**
→ It's the most common failure and it defeats the point of TypeScript.

❌ **Don't leave T unconstrained when you need properties**
→ You won't be able to use the value.

❌ **Don't claim `as T` validates anything**
→ It's an assertion. Knowing this is the depth signal.

❌ **Don't over-generalize**
→ Three type parameters is usually a readability problem.

---

## Key Takeaway for Interview

Generics are the clearest demonstration that you write **framework** code rather than test code.

The example that lands:

> **"One `get<T>` on the API client instead of a method per endpoint — `api.get<User>(...)` returns a typed User, and a field typo is a compile error."**

And the limitation that shows you actually understand them:

> **"`as T` is an assertion, not a validation. Types are erased at compile time, so if the API returns a different shape TypeScript won't catch it. For contracts that matter, generics need schema validation alongside them."**

---

## Checklist

✅ API client is generic, not one method per resource
✅ Factories generic + `Partial<T>` overrides
✅ Constraints used where properties are accessed
✅ No `any` where a generic would work
✅ Schema validation paired with `as T` on critical contracts
✅ Generics introduced on the third repetition, not the first

---

**Remember:** Generics without constraints are just `any` with extra syntax. 🎯
