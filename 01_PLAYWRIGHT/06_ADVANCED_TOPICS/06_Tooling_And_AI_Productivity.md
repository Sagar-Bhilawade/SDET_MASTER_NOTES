# Playwright - Tooling & AI-Assisted Authoring

## What This Covers

Playwright's built-in tooling — codegen, UI mode, trace viewer — and how to talk about AI-assisted test authoring without sounding like a buzzword generator.

That second part matters. AI in testing is a topic where candidates either oversell wildly or avoid it entirely, and both read badly.

---

## Why It Matters (Interview Context)

Two separate signals here:

**Tooling fluency** — do you know the tools exist and when each applies? UI mode in particular changed the authoring loop significantly and plenty of people still don't use it.

**AI positioning** — this is increasingly asked, and the answer that lands is *specific and bounded*. Someone claiming "AI writes our tests" has either not tried it or isn't reviewing the output. Someone who can name exactly where it helps and where it doesn't sounds like an engineer who's actually used it.

---

## Your 30-Second Interview Answer

> "Codegen is useful for discovering selectors and scaffolding a flow quickly, but I treat its output as a first draft — it produces brittle locators and no structure, so it gets refactored into page objects before it lands. UI mode is where I actually author now; the watch mode and time-travel debugging tighten the loop a lot. On AI assistance, I use it for the mechanical parts — boilerplate, converting a manual test case into a scaffold, generating test data variations. Where I don't rely on it is deciding what to assert, because that requires knowing what actually matters, and a wrong assertion that looks plausible is worse than no test."

---

## Your 2-Minute Deep Answer

> "Three built-in tools and then the AI question.
>
> ---
>
> **CODEGEN — `npx playwright codegen`**
>
> Opens a browser, records your interactions, generates test code.
>
> ```bash
> npx playwright codegen https://staging.example.com
> npx playwright codegen --load-storage=auth.json app.com   # already logged in
> ```
>
> **Where it's genuinely useful:** exploring an unfamiliar application and discovering what locators are available. It shows you whether an element has a usable role or test ID, which is faster than reading the DOM.
>
> **Where it isn't:** generated code is a first draft, not a test. It produces locators that are often brittle, no page objects, no meaningful assertions, and a linear script with no structure. Committing codegen output directly is how suites end up full of CSS selectors.
>
> So: use it to discover, then refactor into the framework. I'd never treat it as the authoring workflow.
>
> ---
>
> **UI MODE — `npx playwright test --ui`**
>
> This is the significant one and it's underused.
>
> ```bash
> npx playwright test --ui
> ```
>
> What it gives you: watch mode that re-runs on file save, time-travel through each action with DOM snapshots, a live locator picker, and the network and console panels inline.
>
> The practical effect is that the authoring loop goes from *edit → run → read failure → guess → repeat* to something much tighter, because you can see the DOM state at the exact step that failed and try locators against it interactively.
>
> For debugging a test you're actively writing, this has largely replaced `--debug` for me.
>
> ---
>
> **TRACE VIEWER — `npx playwright show-trace`**
>
> Covered in depth in the debugging notes. The distinction worth keeping clear:
>
> - **UI mode** — for tests you're writing now, locally
> - **Trace viewer** — for failures that already happened, usually in CI
>
> Same underlying technology, different moment in the workflow.
>
> ---
>
> **AI-ASSISTED AUTHORING — the honest version**
>
> Where it genuinely helps:
>
> **Boilerplate and mechanical conversion.** Turning a written manual test case into a scaffolded test structure. Generating a page object skeleton from a list of elements. This is real time saved on work that requires no judgement.
>
> **Test data variations.** Producing edge-case inputs — boundary values, unicode, unusual formats. Good at breadth, which is exactly what you want here.
>
> **Explaining unfamiliar code.** Useful when picking up someone else's framework.
>
> **Refactoring assistance.** Mechanical transformations across many files.
>
> Where I don't rely on it:
>
> **Deciding what to assert.** This is the one that matters. Knowing what's worth verifying requires understanding what actually breaks and what costs money when it does. Generated assertions tend toward the plausible-but-shallow — asserting an element is visible rather than asserting the behaviour is correct. And a weak assertion that looks reasonable is worse than no test, because it counts in coverage and catches nothing.
>
> **Locator strategy.** Generated locators skew toward whatever's in the DOM rather than what's stable. Same failure mode as codegen.
>
> **Anything I wouldn't review line by line.** If I'm not reading it closely enough to catch a subtly wrong assertion, I shouldn't be committing it.
>
> ---
>
> **How I'd summarize the position:**
>
> AI is good at the parts of test writing that are mechanical, and test writing has more mechanical parts than people admit — boilerplate, structure, data variation. That's real productivity.
>
> What it doesn't do is the part that makes a test valuable, which is deciding what's worth checking. That's a judgement call about risk, and it needs someone who knows the system and what failure costs.
>
> The framing I'd use: it accelerates the typing, not the thinking. And on a team, the risk isn't that it writes bad tests — it's that it writes *plausible* ones fast enough that review quality slips. Which is an argument for stricter review, not for avoiding the tools."

---

## Tool Reference

| Tool | Command | When |
|------|---------|------|
| **Codegen** | `playwright codegen <url>` | Exploring an app, discovering locators |
| **UI mode** | `playwright test --ui` | Authoring and local debugging |
| **Debug** | `playwright test --debug` | Step-through with inspector |
| **Trace viewer** | `playwright show-trace <zip>` | Post-hoc failures, usually CI |
| **Headed** | `playwright test --headed` | Watching a run live |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "On tooling, UI mode changed how I author. The watch mode plus being able to time-travel to the exact step that failed and try locators interactively against that DOM state makes the loop much tighter than edit-run-guess.
>
> Codegen I use for discovery rather than authoring — it's a fast way to find out whether an element has a usable role or test ID on an unfamiliar screen. But its output is a first draft: brittle locators, no page objects, no real assertions. It gets refactored before anything lands.
>
> On AI assistance, my position is that it's good at the mechanical parts and there are more of those than people admit. Converting a written manual test case into a scaffold, generating a page object skeleton, producing edge-case data variations — that's real time saved on work that needs no judgement.
>
> Where I wouldn't lean on it is deciding what to assert, because that's a risk judgement that needs someone who knows what actually breaks and what it costs. Generated assertions tend toward plausible-but-shallow — checking something is visible rather than checking the behaviour is right. And that's the exact failure mode I care most about, because a weak assertion counts in coverage and catches nothing.
>
> So the way I'd put it: it accelerates the typing, not the thinking. The risk on a team isn't bad tests, it's plausible ones arriving fast enough that review quality slips — which argues for tighter review rather than avoiding the tools."

---

## Common Interview Questions & Answers

### Q1: "Do you use codegen?"

**A:** "For discovery, not authoring. It's a fast way to find out what locators an unfamiliar screen actually offers — whether things have usable roles or test IDs. But the output is a first draft: brittle selectors, no structure, no meaningful assertions. It gets refactored into page objects before it lands. Committing codegen output directly is how a suite ends up full of CSS selectors."

### Q2: "How do you use AI in your testing work?"

**A:** "For the mechanical parts — scaffolding a test from a written manual case, generating page object skeletons, producing edge-case data variations. That's genuine time saved on work requiring no judgement. Where I don't rely on it is deciding what to assert, because that's a risk judgement, and generated assertions skew toward plausible-but-shallow. A weak assertion that looks reasonable is worse than no test."

### Q3: "Could AI replace manual test writing?"

**A:** "It replaces a meaningful share of the typing, not the thinking. The valuable part of a test is deciding what's worth verifying, which requires knowing what breaks and what failure costs — that's contextual judgement about risk. What I'd actually flag as the team risk isn't that AI writes bad tests; it's that it writes plausible ones quickly enough that review quality slips. That's an argument for stricter review, not for avoiding it."

### Q4: "What's UI mode and why use it?"

**A:** "Watch mode with time-travel debugging. It re-runs on save, lets you step through actions with DOM snapshots at each point, and has an interactive locator picker plus network and console panels. The practical benefit is a much tighter authoring loop — instead of edit, run, read a failure, guess, you can see the DOM at the failing step and try locators against it directly."

### Q5: "UI mode or trace viewer?"

**A:** "Different moments. UI mode for tests I'm writing right now, locally. Trace viewer for failures that already happened, usually in CI, where I download the artifact and replay it. Same underlying technology, different point in the workflow."

---

## Weak Areas to Watch

❌ **Don't oversell AI**
→ "AI writes our tests" reads as either untried or unreviewed.

❌ **Don't dismiss it either**
→ Refusing to engage reads as incurious. Name where it helps.

❌ **Don't present codegen output as production-ready**
→ It's a draft. Saying so is the signal you've used it.

❌ **Don't skip UI mode**
→ Plenty of people still don't know it exists.

---

## Key Takeaway for Interview

On tooling: know the difference between authoring tools and post-hoc debugging tools.

On AI, the framing that lands because it's specific and bounded:

> **"It accelerates the typing, not the thinking. It's genuinely good at boilerplate, scaffolding, and data variation. What it doesn't do is decide what's worth asserting, because that's a risk judgement about what actually breaks."**

And the risk observation that shows you've thought past the obvious:

> **"The team risk isn't that AI writes bad tests — it's that it writes plausible ones fast enough that review quality slips. That's an argument for tighter review, not for avoiding the tools."**

---

## Checklist

✅ Codegen used for discovery, output always refactored
✅ UI mode as the primary local authoring loop
✅ Trace viewer for CI failures
✅ AI used for boilerplate, scaffolding, data variation
✅ Assertion decisions kept human
✅ All generated code reviewed line by line
✅ Position stated as specific and bounded, not evangelical

---

**Remember:** The valuable part of a test is knowing what's worth checking. That part is still yours. 🎯
