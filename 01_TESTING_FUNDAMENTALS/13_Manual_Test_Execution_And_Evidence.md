# Manual Test Execution & Evidence

## 1. What It Is

Manual test execution is running planned and exploratory checks using real judgement.
Evidence is the proof that shows what was tested, what happened, and why a result is
trusted.

## 2. Why It Matters In Interviews

Manual + automation roles still require strong execution discipline. Good evidence
reduces defect back-and-forth and supports release decisions.

## 3. Complete Topic Breakdown

Execution includes:

- Confirm build/environment.
- Confirm test data and role.
- Execute steps.
- Compare expected vs actual.
- Capture evidence.
- Log defect or mark pass/fail/blocked.
- Retest and regression after fix.

Evidence can include:

- Screenshot.
- Video.
- Console/network log.
- API response.
- DB query result.
- Downloaded file.
- Timestamp.
- Build/version.
- User/role/data.

## 4. Manual Testing Angle

During manual execution:

- Do not just follow steps blindly; observe side effects.
- If actual result differs, isolate before logging.
- If blocked, record the blocker clearly.
- For intermittent bugs, capture frequency and conditions.
- For passed tests, evidence may be lighter but critical flows should still have proof.
- Keep notes on unexpected observations even if they are not defects yet.

## 5. Automation Angle

Manual evidence habits improve automation:

- Automation reports should show build, environment, failed step, screenshot/video/trace.
- Assertions should prove expected result.
- Logs help separate product issue from script/data/environment issue.
- Manual repro is useful when automation finds a real issue.

## 6. Exact Surface To Remember

- Pass = expected result met.
- Fail = expected result not met.
- Blocked = cannot execute due to dependency issue.
- Evidence should make the issue reproducible.
- Intermittent defect needs timestamp, data, user, build, and frequency.

## 7. Real Project Example

> "For manual execution, I capture user, role, environment, data, steps, expected/actual,
> and evidence. For automation, I rely on reports, screenshots/videos/traces, and logs to
> triage failures. Good evidence keeps communication factual."

## 8. Common Mistakes / Weak Areas

- Marking fail without evidence.
- Marking blocked without reason.
- Not noting test data.
- Ignoring intermittent patterns.
- Saying "not working" instead of exact actual result.

## 9. 30-Second Answer

Manual execution means validating expected behavior with real data and judgement, then
recording pass, fail, or blocked with useful evidence. Evidence includes build,
environment, user, data, steps, expected/actual, screenshot/video/logs, and impact.

## 10. 2-Minute Answer

Before execution, I confirm the build, environment, role, and test data. While executing,
I compare actual behavior with expected result and observe side effects. If something
fails, I isolate whether it is product, data, environment, or requirement issue. Then I
capture evidence that makes the problem reproducible. This habit also helps automation
because automation reports should provide enough evidence to triage failures quickly.

## 11. Interview Q&A

### Q: What evidence do you attach for a defect?

Screenshot/video, build, environment, user, role, data, steps, expected/actual, logs/API
response where relevant, and impact.

### Q: What does blocked mean?

The test could not be executed because of a dependency issue like environment down, data
missing, access issue, or build unavailable.

## 12. Quick Revision Points

- Execution = steps + judgement.
- Evidence reduces back-and-forth.
- Pass/fail/blocked must be clear.
