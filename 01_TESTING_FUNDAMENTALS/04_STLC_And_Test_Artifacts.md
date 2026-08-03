# STLC & Test Artifacts

## What This Covers

The software testing lifecycle, and the documents that come out of it — test plan, strategy, scenarios, cases, traceability matrix, and entry/exit criteria.

---

## Why It Matters (Interview Context)

Two questions come up reliably:

> "Walk me through the STLC."
> "What's the difference between a test scenario and a test case?"

The risk is answering like a textbook. Everyone can recite the phases. What distinguishes a real answer is knowing **which artifacts genuinely earn their keep and which are ceremony** — because in an agile team, half of the classical documentation set doesn't get produced and shouldn't.

Having an opinion on that is the senior signal.

---

## Your 30-Second Interview Answer

> "STLC runs requirement analysis, test planning, test case design, environment setup, execution, then closure — each with entry and exit criteria. On artifacts, a test strategy is organization-level and rarely changes; a test plan is project-level and specific. A scenario is what to test at a high level; a test case is the detailed steps to verify it. In an agile team I'd be honest that we produce far less formal documentation than the classical model implies — acceptance criteria in the story often replace a separate test case document."

---

## Your 2-Minute Deep Answer

> "**The STLC phases:**
>
> **1. Requirement analysis** — understand what's being built and, critically, whether it's testable. Ambiguous acceptance criteria caught here cost nothing; caught in execution they cost a sprint.
>
> **2. Test planning** — scope, approach, resources, schedule, risks. Produces the test plan.
>
> **3. Test case design** — scenarios and cases derived using the design techniques: equivalence partitioning, boundary values, decision tables.
>
> **4. Environment setup** — often the phase that actually blocks things. Can run in parallel with design.
>
> **5. Test execution** — run, log defects, retest, regression.
>
> **6. Test closure** — summary report, metrics, lessons learned. The phase most often skipped, and the one that would tell you whether the process worked.
>
> Each phase has **entry criteria** (what must be true to start) and **exit criteria** (what must be true to finish).
>
> ---
>
> **Test Strategy vs Test Plan** — commonly confused:
>
> | | Test Strategy | Test Plan |
> |---|---|---|
> | Level | Organization | Project or release |
> | Changes | Rarely | Per project |
> | Contains | Standards, tools, general approach | Scope, schedule, resources, risks |
> | Owner | QA lead / manager | Test lead |
>
> The strategy says *"we automate regression, we use Playwright for web, defects above severity 2 block release."* The plan says *"for this release, we're testing these modules, with these people, on this timeline."*
>
> ---
>
> **Test Scenario vs Test Case** — the other classic:
>
> **Scenario** — a high-level statement of what to test.
> > "Verify user can log in"
>
> **Test case** — the detailed, executable version.
> > Preconditions, steps, test data, expected result, actual result.
>
> One scenario typically produces several test cases:
> ```
> Scenario: Verify user login
>   TC1: Valid credentials → dashboard loads
>   TC2: Invalid password → error message shown
>   TC3: Locked account → appropriate message
>   TC4: Empty fields → validation triggers
> ```
>
> The relationship is one-to-many, and that's usually what the question is really checking.
>
> ---
>
> **RTM — Requirement Traceability Matrix:**
>
> Maps requirements to test cases, both directions:
> - **Forward** — is every requirement covered by at least one test?
> - **Backward** — does every test trace to a requirement? (If not, why does it exist?)
>
> The value is gap detection: a requirement with no test is a coverage hole, and a test with no requirement is either scope creep or a test nobody can justify.
>
> In practice, on an agile team this is usually achieved by linking tests to stories in Jira rather than maintaining a separate spreadsheet — same function, less overhead.
>
> ---
>
> **Entry / Exit criteria — example:**
>
> ```
> Entry to system testing:
>   ✓ Integration testing complete
>   ✓ Test environment available and stable
>   ✓ Build deployed and smoke suite passing
>   ✓ Test data prepared
>
> Exit from system testing:
>   ✓ All planned cases executed
>   ✓ No open defects above agreed severity
>   ✓ Regression suite passing
>   ✓ Test summary report signed off
> ```
>
> The exit criteria matter more than the entry ones, because that's where the release conversation actually happens.
>
> ---
>
> **The honest opinion I'd add:**
>
> In an agile team, most of this classical documentation set is either lightweight or absent. A detailed test plan document per two-week sprint is overhead nobody reads. Acceptance criteria in the story often replace a separate test case document entirely.
>
> What survives and genuinely earns its place:
> - **Exit criteria** — because that's the release conversation
> - **Traceability** — via story links, not a spreadsheet
> - **The test summary** — because escape rate and what you learned is how the process improves
>
> Saying that is better than reciting the full waterfall artifact list, because it shows you understand which documents create value and which create ceremony. But I'd know the classical model too — some organizations, particularly regulated ones, genuinely do produce all of it, and dismissing it would be wrong."

---

## STLC Phases

| Phase | Output |
|-------|--------|
| Requirement analysis | Testability questions, RTM started |
| Test planning | Test plan |
| Test case design | Scenarios, cases, test data |
| Environment setup | Ready environment |
| Test execution | Results, defect reports |
| Test closure | Summary report, metrics, lessons |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "In practice we work in sprints, so the classical STLC compresses considerably. Requirement analysis happens at refinement — that's where I ask testability questions, and it's the cheapest place to find a gap. Design and execution overlap with development rather than following it.
>
> On artifacts, I'd be straightforward that we don't produce a formal test plan document per sprint — it would be overhead nobody reads. What we do maintain is traceability through Jira, linking tests to the stories they cover and defects to the story that introduced them. That gives you the same coverage-gap detection an RTM would, without a spreadsheet to maintain.
>
> The artifact I'd argue hardest for is exit criteria, because that's where the release conversation actually happens. 'No open defects above severity 2, regression passing' is a concrete gate. Without it, ship decisions become opinion.
>
> And closure — the phase most often skipped — is the one that tells you whether the process is working. Escape rate per release is worth more than the count of defects logged, because it measures outcome rather than activity."

---

## Common Interview Questions & Answers

### Q1: "Walk me through the STLC."

**A:** "Requirement analysis, planning, test case design, environment setup, execution, closure — each with entry and exit criteria. In an agile team it compresses heavily: analysis happens at refinement, and design and execution overlap with development rather than following it. Closure is the phase most often skipped, and it's the one that tells you whether the process worked."

### Q2: "Test scenario vs test case?"

**A:** "A scenario is the high-level statement of what to test — 'verify user can log in.' A test case is the executable detail: preconditions, steps, data, expected result. One scenario produces several cases — valid credentials, invalid password, locked account, empty fields. The relationship is one-to-many."

### Q3: "Test strategy vs test plan?"

**A:** "Strategy is organization-level and rarely changes — which tools, what we automate, what severity blocks a release. The plan is project-specific — scope, schedule, resources, risks for this release. Strategy is the standing policy; the plan applies it to a particular piece of work."

### Q4: "What's an RTM and do you use one?"

**A:** "A traceability matrix maps requirements to tests both ways — every requirement has a test, and every test traces to a requirement. Its value is gap detection. In practice on an agile team I'd achieve that by linking tests to stories in Jira rather than maintaining a separate spreadsheet — same function, much less overhead."

### Q5: "How much test documentation do you actually produce?"

**A:** "Less than the classical model implies, deliberately. A formal test plan per two-week sprint is overhead nobody reads. What earns its place is exit criteria, because that's the release conversation; traceability through story links; and a closure summary, because escape rate is how you find out whether the process works. I'd know the full classical set — regulated environments genuinely produce all of it — but I'd be honest about what adds value in an agile team."

---

## Weak Areas to Watch

❌ **Don't recite phases without the agile reality**
→ Textbook recall reads as theory, not practice.

❌ **Don't confuse strategy and plan**
→ Organization-level vs project-level is the distinction.

❌ **Don't say scenario and test case are the same**
→ One-to-many. It's the point of the question.

❌ **Don't dismiss the classical model entirely**
→ Regulated industries produce all of it. Know it, then give your opinion.

---

## Key Takeaway for Interview

Know the model, then have an opinion about what earns its place.

The distinction to have automatic:

> **"A scenario is what to test. A test case is how to verify it. One scenario, several cases."**

And the answer that shows judgement rather than recall:

> **"In an agile team most of the classical documentation is overhead nobody reads. What survives is exit criteria — because that's the release conversation — traceability through story links, and a closure summary, because escape rate is how you learn whether the process works."**

---

## Checklist

✅ Six STLC phases with entry/exit criteria
✅ Strategy vs plan — organization vs project
✅ Scenario vs case — one-to-many
✅ RTM purpose, and the Jira-link alternative
✅ Concrete exit criteria example ready
✅ Opinion on agile documentation, without dismissing the classical model

---

**Remember:** Everyone can list the phases. Having a view on what's worth producing is the differentiator. 🎯
