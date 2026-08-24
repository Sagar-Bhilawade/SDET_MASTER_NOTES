# AI In Testing - AI-Assisted Bug Drafting And Defect Quality

## What it is

Using AI to improve the *write-up* of a defect — turning rough notes and evidence into
a clear, reproducible, well-structured bug report — without letting it invent the facts.

A good bug report has a stable shape: a precise title, exact reproduction steps,
expected vs actual, environment, evidence (logs, trace, screenshot), and a severity
that reflects real impact. AI is good at *formatting and clarity* — taking messy notes
and producing that shape. It is bad, and dangerous, at *facts* — it will happily invent
a plausible reproduction step that you never actually performed.

So the split is sharp: you supply the observed facts and the evidence; AI improves the
wording, structure, and completeness. It never authors the reproduction.

## Why it matters

Defect quality is a QA fundamental (it links straight to
`01_TESTING_FUNDAMENTALS/05_Defect_Lifecycle_And_Reporting`). A bug a developer can't
reproduce gets sent back, and the round-trip is expensive. Interviewers ask about this
to see if you understand that the report is a communication artifact, not a formality.

The AI angle tests your discipline. The tempting failure is letting AI "fill in" the
repro to make the report read cleanly. That produces a report that reads well and is
subtly false — the worst kind, because it wastes the developer's time and burns your
credibility. Showing you use AI for clarity but guard the facts is the mature answer.

## 30-second answer

> "I use AI to sharpen the bug write-up — clearer title, tighter repro steps, clean
> expected-vs-actual, proper structure — but I supply the facts and evidence myself. It
> never invents a reproduction step. The goal is a report a developer can reproduce on
> the first read; AI improves the wording and completeness, not the truth of what I
> observed."

## 2-minute answer

> "A defect report is a communication artifact. Its whole job is to let a developer
> reproduce and fix without a round-trip, so clarity and accuracy both matter — and they
> pull in different directions when you add AI.
>
> Where AI genuinely helps is the shape and the wording. I capture rough notes during
> testing — what I did, what I expected, what happened, plus the trace or logs. AI turns
> that into a structured report: a precise title, ordered reproduction steps, clear
> expected vs actual, environment, and it prompts me for anything missing, like 'which
> browser' or 'is this consistent or intermittent'. That completeness check is real
> value — it catches the gap before the developer does.
>
> Where I hold a hard line is the facts. AI will invent a plausible repro step to make
> the report flow, and a bug report that reads cleanly but is subtly false is worse than
> a messy true one — the developer can't reproduce it, sends it back, and I've cost two
> people time and some credibility. So AI never authors the reproduction; it only
> formats what I actually observed.
>
> Severity is the other thing I keep. AI can suggest one, but severity is business
> impact and I own that call. The trade-off is simple: AI raises the *floor* of my report
> quality and speeds up the write-up, but the facts, the evidence, and the severity stay
> mine."

## Anatomy of the report — who supplies what

| Part | Source | AI's role |
|---|---|---|
| Title | You | Sharpen to be specific and searchable |
| Reproduction steps | **You — observed only** | Order and clarify, never invent |
| Expected vs actual | You | Phrase crisply |
| Environment | You | Prompt for what's missing |
| Evidence (trace/log/screenshot) | You | — |
| Severity / priority | **You — business call** | May suggest, you decide |
| Completeness | — | Flag gaps before the developer sees them |

## Real project example

> "When I hit a defect I capture rough notes and the evidence — a Playwright trace or the
> logs — while it's fresh. Then I use AI to turn that into a clean report: precise title,
> ordered repro, expected vs actual, environment. It also nudges me on gaps, like whether
> the issue is consistent or intermittent, which is exactly what a developer asks first.
> But I've caught it trying to 'complete' a repro with a step I never did, so the rule is
> firm — it formats what I observed, it doesn't author the reproduction. Severity stays my
> call because it's about impact on the release. The result is fewer 'can't reproduce'
> bounce-backs, which is the whole point of a good report."

## Common interview Q&A

### Q1: How do you use AI for bug reports?
**A:** "For clarity and completeness. I supply the observed facts and evidence; AI
tightens the title, orders the repro steps, phrases expected-vs-actual cleanly, and
flags missing details. The aim is first-read reproducibility."

### Q2: What's the danger?
**A:** "It invents plausible repro steps to make the report flow. A clean-but-false bug
is worse than a messy true one — the developer can't reproduce it and it bounces back. So
AI never authors the reproduction; it only formats what I actually did."

### Q3: Does AI decide severity?
**A:** "No. It can suggest, but severity is business impact and that's my call. A tool
doesn't know what the release can tolerate."

### Q4: What actually improves from using AI here?
**A:** "The floor. My worst-written report gets better, and completeness gaps get caught
before the developer sees them. That reduces round-trips, which is where defect
reporting actually costs the team time."

## Weak areas to watch

- Never let AI invent or 'complete' a reproduction step — facts are yours.
- Don't outsource severity; it's a business-impact judgment.
- Don't paste real defect content, internal logs, or app details into an interview.
- Don't confuse a well-formatted report with a correct one — clarity ≠ accuracy.

## Key takeaway

> "AI raises the floor on bug-report quality — clearer, more complete, faster — but the
> reproduction, the evidence, and the severity are facts I own. A clean report that's
> subtly false is worse than a messy one that's true."
