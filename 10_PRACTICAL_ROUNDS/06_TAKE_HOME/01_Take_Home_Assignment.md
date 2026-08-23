# Take-Home Assignment — Scoping Guide

## What it is

The assignment they send you to do on your own time: *"Build an automation framework for this site,"* or *"Write tests for this API and send us the repo."* It's an open-ended deliverable with a time expectation, and it's graded by an engineer who will read your code. This note is about **scoping it right** — doing enough to stand out without sinking a weekend, and packaging it so it reads as senior work.

---

## Why it matters

The take-home is a double-edged round. Done well, it's your strongest asset — a real repo that proves everything your resume claims, reviewed at leisure by someone who'll be impressed by craft. Done wrong, it's where good candidates get rejected: they either over-build (a week of work signalling poor prioritization) or under-build (a single test file that looks junior). The skill being tested is partly technical and partly **judgment** — can you scope, prioritize, and communicate trade-offs? That's exactly the senior signal.

---

## The scoping principle

**They are not grading raw coverage. They're grading engineering judgment.** A framework with 6 well-structured tests, a clean README, and a "here's what I'd do with more time" section beats 40 messy tests every time.

Budget the time they suggest — if they say "2-3 hours," do 3, not 10. Going massively over signals you can't prioritize; going way under signals low effort. The move is to **scope tight, build well, document the rest.**

---

## What to actually build (the structure that reads senior)

1. **A real framework skeleton, not a flat script.**
   - Page Objects (or an API client layer for API assignments)
   - Config separated from tests (base URL, credentials via env, not hardcoded)
   - A fixture/setup layer for test data and auth
   - Clear folder structure — this alone separates you from 80% of submissions

2. **A focused set of tests that show range, not volume.**
   - 2-3 happy paths
   - 2-3 negative/boundary cases (proves you're a *tester*, not a dev writing smoke tests)
   - One test that demonstrates something senior: data isolation, API-level validation of a UI action, or a mocked external dependency

3. **CI config** — a GitHub Actions workflow that runs the suite. Even a simple one. It says "I ship, I don't just write."

4. **A README that does the talking** (see below).

5. **Clean git history** — a few meaningful commits, not one giant "final" dump. Reviewers look.

---

## The README is half the grade

Most candidates send code with a one-line README. Yours explains your *thinking*, which is what a senior reviewer is actually looking for. Include:

- **How to run it** — one command, clearly. If they can't run it in 60 seconds, you've lost points.
- **Architecture decisions, with trade-offs.** *"I used fixtures over hooks because they compose and evaluate lazily; the cost is a steeper learning curve for someone new to the framework."* This paragraph is where you win. Every decision, one cost.
- **What I tested and why** — and what I deliberately left out. *"I covered the happy path and top negatives at UI level, and pushed field validation to the API layer for speed. I didn't automate the rare edge cases — they'd be manual until they earn automation."*
- **"With more time I would..."** — scaling parallelism, visual testing, more negative coverage, DB validation. This turns a small submission into evidence of a big-picture thinker.

---

## Worked scoping example

**Assignment:** "Write UI + API automation for this demo e-commerce site. ~3 hours."

**Scope it as:**
| Deliverable | Effort | Why |
|-------------|:------:|-----|
| Playwright + TS skeleton, POM, config | 45 min | Structure = the senior signal |
| Login + add-to-cart happy path (UI) | 30 min | Core flow |
| 2 negatives: invalid login, out-of-stock | 30 min | Proves tester instinct |
| API: create order, validate body + schema | 30 min | Shows you push down the pyramid |
| One mocked external call (payment) | 20 min | Senior touch |
| GitHub Actions workflow | 15 min | "I ship" |
| README with decisions + "more time" | 30 min | Half the grade |

Total ~3.5 hrs, and every piece signals something. **Not** 30 UI tests and no README.

---

## Common interview Q&A (the follow-up discussion)

The take-home almost always has a review call. Prepare to defend every decision.

**Q: "Walk me through your framework."**
Same as your [Project Stories](../../07_INTERVIEW_PREP/02_PROJECT_STORIES/02_Project_Stories.md) muscle — decisions with costs, not a file tour.

**Q: "Why did you structure it this way?"**
Have a reason per layer. "Config separate so it runs in any environment; POM so a selector change touches one file."

**Q: "What would you add with more time?"**
Your README already answered this — repeat it confidently. Parallelism tuning, DB validation, visual tests, broader negatives.

**Q: "Why only N tests?"**
*"I scoped to demonstrate range and structure within the time, not maximize count. Coverage is about risk — I'd prioritize by what's most likely to break in production."* Judgment, framed as a strength.

---

## Weak areas to watch

- **Don't over-build.** A 15-hour submission signals poor prioritization, not dedication. Budget their stated time.
- **Don't send a flat script.** Structure — POM, config, fixtures — is the single biggest differentiator, even with few tests.
- **Don't skip the README.** Code without explained decisions reads as junior no matter how good the code is.
- **Don't hardcode credentials/URLs.** Env/config. A reviewer notices instantly, and it's a security-hygiene red flag.
- **Don't send one giant commit.** Clean, meaningful history signals how you actually work.
- **Make it run in one command.** If setup is fiddly, the reviewer's first impression is friction.
- **Only real assertions.** A take-home full of click-scripts with no meaningful assertions fails hardest — you're the *testing* hire.

---

## Key takeaway

> **Scope tight, build well, document the thinking — a 6-test framework with a decisions-and-trade-offs README beats 40 messy tests.** The take-home grades judgment as much as code: the structure (POM, config, fixtures, CI) and the README's "why I chose this, what it costs, what I'd add next" are where a senior reviewer decides you're worth 20-24. Never confuse coverage volume with quality of engineering.
