# Company-Specific Prep — A Research Approach

## What it is

A repeatable method for preparing for a *specific* company's interview, rather than generic prep. Not a list of companies — a process you run in 30-45 minutes per company that tells you what to emphasize, what they'll probe, and what smart questions to ask back.

---

## Why it matters

Generic prep gets you through the technical rounds. Company-specific prep is what makes an interviewer think *"this person actually wants to work here."* At the 20-24 band, fit and intent are part of the assessment — two technically-equal candidates get separated by who researched the role. It also protects your negotiation: knowing a company's stack, scale, and pain points lets you position your two-track story toward exactly what they need.

You're applying at volume through the [job pipeline](../../../JOB_PIPELINE/), so this has to be *fast and repeatable*, not a research project per company.

---

## The 30-minute research pass

Run this before any scheduled interview. Five inputs, one output.

| # | Input | What you're looking for | Where |
|---|-------|-------------------------|-------|
| 1 | **The JD, re-read closely** | Their exact vocabulary, must-haves vs nice-to-haves, the stack | The listing |
| 2 | **The stack** | Playwright or Selenium? Cypress? Java or JS/TS shop? Cloud? | JD + engineering blog + job board history |
| 3 | **Scale & domain** | Product company vs services? Data-heavy? B2B/B2C? Regulated? | Company site, LinkedIn, Glassdoor |
| 4 | **The team/role level** | IC or lead? Greenfield or maintenance? Who you'd report to | JD title + LinkedIn of the team |
| 5 | **Recent signals** | Funding, product launches, hiring surge, tech migration | News, LinkedIn posts, their blog |

**Output:** a 4-line prep card —
1. *Emphasize:* (which track/skills to lead with)
2. *They'll probe:* (the 2-3 topics their stack guarantees)
3. *My gap to defend:* (anything in the JD I'm light on)
4. *Questions to ask them:* (2-3 specific ones)

---

## Positioning by company type

Your two-track story is an asset because you can aim it. Read the company, then pick the lead.

**Modern product company / greenfield / TS shop** → Lead Playwright hard.
> Framework ownership, CI 40→12, UI+API in one framework, AI-assisted workflows. Selenium is your "range" mention, not your lead.

**Enterprise / services / Java shop / mature suite** → Lead Selenium + Java depth.
> ThreadLocal parallel stability, flakiness under 2%, RestAssured, SQL/DB validation, Jenkins. Playwright becomes "and I've also moved a team to modern tooling" — which is *exactly* what a legacy shop modernizing wants to hear.

**Cypress shop** → Lead Playwright, but be ready to compare honestly.
> Know why Playwright beats Cypress (multi-tab/context, true parallelism, cross-browser) *and* where Cypress is genuinely nice (DX, time-travel debugging). Don't trash their tool — you might be brought in to migrate it. See [Tool Selection](../../06_LEADERSHIP_AND_ARCHITECTURE/01_Tool_Selection_Framework.md).

**Data / analytics domain (like your BASES work)** → Lean into DB validation + data-heavy testing.
> This is where your SQL, database testing strategy, and data-management notes shine. You've validated data-heavy products for years — say so.

---

## Questions to ask them (and why)

Asking good questions is half the fit signal. Prepare 3-4 per company; ask 2-3. Aim for questions that reveal you think like an owner.

**Strong, senior-signal questions:**
- "What does the test suite's health look like today — flakiness, CI time, coverage? What's the biggest pain point?" *(You're the person who fixes exactly this.)*
- "Is this role owning an existing framework or building/migrating one? What state is it in?"
- "How do QA and dev split responsibility for test IDs and testability?" *(Signals your influence-without-authority experience.)*
- "What's the CI/CD setup, and how long is the feedback loop today?" *(Your 40→12 story is right there.)*

**Avoid:** questions answered on their careers page, salary/leave in a technical round (save for HR), or anything that reads as "what's in it for me" before you've shown value.

---

## Real project example

For a data-heavy analytics company (your natural fit): the prep card writes itself —
1. *Emphasize:* BASES Playwright data-validation + DB testing; the two-track range.
2. *They'll probe:* how you validate large datasets, test-data isolation, DB-to-UI consistency.
3. *My gap to defend:* if they use a tool I don't (e.g. a specific data pipeline), acknowledge and bridge from what I know.
4. *Questions:* suite health, framework state, data-seeding strategy.

For a Java enterprise modernizing to Playwright — the *perfect* fit for your story:
1. *Emphasize:* "I've done exactly this transition — two years Selenium/Java, then owned the Playwright migration."
2. *They'll probe:* migration strategy, what to migrate vs leave, team adoption.
3. *Questions:* "What's driving the migration, and what's the current suite's state?"

---

## Common interview Q&A

**Q: "Why do you want to work here specifically?"**
Never "for growth" or "great company." Tie a *specific* thing you found to *your* strength. *"You're moving from Selenium to Playwright — that's the exact transition I've led, so I can contribute from week one rather than ramping up."*

**Q: "What do you know about what we do?"**
Two sentences on their product/domain, then connect to your experience. Shows you researched without reciting their About page.

**Q: "Do you have any questions for us?"**
Always yes. Never "no, you covered everything" — it reads as low interest. Two prepared questions minimum.

---

## Weak areas to watch

- **Don't over-invest per company.** You're applying at volume. 30 minutes, a 4-line card, done. Save the deep prep for confirmed later-round interviews.
- **Don't fake enthusiasm you can't back.** "I love your mission" with nothing specific behind it is worse than honest professional interest.
- **Don't trash their current tooling** — you might be hired to work with or migrate it. Compare honestly, respect the context.
- **Don't ask salary/leave in technical rounds.** Wrong audience, wrong time. That's the [HR round](../03_HR_AND_SALARY/03_HR_And_Salary.md).
- **Match your lead to their stack.** Leading Selenium at a modern TS startup, or Playwright at a Java-only enterprise, wastes your best material.

---

## Key takeaway

> **30 minutes, five inputs, a 4-line prep card — then aim your two-track story at what they actually need.** Company-specific prep isn't about memorizing their history; it's about walking in knowing which of your two stacks to lead with and which two questions prove you think like an owner. Fit is assessed, and it's cheap to win.
