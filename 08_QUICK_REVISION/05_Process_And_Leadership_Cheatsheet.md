# Process & Leadership Cheat Sheet — Ways of Working + Senior Round

> Dense recall for `05_PROCESS_AND_TOOLING` and `06_LEADERSHIP_AND_ARCHITECTURE`.
> The manager/senior round is won on trade-offs — every answer here names a cost.

---

## Agile / Scrum / Jira

- **Ceremonies:** Sprint Planning, Daily Standup, Review (demo), Retro. QA is in *all* of them, not a downstream gate.
- **The mini-waterfall trap:** dev codes days 1-8, "throws over the wall" to QA days 9-10 → crunch, no time to fix. Fix: test in-sprint, shift-left, define testability with dev up front.
- **DoD (Definition of Done):** includes tests written + passing + reviewed, not just "code merged."
- **QA in a sprint:** refine acceptance criteria, write test cases alongside dev, automate as features stabilize.

---

## Git workflow

- **Branching:** feature branches, PRs, review before merge. Same discipline for test code as app code.
- **Conflicts in test code:** usually in shared page objects/config — resolve by understanding both changes, not blindly.
- **Committed secrets:** the classic incident. Never commit credentials; use env/secrets store; if leaked, rotate immediately (removing from history isn't enough — it's already out).

---

## Docker & Kubernetes (why it matters to tests)

- **Docker:** image/version match between local, CI, and prod. Version drift = "works on my machine." Fail-fast on mismatch.
- **Kubernetes rolling deploys break tests:** the target can swap mid-run → in-flight requests hit a dying pod. Fix: **readiness checks** before testing, retry on the right transient conditions.
- Env config via env vars/config maps, not hardcoded.

---

## Tooling one-liners

- **Postman/Newman:** Postman for exploration; Newman (CLI) for CI. Newman's limit: not a full framework — no complex data setup, weaker structure. Graduate real regression to code.
- **Allure:** **history is the differentiator** — shows flakiness trends over time, not just this run. `failed` (assertion) vs `broken` (unexpected error) distinction. Cross-stack (works with both).

---

## Leadership: Tool Selection (the framework)

Decide on: team skills, ecosystem fit, project maturity, maintenance cost, hiring pool — **not** hype.

**Playwright vs Cypress vs Selenium:**
- Playwright: multi-tab/context, true parallel, cross-browser, UI+API, modern. Younger ecosystem.
- Cypress: great DX, time-travel debug. Single-tab limits, no true parallel (free), JS-only.
- Selenium: mature, huge ecosystem, any language, grid. More boilerplate, you manage waits.

**The senior line:** *"I'd tell a Java team with a working Selenium grid NOT to migrate — the migration cost outweighs the gain. I can say that credibly because I've built in both."* **Knowing when NOT to pick Playwright is the signal.**

---

## Leadership: Architecture Decisions

Every decision = a trade-off. Have these ready:
- **Fixtures vs hooks** — composition & laziness vs learning curve.
- **Storage-state auth** — speed vs the login-coverage gap.
- **Abstract early vs late** — DRY vs wrong abstraction (wait for the 3rd repetition).
- **UI vs API test level** — confidence in wiring vs speed/stability.
- **Retries** — safety net for real externals vs hiding races.

*"What trade-offs did you make?" is the whole round. If an answer has no cost, it's not senior.*

---

## Leadership: Mentoring & Code Review

- **Onboarding a junior:** pair on a real test day one (goes into the actual suite), they drive the next, you review the third. Convention lands on *their* code, not a wiki.
- **Code review:** lead with impact (flaky sleep) not style (naming); explain the cost; separate must-fix from nice-to-have; teach, don't just flag. See `10/03`.
- **Reviewing without being the bottleneck:** automate the mechanical (linters/formatters) so humans review logic and design.

---

## Leadership: Test Automation Strategy

- **What to automate:** stable, repetitive, high-value, run often. **Not:** one-offs, rapidly-changing UI, things cheaper to check manually.
- **The pyramid:** many unit/API, few UI. Push coverage down for speed and stability.
- **ROI framing:** automation is an investment — it costs to build and maintain. Automate where it pays back, leave the rest exploratory/manual.
- **"How do you decide what to automate?"** → risk × frequency × stability. High on all three = automate first.

---

## The senior tells (weave these in)

- Name a **cost** in every design answer.
- Say when you'd **not** do the "modern" thing (don't migrate, don't automate, don't add retries).
- Separate **"I" vs "we"** cleanly.
- Frame quality as a **team/velocity** concern, not a QA silo.

---

## Key takeaway

> **The senior round is won on trade-offs — an answer with no named cost is a mid-level
> answer.** Your unique asset is having built in both paradigms, which lets you say the
> most senior thing there is: *"here's when I would NOT do the shiny thing, and why."*
> Everything ladders to owning decisions, not implementing them.
