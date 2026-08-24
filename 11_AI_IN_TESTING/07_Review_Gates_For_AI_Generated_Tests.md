# AI In Testing - Review Gates For AI-Generated Tests

## What it is

The explicit checkpoints an AI-generated test must pass before it enters the suite. If
the earlier notes are about *using* AI, this one is about the *control* — the discipline
that keeps AI throughput from becoming AI debt.

A review gate is a small, fixed set of questions applied to every generated test:

1. **Why does this test exist?** — can I state the risk it covers? If not, delete it.
2. **Does the assertion prove behavior?** — not just visibility or a status code.
3. **Is it at the right level?** — UI, API, or unit; most generated tests over-use UI.
4. **Are locators stable?** — role/test-id, not brittle CSS.
5. **Is timing sound?** — web-first waiting, no hard sleeps.
6. **Is data isolated?** — seeded per test, cleaned up, no shared state.

If a test can't pass all six, it doesn't merge. The gate is deliberately boring and
mechanical — that's what makes it hold under time pressure.

## Why it matters

This is the note that proves the whole section is honest. Anyone can say "I use AI"; the
seniority signal is having a *guard* against it. The core risk of AI in testing isn't bad
tests you'd obviously reject — it's plausible tests generated so fast that review erodes,
and six months later you have a large green suite that proves very little.

Interviewers who go deep on AI are really probing for this. If you can describe the gate
— and especially the "if I can't explain why it exists, I delete it" rule — you've shown
you understand that AI shifts effort *from writing to reviewing*, and that review is
where quality now lives.

## 30-second answer

> "Every AI-generated test passes a fixed gate before it merges: can I say why it exists,
> does the assertion prove behavior, is it at the right level, are the locators stable, is
> the timing sound, is the data isolated. If it fails any of those, it doesn't go in. AI
> shifts my effort from writing tests to reviewing them, so the gate is where quality now
> lives — the biggest risk is a fast green suite that proves nothing."

## 2-minute answer

> "AI changes where the effort goes. Writing tests gets cheap, so the bottleneck moves to
> review — and if you don't formalize that review, throughput quietly turns into debt.
>
> So I run every generated test through a fixed gate. The first question is the most
> important: why does this test exist? If I can't state the risk it covers in a sentence,
> I delete it, because a test I can't justify is noise that slows the suite and dilutes
> signal. Then the mechanical checks: the assertion has to prove behavior, not just that
> something rendered; the test has to sit at the right level, because AI over-produces UI
> tests for things that should be API or unit; locators have to be stable; timing has to
> use web-first waiting, not sleeps; and data has to be isolated and cleaned up.
>
> The reason this matters more with AI than without is the failure mode. Hand-written bad
> tests tend to look bad. AI-generated bad tests look *good* — clean code, passing
> locally — so they slip through a casual review. A large green suite that proves nothing
> is the expensive outcome, because you trust it and it isn't earning that trust.
>
> The trade-off I accept openly: the gate slows down merging AI output. That's the point.
> The speed-up is in drafting; I deliberately don't let it leak into review. Net, I still
> come out faster, but with a suite I can defend."

## The gate as a checklist

| Gate | Pass condition | Common AI failure it catches |
|---|---|---|
| Purpose | I can state the risk in one sentence | Plausible test with no real reason to exist |
| Assertion | Proves behavior / real outcome | Asserts visibility or bare 200 |
| Level | UI/API/unit chosen deliberately | Everything forced through the UI |
| Locators | Role / test-id, stable | Brittle CSS or auto-generated selectors |
| Timing | Web-first waiting | `waitForTimeout` / hard sleeps |
| Data | Seeded + isolated + cleaned up | Shared state, order dependence |

## Real project example

> "As AI drafting became part of how I produce Playwright tests, I made the review
> explicit instead of ad-hoc, because the tests come in fast and passing. Before anything
> merges I run the gate — purpose, assertion, level, locators, timing, data. The purpose
> check does the most work: it's easy to accept a clean-looking test, and asking 'what
> risk does this cover?' is what catches the ones that shouldn't exist. This is also why I
> can claim the 30% efficiency gain honestly — it's speed *with* a gate, not speed by
> skipping review. Without the gate, the AI-assisted suite would have grown faster and
> meant less; the gate is what keeps green trustworthy."

## Common interview Q&A

### Q1: How do you stop AI from lowering test quality?
**A:** "A fixed review gate every generated test passes before merge: purpose,
assertion, level, locators, timing, data isolation. Fail any, it doesn't go in. AI moves
the effort from writing to reviewing, so the gate is where quality lives now."

### Q2: What's the single most important gate?
**A:** "Purpose. If I can't state in one sentence why the test exists, I delete it. A
test I can't justify is noise that slows the suite and dilutes the signal — and AI
produces those fast because it doesn't know what matters."

### Q3: Why is review harder with AI, not easier?
**A:** "Because AI-generated bad tests look good — clean, passing locally — so they slip a
casual review. Hand-written bad tests usually look bad. The risk is a large green suite
that proves nothing, and that only shows up when a real regression walks past it."

### Q4: Doesn't the gate cancel the speed-up?
**A:** "It slows merging on purpose, and I'm fine with that. The speed-up is in drafting;
I don't let it leak into review. I still come out net faster, but with a suite I can
defend — which is the only kind worth having."

## Weak areas to watch

- Don't describe AI use without the gate — the gate is what makes the story credible.
- Don't treat "passes locally" as a gate; that's the exact trap.
- Don't merge a test you can't justify because deleting feels wasteful — noise costs more.
- Don't claim the efficiency gain came from skipping review; it came with the gate.

## Key takeaway

> "AI moves the effort from writing tests to reviewing them, so I put a fixed gate on
> every generated test — and the first question is always 'why does this exist?'. The
> real risk of AI in testing is a fast green suite that proves nothing, and the gate is
> what stops it."
