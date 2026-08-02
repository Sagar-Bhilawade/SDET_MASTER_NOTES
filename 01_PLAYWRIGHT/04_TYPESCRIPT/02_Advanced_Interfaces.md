# TypeScript - Advanced Interfaces & Utility Types

## What This Covers

The type patterns that make framework code clean — utility types, extension, and composition. This is where TypeScript stops being annotation and starts being design.

---

## Why It Matters (Interview Context)

Basic types show you can write TypeScript. **Utility types show you can design with it.**

The specific signal: someone who writes `Partial<User>` instead of duplicating an interface with everything marked optional understands that types can be *derived* rather than repeated. That's the same instinct as not duplicating code — and it's what an interviewer is checking when they ask about your framework's type design.

---

## Your 30-Second Interview Answer

> "The utility types I use constantly are `Partial` for factory overrides, `Pick` and `Omit` for deriving related shapes, and `Record` for keyed maps like test data by role. The principle is that types should be derived from a single source, not duplicated — if `User` changes, everything built from it updates automatically. Duplicated interfaces drift, and drifted types are worse than no types."

---

## Your 2-Minute Deep Answer

> "Utility types let you build types from other types instead of rewriting them.
>
> **`Partial<T>` — all fields optional**
>
> The single most useful one in test code, because it's what makes factories work:
>
> ```typescript
> interface User {
>   id: number;
>   email: string;
>   role: 'admin' | 'user' | 'viewer';
>   name: string;
> }
>
> function buildUser(overrides: Partial<User> = {}): User {
>   return {
>     id: Date.now(),
>     email: `user-${crypto.randomUUID()}@test.com`,
>     role: 'user',
>     name: 'Test User',
>     ...overrides,
>   };
> }
>
> const admin = buildUser({ role: 'admin' });        // only what matters
> const named = buildUser({ name: 'Specific' });
> ```
>
> `Partial<User>` means the caller can pass any subset. Without it you'd need a separate interface with every field optional — duplicated, and it drifts the moment `User` changes.
>
> This is the pattern that makes tests readable: the test states only the field under test, and the factory absorbs everything else.
>
> **`Pick<T, K>` and `Omit<T, K>` — derive a subset**
>
> ```typescript
> type LoginCredentials = Pick<User, 'email' | 'password'>;
> type UserResponse = Omit<User, 'password'>;    // API never returns it
> type NewUser = Omit<User, 'id' | 'createdAt'>; // server generates these
> ```
>
> `Omit` is particularly good for modelling API contracts. A create request doesn't include the ID; a response doesn't include the password. Deriving those from `User` means one source of truth.
>
> **`Record<K, V>` — keyed maps**
>
> ```typescript
> type Role = 'admin' | 'user' | 'viewer';
>
> const testUsers: Record<Role, User> = {
>   admin:  buildUser({ role: 'admin' }),
>   user:   buildUser({ role: 'user' }),
>   viewer: buildUser({ role: 'viewer' }),
> };
> ```
>
> The useful property: if you add `'auditor'` to `Role`, this object becomes a compile error until you add the entry. The type system enforces completeness — you can't forget a case.
>
> **`Required<T>` and `Readonly<T>`**
>
> ```typescript
> type CompleteConfig = Required<Config>;   // all optionals become mandatory
> type FrozenData = Readonly<TestData>;     // nothing reassignable
> ```
>
> `Readonly` is useful for shared fixtures — it stops a test accidentally mutating data another test depends on. Compile-time protection against a real source of flakiness.
>
> ---
>
> **Interface extension — page object hierarchies**
>
> ```typescript
> interface BasePage {
>   readonly url: string;
>   navigate(): Promise<void>;
> }
>
> interface AuthenticatedPage extends BasePage {
>   logout(): Promise<void>;
> }
>
> interface AdminPage extends AuthenticatedPage {
>   deleteUser(id: number): Promise<void>;
> }
> ```
>
> Each level adds capability. Where this pays off is that a function accepting `AuthenticatedPage` works with any page that extends it — you get polymorphism without inheritance coupling.
>
> **Intersections — combining**
>
> ```typescript
> type TestContext = AuthContext & DatabaseContext & APIContext;
> ```
>
> Useful for composed fixtures where a test needs several capabilities at once.
>
> ---
>
> **Deriving from real values with `typeof`**
>
> This one's genuinely elegant for config:
>
> ```typescript
> const config = {
>   baseURL: 'http://localhost:3000',
>   timeout: 30000,
>   retries: 2,
> } as const;
>
> type Config = typeof config;
> ```
>
> `as const` makes the values literal types, and `typeof` derives the type from the object. One source of truth — change the object and the type follows. No separate interface to keep in sync.
>
> ---
>
> **The principle underneath all of it:** derive, don't duplicate. Every type written by hand that could have been derived is a thing that will drift. And a type that's drifted from reality is worse than no type, because it's actively lying to the next person who reads it."

---

## Utility Type Reference

| Utility | Does | Use in automation |
|---------|------|-------------------|
| `Partial<T>` | All optional | Factory overrides |
| `Required<T>` | All mandatory | Fully-resolved config |
| `Pick<T, K>` | Subset of fields | Login credentials from User |
| `Omit<T, K>` | Everything except | API request/response shapes |
| `Record<K, V>` | Keyed map | Test data per role |
| `Readonly<T>` | Immutable | Shared fixture data |
| `typeof x` | Type from value | Config objects |
| `as const` | Literal types | Constant maps |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "The pattern I lean on most is `Partial<T>` for factories. A factory takes `Partial<User>` and merges over defaults, so a test writes `buildUser({ role: 'admin' })` and states only the thing it's actually testing. Everything else is absorbed. That's what keeps tests readable — the reader immediately knows role is the variable.
>
> `Omit` is the other one I use constantly, for modelling API shapes. A create request omits the ID because the server generates it; a response omits the password. Deriving those from a single `User` interface means one source of truth rather than three interfaces that drift apart.
>
> The one I'd point to as genuinely clever is `Record` over a union key. If test data is typed as `Record<Role, User>` and someone adds a new role to the union, the data object stops compiling until it's handled. The type system enforces that you didn't forget a case — that's a class of bug caught for free.
>
> Coming from Java, generics and interfaces were familiar. What TypeScript adds is this ability to *derive* types from other types, which Java doesn't really have an equivalent for."

---

## Common Interview Questions & Answers

### Q1: "How do you type a factory function?"

**A:** "`Partial<T>` for the overrides parameter, returning the full type. So `buildUser(overrides: Partial<User> = {}): User` — the caller passes any subset, the factory merges over defaults. That's what lets a test specify only the field under test, which is the whole readability benefit."

### Q2: "What's the difference between `Pick` and `Omit`?"

**A:** "Opposite directions — `Pick` selects fields to keep, `Omit` selects fields to remove. I use `Pick` when I want a small subset, like credentials from a user, and `Omit` when I want most of it minus a few, like a response that excludes the password. Whichever names fewer fields is usually the clearer choice."

### Q3: "Why derive types instead of writing them out?"

**A:** "Drift. A hand-written duplicate of an interface doesn't update when the original changes, so it silently diverges. A type that's out of sync with reality is worse than no type, because it's actively misleading. Deriving means one source of truth — change `User` and everything built from it follows."

### Q4: "What does `as const` do?"

**A:** "Makes values literal types rather than widened ones. Without it, `{ role: 'admin' }` infers `string`; with it, the literal `'admin'`. Combined with `typeof`, it lets you derive a precise type from a config object, so you don't maintain a separate interface alongside it."

### Q5: "When would you use `Readonly`?"

**A:** "Shared fixture data that tests shouldn't mutate. If a worker-scoped fixture provides reference data and one test mutates it, every subsequent test in that worker sees the change — that's a real and very confusing source of flakiness. `Readonly` makes it a compile error instead."

---

## Weak Areas to Watch

❌ **Don't duplicate interfaces**
→ Derive with `Partial`, `Pick`, `Omit`. Duplicates drift.

❌ **Don't type factory params as the full type**
→ `Partial<T>` is what makes overrides work.

❌ **Don't use plain objects for role-keyed data**
→ `Record<Union, T>` enforces completeness.

❌ **Don't forget `as const` on config**
→ Without it, literal types widen to `string`.

---

## Key Takeaway for Interview

Utility types are how TypeScript stops being annotation and becomes design.

The principle:

> **"Derive, don't duplicate. Every hand-written type that could have been derived will eventually drift — and a drifted type is worse than no type, because it lies to the next reader."**

The pattern that shows framework thinking:

> **"`Record<Role, User>` means adding a new role breaks compilation until the data is handled. The type system enforces you didn't forget a case."**

---

## Checklist

✅ Factories take `Partial<T>` overrides
✅ API shapes derived with `Pick` / `Omit`
✅ Role-keyed data typed as `Record<Union, T>`
✅ Shared fixture data marked `Readonly`
✅ Config typed via `as const` + `typeof`
✅ Page object hierarchy built with `extends`
✅ No hand-duplicated interfaces

---

**Remember:** A type you had to write twice is a type that will be wrong once. 🎯
