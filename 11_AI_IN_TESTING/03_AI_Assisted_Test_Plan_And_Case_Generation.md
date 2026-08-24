# AI In Testing - AI-Assisted Test Plan And Functional Case Generation

## What it is

Using AI to draft the test artifacts — a test plan structure, functional test cases,
negative and boundary cases — from a requirement, so the tester spends time *editing
and pruning* instead of typing from a blank page.

The key reframe: AI is good at **breadth**, weak at **depth and priority**. It will
happily list twenty cases. Deciding which five actually matter, which three are
duplicates, and which one edge case is the real risk — that's the QA work, and it's
where AI is worst.

So the workflow is inverted from what people expect. You don't ask AI for "the test
cases." You ask it for a wide first draft, then you *cut*. Good QA with AI is mostly
deletion.

## Why it matters

Case-writing is where AI most tempts you into shallow coverage. It produces cases that
look thorough — right steps, plausible titles — but assert the wrong thing or stack up
near-duplicates. An interviewer who asks about this is checking whether you can tell
good coverage from voluminous coverage.

It also connects to test design technique (EP, BVA, decision tables). AI can *apply*
those techniques mechanically, but only if you know them well enough to check its
work. This is where your ISTQB foundation pays off: you're grading the draft against
technique you already own.

## 30-second answer

> "I use AI to draft a wide first set of cases from a requirement — happy path,
> negatives, boundaries — then I cut hard. AI is good at breadth but bad at priority
> and depth, so most of my work is deleting duplicates and shallow cases and fixing the
> assertions. It removes the blank-page cost; it doesn't decide coverage."

## 2-minute answer

> "My mental model is that AI gives me breadth for free and I supply depth and priority.
>
> I start by feeding the requirement and asking for a structured draft: a plan outline
> plus functional cases across happy path, negative paths, and boundaries. It's fast and
> it rarely misses an obvious category, which is genuinely useful — the blank page is
> the expensive part.
>
> Then I do the real work, which is mostly subtraction. I remove near-duplicates, drop
> cases that assert visibility instead of behavior, and merge cases that test the same
> risk. I check it against actual design techniques — is the boundary analysis right, are
> the equivalence partitions real, does the decision table cover the true combinations.
> AI applies those techniques mechanically, so it needs someone who knows them to catch
> where it went wide but shallow.
>
> The last step is priority, which AI can't do because it doesn't know the business
> risk. I mark what's smoke, what's regression, what's exploratory-only. The trade-off
> I'm always managing: AI makes it cheap to generate *more* cases, and more cases isn't
> better — a bloated suite is slower and hides the important checks. So the discipline
> is to let AI expand, then aggressively cut back to what proves the behavior."

## Real project example

> "For a new feature on the Playwright track, I'll draft the functional cases with AI
> from the story and acceptance criteria — it gives me the happy path plus a spread of
> negatives and boundaries in one pass. Then I prune. On a recent one it generated
> several cases that all really tested the same validation rule with different labels, so
> I collapsed them into one parametrized case and added the one boundary it had missed.
> The output I kept was maybe half of what it produced, but I got there in a fraction of
> the time it takes to write from scratch. The judgment — which cases matter, what level
> they belong at, what's smoke vs regression — stayed mine."

## What AI is good and bad at here

| Task | AI strength | Where you take over |
|---|---|---|
| Listing case categories | High — rarely misses a category | — |
| Expanding negatives/boundaries | High — fast breadth | Verify boundaries are *correct*, not just present |
| Applying EP/BVA/decision tables | Mechanical | Confirm partitions and combinations are real |
| Writing assertions | Weak | Fix "asserts visibility, not behavior" |
| Removing duplicates | Weak | Merge cases testing the same risk |
| Assigning priority | Cannot — no business context | Mark smoke / regression / exploratory |

## Common interview Q&A

### Q1: How do you use AI to write test cases?
**A:** "For breadth. It drafts a wide set fast, then I cut hard — remove duplicates, fix
shallow assertions, and set priority. Good QA with AI is mostly deletion, because more
cases isn't better coverage."

### Q2: Doesn't AI-generated coverage look complete but miss things?
**A:** "Yes, that's the main risk. It writes plausible cases that assert the wrong thing
or stack near-duplicates. I grade the draft against real design techniques — BVA, EP,
decision tables — which is why knowing those matters more, not less, when you use AI."

### Q3: What can't AI do in case design?
**A:** "Priority. It doesn't know the business risk, so it can't tell you which case is
smoke and which is a nice-to-have. That ranking is the QA decision, and it's the part I
never delegate."

### Q4: How do you keep the suite from bloating?
**A:** "By treating AI's output as a candidate list, not the final set. I merge, cut,
and parametrize. A bloated suite runs slower and buries the important checks, so I
optimize for the fewest cases that prove the behavior."

## Weak areas to watch

- Don't present the raw AI output as your test plan — the value is in what you cut.
- Don't skip the technique check; AI applies BVA/EP mechanically and gets boundaries
  wrong.
- Don't let case count grow just because generation is cheap.
- Don't paste the real requirement into an interview — describe the method.

## Key takeaway

> "AI gives breadth; I supply depth and priority. It makes generating cases cheap, so
> the skill shifts to cutting — the strongest QA use of AI is disciplined deletion."
