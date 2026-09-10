# Defect Lifecycle & Bug Reporting

## 1. What It Is

Defect lifecycle is the path a bug follows from discovery to closure. Bug reporting is
the skill of explaining a problem so developers can reproduce, understand, prioritize,
and fix it with minimum back-and-forth.

## 2. Why It Matters In Interviews

This is a core manual QA topic and also important for automation failures. Interviewers
want to know whether you can separate product defects from test/data/environment issues
and communicate impact clearly.

## 3. Complete Topic Breakdown

### Defect lifecycle

| State | Meaning | QA action |
|-------|---------|-----------|
| New | Bug logged | Add clear repro and evidence |
| Open/Assigned | Accepted for analysis/fix | Support clarification |
| Fixed/Resolved | Developer says fixed | Retest exact issue |
| Retest | QA verifies fix | Same steps and data if possible |
| Reopened | Issue still exists | Add new evidence |
| Closed | Verified fixed | Close after fix is confirmed |
| Rejected/Not a bug | Expected behavior claimed | Verify requirement and discuss |
| Duplicate | Already logged | Link to existing defect |
| Deferred | Valid but postponed | Record release risk |

### Strong bug report fields

- Clear title.
- Module/feature.
- Environment/build/browser.
- Preconditions.
- Test data.
- Repro steps.
- Expected result.
- Actual result.
- Evidence: screenshot, video, logs, API response, trace, DB observation.
- Severity and priority.
- Frequency.
- Business/user impact.

## 4. Manual Testing Angle

Manual QA owns reproduction quality.

- Isolate the issue before logging.
- Check whether it is data, environment, requirement, or real product issue.
- Capture exact data and environment.
- Use clear expected vs actual result.
- Mention impact, not emotion.
- Retest the exact fix first, then run impacted regression.

If developer says "not reproducible", improve evidence: build, user, data, timestamp,
screenshots/video, logs, and exact preconditions.

## 5. Automation Angle

Automation failure is a signal, not automatically a product bug.

Triage first:

- Product defect.
- Locator/test script issue.
- Wait/timing issue.
- Test data issue.
- Environment issue.
- Feature flag/config issue.
- Weak assertion.

If it is a real product defect, log it like any manual defect and attach automation
evidence such as trace/video/logs. After repeated or high-risk defects, add a regression
test if the expected result is stable.

## 6. Exact Surface To Remember

- Severity = impact.
- Priority = urgency.
- Retesting = specific fix verification.
- Regression = impact check around the fix.
- Good bug report = reproducible, specific, evidenced, impact-aware.

## 7. Real Project Example

> "When I find a defect manually, I include environment, user, data, repro steps,
> expected and actual result, and evidence. For automation failures, I first triage
> whether the failure is application, data, environment, or script related. Only real
> product failures become defects."

## 8. Common Mistakes / Weak Areas

- Logging vague titles.
- Missing expected result.
- Missing test data.
- Confusing severity and priority.
- Logging automation failure without triage.
- Closing defect without impacted regression.

## 9. 30-Second Answer

A defect usually moves from New to Open, Fixed, Retest, and Closed, with possible states
like Reopened, Rejected, Duplicate, or Deferred. A good bug report has clear repro steps,
expected and actual result, environment, data, evidence, severity, priority, and impact.

## 10. 2-Minute Answer

When I find a bug, I first make sure it is reproducible and not caused by test data,
environment, or misunderstanding. Then I log a clear report with build, browser, user,
data, preconditions, steps, expected result, actual result, evidence, severity, priority,
and impact. After a fix, I retest the exact issue first. If fixed, I run regression
around the impacted area. For automation failures, I do the same triage before calling it
a product bug.

## 11. Interview Q&A

### Q: Severity vs priority?

Severity is the impact of the defect. Priority is how urgently it should be fixed.

### Q: Developer marks bug not reproducible. What do you do?

Recheck build/data/steps, reproduce again, add stronger evidence, logs/timestamp if
available, and discuss with developer using facts.

### Q: Should every defect be automated later?

No. Automate when the risk is likely to repeat, the behavior is stable, and the expected
result is clear.

## 12. Quick Revision Points

- Defect report must reduce back-and-forth.
- Automation failure needs triage.
- Retest first, then regression.
- Severity = impact, priority = urgency.
