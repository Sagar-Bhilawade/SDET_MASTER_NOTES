# AI In Testing - Manual Flow To Playwright Automation Drafts

## What it is

Taking a reviewed manual flow — a sequence of steps a tester performs by hand — and
using AI to draft the Playwright automation for it: the page object methods, the test
body, the locators, and the assertions. The output is a *scaffold*, reviewed and
hardened by the engineer before it enters the suite.

The critical word is **reviewed**. The manual flow goes in already understood — the
tester knows what it proves and what the real assertion is. AI accelerates the
*translation to code*, not the decision about what to test. Converting an
un-thought-through flow just produces fast garbage.

Where it fits with tooling: Playwright's own `codegen` records a flow into a script,
and an AI assistant can go further — turning a described flow into page-object-shaped
code and suggesting assertions. Both produce drafts. Neither produces a mergeable test.

## Why it matters

This is the most concrete "show me you actually use AI" question, and it's directly on
your resume (Playwright CLI agent-style workflows, script generation + maintenance). An
interviewer wants to hear where the draft is good, where it's dangerous, and exactly
what you fix before it lands.

The danger is specific and worth naming: AI-generated Playwright looks clean and passes
locally, which is the most seductive failure mode. It'll use a brittle CSS locator, a
`waitForTimeout`, or an assertion that checks visibility instead of behavior — and it'll
be green. Green-but-wrong is harder to catch than red, so review is where the real
engineering is.

## 30-second answer

> "I take a manual flow I've already reviewed and use AI to draft the Playwright — page
> methods, test body, locators, assertions. Then I harden it: swap brittle locators for
> role or test-id, remove any hard waits in favor of web-first assertions, and fix
> assertions that check visibility instead of behavior. The draft saves the boilerplate;
> the review is where the test actually becomes trustworthy."

## 2-minute answer

> "The flow enters the process already understood — I know what it proves before any
> code exists. That's the precondition; converting a flow I haven't thought about just
> gives me fast garbage.
>
> Given a clear flow, AI drafts the automation quickly: it scaffolds the page object
> methods, writes the test body, and proposes locators and assertions. For boilerplate —
> navigation, form fill, structure — it's genuinely fast and I keep most of it.
>
> Then I harden it, and I know exactly what to look for because the failure modes are
> predictable. First, locators: AI reaches for brittle CSS or auto-generated selectors,
> so I move them up the hierarchy to role or `data-testid`. Second, waits: it likes
> `waitForTimeout`, which I replace with Playwright's web-first assertions and
> auto-waiting — hard sleeps are exactly the flakiness source I've spent effort removing.
> Third, and most important, assertions: AI often asserts that something is *visible*
> rather than that the behavior is *correct*, so I rewrite the assertion to prove the
> actual outcome. Then data setup and cleanup, and whether this even belongs at UI level
> or should be an API test.
>
> The trade-off is real: AI makes it cheap to produce tests that pass locally and look
> clean, and green-but-wrong is more dangerous than red. So the speed-up is in drafting,
> and the value I add is in the review — that's the part that makes it a real test, and
> it's the part I never skip."

## The review checklist for a generated draft

| Check | What AI tends to get wrong | The fix |
|---|---|---|
| Locators | Brittle CSS / auto-generated selectors | Role → test-id → text, up the hierarchy |
| Waiting | `waitForTimeout` / hard sleeps | Web-first assertions, auto-waiting |
| Assertions | Checks visibility, not behavior | Assert the real outcome |
| Test level | Everything as a UI test | Push to API/unit where it belongs |
| Data | No isolation, shared state | Seed per test, unique data |
| Cleanup | Missing teardown | Restore state / fixtures |

## Real project example

> "On the Playwright track I use CLI-style, agent-assisted workflows to speed up script
> generation and maintenance. A reviewed manual flow becomes an AI-drafted scaffold —
> page methods, test body, first-pass locators and assertions. The draft is fast, but I
> always harden it: on a recent one the generated code used a CSS class chain and a
> `waitForTimeout`, and asserted the success banner was visible rather than checking the
> record was actually created. I moved the locators to role/test-id, replaced the sleep
> with a web-first assertion, and changed the assertion to verify the created state. That
> AI-assisted drafting is part of the 30% efficiency gain — but the number comes with
> the review, not instead of it. I don't run self-healing selectors or an autonomous
> generator in production; it's drafting plus disciplined review."

## Common interview Q&A

### Q1: How do you convert a manual flow to automation with AI?
**A:** "The flow enters already reviewed — I know what it proves. AI drafts the
Playwright scaffold: page methods, test body, locators, assertions. Then I harden it
against the predictable failures — brittle locators, hard waits, and visibility-only
assertions — before it enters the suite."

### Q2: What's wrong with AI-generated Playwright by default?
**A:** "It passes locally and looks clean, which hides the problems. It picks brittle CSS
locators, uses `waitForTimeout`, and asserts visibility instead of behavior. Green-but-
wrong is harder to catch than red, so the review is where the engineering is."

### Q3: Isn't `codegen` enough?
**A:** "Codegen records a flow into a script, which is a starting point, but it produces
absolute selectors and no real assertions. AI drafting goes further into
page-object-shaped code, but both are drafts — neither is mergeable without hardening."

### Q4: How is this not overselling AI?
**A:** "Because I'm precise about the boundary. AI drafts and I review; I don't run
self-healing selectors or an autonomous generator in production. The efficiency gain
comes from faster drafting plus disciplined review, not from trusting the output."

## Weak areas to watch

- Never claim self-healing selectors or an autonomous Planner/Generator/Healer in
  production — that contradicts the resume and the truth.
- Don't skip the assertion review; visibility-only assertions are the classic trap.
- Don't keep a hard wait just because the generated test is green.
- Don't automate a flow you haven't reviewed — the draft inherits the confusion.
- Don't paste internal app locators or code into an interview answer.

## Key takeaway

> "AI drafts the Playwright fast, but the draft is green-but-wrong until I harden the
> locators, waits, and assertions. The speed is in the drafting; the trust is in the
> review — and I own the review."
