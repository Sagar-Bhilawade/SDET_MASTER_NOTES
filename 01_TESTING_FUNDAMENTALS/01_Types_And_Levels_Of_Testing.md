# Types & Levels of Testing

## 1. What It Is

Testing types explain **what quality risk** we are checking. Testing levels explain
**where in the system** we are checking it.

For interviews, do not only define smoke, sanity, regression, and retesting. Show that
you know how to use them in real sprint work: what should be manual, what should be
automated, what should block CI, and what should run later.

## 2. Why It Matters In Interviews

This is usually a warm-up question. If the answer is confused, the interviewer assumes
your foundation is weak. If you answer cleanly and add operational judgement, you sound
like someone who has tested real releases.

Strong signal:

- You separate testing **type** from testing **level**.
- You know smoke vs sanity and regression vs retesting instantly.
- You know automation is strongest for repeated regression, not for every human check.
- You can explain the test pyramid as a cost and stability decision.

## 3. Complete Topic Breakdown

### Testing types

| Type                   | What it checks                               | Practical usage                                 |
| ---------------------- | -------------------------------------------- | ----------------------------------------------- |
| Functional testing     | Whether feature behavior matches requirement | Login, search, filters, exports, calculations   |
| Non-functional testing | How well the system works                    | Performance, security, usability, accessibility |
| Smoke testing          | Whether the build is testable                | Fast build gate after deployment                |
| Sanity testing         | Whether a specific change/fix is reasonable  | Focused check before deeper testing             |
| Regression testing     | Existing behavior after a change             | Main automation candidate                       |
| Retesting              | Specific defect fix                          | Same bug, same steps, new build                 |
| Exploratory testing    | Unknown risks and surprising behavior        | Charter-based investigation                     |
| UAT                    | Business acceptability                       | Product/user sign-off                           |
| Compatibility testing  | Supported browser/device/platform behavior   | Cross-browser and responsive checks             |

### Testing levels

| Level             | Scope                           | Best use                      |
| ----------------- | ------------------------------- | ----------------------------- |
| Unit              | Function/class                  | Logic and small branches      |
| Component/service | One module/service              | API contracts and validation  |
| Integration       | Multiple components together    | Service-to-service, API-to-DB |
| System            | Whole application               | Full feature behavior         |
| End-to-end        | Business journey across systems | Critical user flows only      |
| Acceptance/UAT    | Business workflow               | Final business confidence     |

### Classic pairs

**Smoke vs sanity**

- Smoke is shallow and wide: is the build worth testing?
- Sanity is narrow and focused: does this specific change look okay?

**Regression vs retesting**

- Retesting proves the reported defect is fixed.
- Regression proves existing behavior did not break because of the fix/change.

**Functional vs non-functional**

- Functional is "what the system does".
- Non-functional is "how well the system does it".

## 4. Manual Testing Angle

Manual testing is strongest when the feature is new, changing, ambiguous, visual,
business-heavy, or needs human judgement.

Manual QA focus:

- Understand the story and acceptance criteria.
- Identify positive, negative, boundary, role, permission, data, and integration cases.
- Execute exploratory testing around new or risky behavior.
- Observe usability, layout, confusing error messages, and unexpected side effects.
- Capture evidence when behavior differs from expectation.

Manual testing is not "just clicking". It is risk discovery, requirement clarification,
and release judgement.

## 5. Automation Angle

Automation is strongest when the check is repeated, stable, important, and has a clear
expected result.

Automate:

- Smoke suite for CI gate.
- Stable regression flows.
- API validations for business rules.
- Cross-browser smoke for critical flows.
- Regression for repeated production or sprint defects.

Keep manual:

- Exploratory testing.
- Usability judgement.
- One-time checks.
- Newly changing screens.
- Ambiguous expected results.

Key trade-off:

> UI automation gives business confidence but costs more to maintain. Push logic checks
> to API/component level when possible and keep E2E only for true user journeys.

## 6. Exact Surface To Remember

- Smoke: "Is this build worth testing?"
- Sanity: "Does this specific change look reasonable enough to continue?"
- Retesting: "Same defect, same steps, fixed build."
- Regression: "Everything else still works after the change."
- Exploratory: "Structured investigation with a charter, not random clicking."
- Test pyramid: "Most checks low, fewer checks high, because cost and flakiness increase upward."

## 7. Real Project Example

In NielsenIQ work, position it like this:

> "I handle both manual and automation coverage. For a new feature, I first validate
> requirements and run manual/exploratory checks because behavior may still change.
> Once the flow stabilizes, I convert the repeated checks into automation. Fast smoke
> flows go into CI, broader regression runs for release confidence, and exploratory
> testing stays manual because automation only checks what we already predicted."

## 8. Common Mistakes / Weak Areas

- Saying smoke and sanity are exactly the same.
- Saying regression means rerunning the failed test.
- Saying everything should be automated.
- Ignoring exploratory testing.
- Putting every validation into UI automation.
- Talking only textbook definitions without release strategy.

## 9. 30-Second Answer

Testing types describe the kind of risk, like functional, regression, smoke, usability,
or performance. Testing levels describe the system scope, like unit, integration, system,
and E2E. In real work I use these to decide what should be manual, what should be
automated, and what should block a build. Regression is the main automation candidate,
while exploratory and usability checks need human judgement.

## 10. 2-Minute Answer

I separate the classification from the execution strategy. Smoke testing is a quick,
wide build gate. Sanity is a focused check around a specific change. Retesting verifies
the original defect is fixed, while regression checks that nearby or existing behavior
has not broken.

The level decision is where SDET judgement comes in. If a rule can be tested at API or
component level, I do not push every case through the UI because that makes the suite
slow and fragile. UI/E2E tests are reserved for critical user journeys. Manual testing
still matters for new features, exploratory checks, usability, and unclear behavior.
So manual and automation are not competing tracks; manual finds and shapes the risk,
automation preserves the stable repeated checks.

## 11. Interview Q&A

### Q: Smoke vs sanity?

Smoke is shallow and wide. It checks whether the build is worth testing. Sanity is
narrow and focused. It checks whether one specific fix or change looks reasonable.

### Q: Regression vs retesting?

Retesting verifies the specific defect is fixed. Regression verifies existing behavior
still works after the change.

### Q: Is exploratory testing still needed when automation exists?

Yes. Automation checks known expectations. Exploratory testing discovers unknown risks,
requirement gaps, usability problems, and unexpected side effects.

### Q: What should be automated first?

Critical, stable, repeated flows with clear expected results: smoke, release regression,
API validations, and repeated defect-prone areas.

## 12. Quick Revision Points

- Smoke = build worth testing.
- Sanity = specific change is reasonable.
- Retesting = same defect fixed.
- Regression = existing behavior still works.
- Automation preserves repeated known checks.
- Manual testing discovers risk and gives judgement.
