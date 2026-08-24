# AI In Testing Cheat Sheet — AI-Augmented QA Round

> Dense recall for `11_AI_IN_TESTING`. Learn from the full notes; fire from this.
> One line runs through everything: **AI drafts fast; QA judgment owns risk, assertions,
> coverage, and release confidence.** Every answer names that boundary.

---

## The one-sentence frame (open with this)

> *"I use AI to accelerate analysis and drafting, but the testing decisions stay mine:
> what risk matters, what assertion proves the behavior, what to automate, and what
> needs exploratory/manual validation."*

Resume title backs it: **AI-Augmented QA Automation Engineer.** Metric: **30% efficiency
gain** on BASES_PLAYWRIGHT — always quoted *with* the review, never instead of it.

---

## Two-layer / two-agent model

- **QA Knowledge Agent** — requirement → coverage. Story/PR analysis, test plans,
  functional + regression cases, bug drafts, coverage tracker.
- **QA Automation Agent** — reviewed flow → automation. Playwright scaffolds, page-object
  drafts, code review, locator/flaky fixes. Uses **Playwright CLI-style exploration**
  (lighter/cheaper than a browser MCP session).

---

## PR & Jira analysis (note 02)

- **Story = intended behavior. PR = actual change. Test the GAP between them.**
- PR read → **blast radius**: shared util / API contract changed = wider than the story says.
- Most useful output = **the open questions** (empty states, permissions, error handling) → back to BA/dev *before* writing tests.
- Boundary: AI describes what code *looks like* it does; **a diff summary can be confidently wrong**. Validate real behavior yourself.

---

## Case generation (note 03)

- **AI = breadth. You = depth + priority.** It lists 20; you keep 5.
- **Good QA with AI is mostly deletion** — merge duplicates, kill shallow cases, parametrize.
- Grade the draft against **real technique** (EP / BVA / decision tables) — AI applies them mechanically and gets boundaries wrong.
- AI **cannot do priority** (no business risk) → you mark smoke / regression / exploratory.
- Trap: more cases ≠ better. Bloated suite is slower and **buries the important checks**.

---

## Regression selection & coverage (note 04)

- Selection by **risk**, not "run everything" or a fixed set: smoke core + AI-flagged impacted flows; **widen the net yourself for shared-code changes**.
- AI impact call can miss an **indirect dependency** → a missed regression is the costliest miss.
- Coverage = living map: automated / manual / to-automate. AI keeps it current, gives a prioritized backlog.
- **Honesty line: this is risk-and-gap reasoning, NOT real code coverage.** Real coverage needs instrumentation. (Use resume's 90% only as stated.)

---

## Manual flow → Playwright draft (note 05)

- Flow enters **already reviewed** — know what it proves. Converting an un-thought flow = fast garbage.
- **Green-but-wrong is the trap** — AI Playwright passes locally and looks clean. Harder to catch than red.
- Harden every draft — the predictable failures:

| Check | AI default | Fix |
|---|---|---|
| Locators | brittle CSS / auto-gen | role → test-id |
| Waits | `waitForTimeout` | web-first assertion |
| Assertion | asserts **visibility** | assert real **outcome** |
| Level | everything UI | push to API/unit |
| Data | shared state | seed + isolate + clean up |

- `codegen` records a flow (absolute selectors, no real assertions) — a start, not mergeable.
- **Never claim** self-healing selectors or an autonomous Planner/Generator/Healer in prod.

---

## Bug drafting (note 06)

- **AI formats; you own the facts.** It sharpens title, orders repro, phrases expected-vs-actual, flags missing detail.
- **Never lets AI invent/complete a repro step** — clean-but-false is worse than messy-but-true (dev can't reproduce → bounce-back → lost credibility).
- **Severity is yours** — it's business impact; AI may suggest, you decide.
- Value = raises the **floor** on report quality → fewer "can't reproduce" round-trips.

---

## Review gates (note 07) — the note that proves the section is honest

AI moves effort **from writing → reviewing.** Every generated test passes the gate or it doesn't merge:

1. **Why does this exist?** — can't state the risk in a sentence → **delete it.**
2. Assertion proves **behavior**, not visibility / bare 200.
3. Right **level** (UI / API / unit — AI over-uses UI).
4. **Locators** stable (role / test-id).
5. **Timing** web-first, no sleeps.
6. **Data** isolated + cleaned up.

- Real risk = **a large green suite that proves nothing** (AI bad tests look *good*).
- Gate slows merging **on purpose**; the speed-up stays in drafting. Net faster, suite you can defend.

---

## What NOT to say (red lines)

- ❌ "AI writes and owns my tests." → ✅ "AI drafts; I review and own the QA decisions."
- ❌ "We run production self-healing selectors." (contradicts resume + truth)
- ❌ "Agents understand requirements correctly." / "AI creates correct assertions automatically."
- ❌ Calling AI impact analysis "code coverage."
- ❌ Pasting real Jira / PR / internal code / logs into any answer — describe the **workflow**.

---

## Rapid-fire answers

- **"How do you use AI in testing?"** → Two layers: analysis (story/PR → plans, cases, regression, candidates) + automation (reviewed flow → Playwright drafts). I review assertions, locators, data, level. AI drafts; I own the decisions.
- **"Biggest risk of AI tests?"** → Plausible-but-shallow. Green-but-wrong. That's why review matters *more*, not less.
- **"Isn't that overselling AI?"** → No — clear boundary. No self-healing, no autonomous generator. Speed from drafting + a review gate.
- **"Most important review question?"** → "Why does this test exist?" Can't answer → delete.

---

## Key takeaway

> **AI shifts the effort from writing to reviewing — so review is where quality now lives.**
> The strongest AI answer is the bounded one: AI gives breadth and speed, I supply depth,
> priority, and the gate. The tell that you're senior about AI is naming its limit as
> confidently as its value — and knowing the real risk is a fast green suite that proves
> nothing.
