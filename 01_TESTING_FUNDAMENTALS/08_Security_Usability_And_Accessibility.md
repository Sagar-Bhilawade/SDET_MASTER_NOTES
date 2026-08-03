# Security, Usability, Compatibility & Accessibility

## What This Covers

The non-functional areas beyond performance — what each checks, which parts an SDET realistically owns, and which need specialists or real users.

---

## Why It Matters (Interview Context)

These get asked as a set — *"what other non-functional testing do you do?"* — and the honest answer is more valuable than an exhaustive one.

**Security is the one worth being precise about.** There's a meaningful slice you genuinely own as an SDET (authorization testing), and a slice that's a specialist discipline (penetration testing). Claiming the second collapses immediately; claiming the first well is a differentiator, because plenty of automation engineers never test authorization at all.

The accessibility angle has a nice connection to your locator strategy that's worth raising unprompted.

---

## Your 30-Second Interview Answer

> "Security testing at my level means authorization — same API call as a different role should return 403, changing a resource ID in a request shouldn't leak another user's data, tokens should actually expire. Those belong in the automated API suite. Penetration testing is a specialist discipline and I wouldn't claim it. Accessibility I integrate with axe, which catches the mechanical half automatically — and there's a useful overlap where role-based locators surface real accessibility defects as a side effect. Usability genuinely needs real users; it isn't automatable."

---

## Your 2-Minute Deep Answer

> "**SECURITY — where the SDET line actually sits:**
>
> The OWASP Top 10 is the reference point. The categories that matter most in practice:
>
> - **Broken access control** — can a user reach another user's data?
> - **Broken authentication** — do sessions and tokens behave correctly?
> - **Injection** — SQL, command, script
> - **Sensitive data exposure** — is anything leaking in responses or logs?
> - **Security misconfiguration** — default credentials, verbose errors, open endpoints
>
> **What I genuinely own — authorization testing in the API suite:**
>
> ```
> Same endpoint, different role         → 403, not 200
> Change resource ID in the request     → 403/404, not another user's data
> Expired token                          → 401
> No token                               → 401
> Valid token, insufficient permission   → 403 (not 401)
> ```
>
> That **IDOR** case — insecure direct object reference — is the one worth naming. Change `/api/orders/1234` to `/api/orders/1235` and see whether you get someone else's order. It's a straightforward automated check and it catches a genuinely serious class of bug that functional testing walks straight past, because the happy path never tries it.
>
> The **401 vs 403** distinction matters here too. Returning 401 for a permissions problem sends the client into a pointless re-authentication loop — and in a UI with automatic token refresh, that can be an infinite loop.
>
> **What I wouldn't claim:** penetration testing, threat modelling, deep vulnerability assessment. Different discipline. What I'd add is that dependency scanning — Snyk, Dependabot, `npm audit` — is cheap to wire into CI and catches known CVEs in libraries without any security expertise.
>
> ---
>
> **USABILITY — the honest limit:**
>
> Can real users accomplish their goal without confusion? This is genuinely **not automatable**, because automation checks what you told it to check, and *"is this flow confusing"* isn't expressible as an assertion.
>
> It needs real users doing real tasks while someone observes. What automation can contribute is the mechanical adjacent checks — are error messages present and specific, is there a loading state, does the back button work.
>
> Saying clearly that this needs humans is better than pretending otherwise.
>
> ---
>
> **COMPATIBILITY:**
>
> Browsers and versions, operating systems, screen sizes, devices.
>
> This is where cross-browser automation earns its place — but I'd **tier** it. Full cross-browser on every commit is expensive for the value it returns. Nightly or pre-release is usually the right cadence, with the primary browser on every run.
>
> **Responsive breakpoints are under-tested.** A layout that works at 1920px and collapses at 768px passes every desktop test you have. Playwright makes this cheap:
>
> ```typescript
> test.use({ viewport: { width: 375, height: 812 } });
> ```
>
> Worth having a small suite that runs the critical flows at mobile width, because that's where real users are.
>
> ---
>
> **ACCESSIBILITY:**
>
> WCAG compliance — screen reader support, keyboard navigation, colour contrast, focus order, semantic structure.
>
> **What automates well** (roughly the mechanical half): missing alt text, insufficient colour contrast, missing form labels, invalid ARIA, missing page landmarks. Tools like **axe** integrate directly:
>
> ```typescript
> const results = await new AxeBuilder({ page }).analyze();
> expect(results.violations).toEqual([]);
> ```
>
> **What doesn't automate:** whether the screen reader experience actually makes sense, whether focus order is logical, whether alt text is *meaningful* rather than merely present. `alt="image1"` passes an automated check and helps nobody.
>
> **The connection worth raising unprompted:**
>
> Role-based locators and accessibility overlap directly. If `getByRole('button', { name: 'Submit' })` can't find your element, that's often a **genuine accessibility defect** — the element isn't exposing a proper role or accessible name. A screen reader user would hit the same wall.
>
> So writing accessible-first locators surfaces real accessibility issues as a by-product, rather than as a separate effort. That's a nice example of a testing practice producing a product-quality outcome, and it's the kind of connection interviewers remember.
>
> ---
>
> **Where these fit in the strategy:**
>
> Automated in the suite: authorization, accessibility (mechanical), responsive breakpoints, dependency scanning.
>
> Specialist or manual: penetration testing, usability, screen reader experience.
>
> Being clear about which is which is the answer — an exhaustive claim to cover all of it wouldn't be credible."

---

## What Automates, What Doesn't

| Area | Automatable | Owned by SDET? |
|------|:---:|:---:|
| Authorization / IDOR | ✅ Fully | ✅ Yes |
| Dependency CVEs | ✅ Fully | ✅ Yes (CI) |
| Injection | ⚠️ Partly | ⚠️ Basic cases |
| Penetration testing | ❌ | ❌ Specialist |
| Accessibility — mechanical | ✅ axe | ✅ Yes |
| Accessibility — experience | ❌ | ❌ Needs users |
| Responsive breakpoints | ✅ | ✅ Yes |
| Cross-browser | ✅ | ✅ Yes (tiered) |
| Usability | ❌ | ❌ Needs users |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "The security slice I genuinely own is authorization, as part of API validation. Same endpoint called as a different role should return 403 rather than 200. Changing a resource ID in the request shouldn't return someone else's data — that's the IDOR case, and it's a simple automated check that catches a serious class of bug functional testing walks past entirely, because the happy path never tries it.
>
> I'd be clear that penetration testing isn't mine. What I'd add is that dependency scanning in CI is cheap and catches known CVEs in libraries without needing security expertise.
>
> On accessibility, the connection I find genuinely useful is that role-based locators overlap with it. If `getByRole` can't find an element by its accessible name, that's often a real accessibility defect — the element isn't exposing a role or label, and a screen reader user hits the same wall. So accessible-first locators surface those issues as a by-product rather than as a separate initiative. It's a case where a testing practice produces a product-quality outcome.
>
> And I'd say plainly that usability isn't automatable. Automation checks what you told it to check — 'is this confusing' isn't an assertion. That needs real users, and pretending otherwise would be wrong."

---

## Common Interview Questions & Answers

### Q1: "What security testing do you do?"

**A:** "Authorization, in the API suite — same call as a different role returns 403, changing a resource ID doesn't leak another user's data, tokens actually expire. The IDOR check is the valuable one: change the ID in the URL and see whether you get someone else's record. Penetration testing is a specialist discipline and I wouldn't claim it, though dependency scanning in CI is cheap and worth having."

### Q2: "What's IDOR?"

**A:** "Insecure direct object reference — changing an identifier in a request to access something you shouldn't. `/api/orders/1234` becomes `/api/orders/1235` and you get another customer's order. It's easy to automate and it catches a serious bug class, because the functional happy path never attempts it."

### Q3: "Can accessibility testing be automated?"

**A:** "The mechanical half — axe catches missing alt text, contrast failures, missing labels, invalid ARIA. What it can't catch is whether the experience makes sense: whether focus order is logical, or whether alt text is meaningful rather than just present. `alt='image1'` passes automated checks and helps nobody."

### Q4: "How do you handle cross-browser and responsive testing?"

**A:** "Tiered — primary browser on every run, full cross-browser nightly or pre-release, because running everything everywhere on every commit is expensive for the value. Responsive breakpoints are the under-tested part: a layout that works at 1920px and collapses at 768px passes every desktop test. A small mobile-viewport suite over the critical flows is worth having."

### Q5: "Why isn't usability testing automatable?"

**A:** "Because automation only checks what you thought to tell it, and 'is this confusing' isn't expressible as an assertion. It needs real users doing real tasks with someone watching. Automation can cover the adjacent mechanical things — are error messages present and specific, is there a loading state — but not the judgement."

---

## Weak Areas to Watch

❌ **Don't claim penetration testing**
→ Specialist discipline. The authorization slice is yours; say that.

❌ **Don't say accessibility is fully automatable**
→ Tools catch the mechanical half. Meaningful alt text isn't checkable.

❌ **Don't claim usability can be automated**
→ It can't, and saying so honestly is stronger.

❌ **Don't run full cross-browser on every commit**
→ Expensive for the value. Tier it.

❌ **Don't skip responsive breakpoints**
→ Desktop tests are blind to mobile layout collapse.

---

## Key Takeaway for Interview

Precision about what you own beats an exhaustive claim.

The security slice worth naming:

> **"Authorization testing in the API suite — same call as a different role returns 403, and the IDOR check: change the resource ID and see whether you get someone else's data. Functional testing walks past that entirely, because the happy path never tries it."**

And the connection worth raising unprompted:

> **"If `getByRole` can't find an element by its accessible name, that's often a genuine accessibility defect — a screen reader user hits the same wall. Accessible-first locators surface those issues as a by-product."**

---

## Checklist

✅ Authorization tested across roles in the API suite
✅ IDOR checks on resource-ID endpoints
✅ 401 vs 403 distinction verified
✅ Dependency scanning wired into CI
✅ axe integrated for mechanical accessibility
✅ Responsive breakpoint suite over critical flows
✅ Cross-browser tiered, not every commit
✅ Honest boundaries: no penetration testing, no automated usability

---

**Remember:** `alt="image1"` passes the automated check and helps nobody. 🎯
