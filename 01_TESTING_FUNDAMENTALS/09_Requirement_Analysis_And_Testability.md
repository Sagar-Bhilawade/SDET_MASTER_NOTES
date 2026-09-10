# Requirement Analysis & Testability

## 1. What It Is

Requirement analysis is the QA activity of understanding what needs to be built, finding
ambiguity, and deciding how the feature can be tested. Testability means the requirement
is clear enough to verify with data, steps, and expected results.

## 2. Why It Matters In Interviews

This is one of the strongest manual + automation selling points. It shows you are not
waiting for a completed build. You are improving quality before coding finishes.

## 3. Complete Topic Breakdown

Good requirement analysis checks:

- Business goal.
- User role and permission.
- Acceptance criteria.
- Positive and negative behavior.
- Validation rules.
- Error messages.
- Data requirements.
- Integrations.
- Audit/log/reporting impact.
- Out-of-scope items.
- Backward compatibility.
- Test environment needs.

Untestable requirement examples:

- "System should be fast."
- "User should get proper error."
- "Export should work correctly."

Better wording:

- "Report should load within agreed SLA for 10,000 records in test-like data."
- "Invalid email should show 'Enter a valid email address' under Email field."
- "Export should include columns A, B, C and preserve applied filters."

## 4. Manual Testing Angle

Manual QA owns the first quality gate:

- Ask what happens for blank, invalid, duplicate, expired, unauthorized, and boundary data.
- Ask what should happen when dependent service/API fails.
- Ask whether old records and migrated data are affected.
- Ask whether the behavior differs by role, region, browser, or feature flag.
- Ask what evidence proves success: UI message, DB state, API response, downloaded file.

## 5. Automation Angle

Good requirement analysis makes automation cheaper.

- Clear acceptance criteria become clear assertions.
- Stable fields and roles become reusable test data.
- API contracts become API automation candidates.
- Predictable error messages become assertable checks.
- Ambiguous requirements create weak automation and false confidence.

## 6. Exact Surface To Remember

- Acceptance criteria should be specific, testable, and observable.
- Requirement gap = missing expected behavior.
- Ambiguity = multiple valid interpretations.
- Testability question = "How will we prove this passed?"

## 7. Real Project Example

> "Before testing a story, I clarify acceptance criteria, role behavior, validation,
> data, and impacted areas. This helps manual testing because scenarios become clear,
> and it helps automation because assertions and test data are stable."

## 8. Common Mistakes / Weak Areas

- Waiting for build before asking questions.
- Writing test cases from vague requirements.
- Automating unclear behavior.
- Missing role, data, and integration impact.
- Not asking what should happen for negative cases.

## 9. 30-Second Answer

Requirement analysis is where QA checks whether the story is clear, complete, and
testable. I look for gaps in acceptance criteria, data, roles, validations, integrations,
and expected results. This prevents defects early and also makes automation stronger.

## 10. 2-Minute Answer

I do not treat requirement analysis as a formality. I read the story and acceptance
criteria, then ask what can fail: invalid data, missing permissions, duplicate records,
API failure, old data, cross-browser behavior, and error handling. I also ask how we will
prove the feature works: UI state, API response, DB value, report content, or audit log.

If this is clear, manual scenarios become stronger and automation has clean assertions.
If it is not clear, automation becomes a script that clicks things without proving the
right behavior.

## 11. Interview Q&A

### Q: What do you do if requirements are unclear?

I raise specific questions with examples, document the agreed behavior, and avoid
automating until expected results are clear.

### Q: What makes a requirement testable?

Clear input, action, expected result, data, role, environment, and observable pass/fail
condition.

## 12. Quick Revision Points

- Ask before build.
- Every story needs observable expected results.
- Clear requirement -> strong manual case -> strong automation assertion.
