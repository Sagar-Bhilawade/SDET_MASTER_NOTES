# Test Scenarios, Test Cases & Checklists

## 1. What It Is

A test scenario is a high-level condition to validate. A test case is a detailed,
executable check with preconditions, steps, data, and expected result. A checklist is a
lighter coverage aid used when detailed cases are too heavy or the work is exploratory.

## 2. Why It Matters In Interviews

Manual + automation roles expect you to design coverage, not only execute scripts.
Interviewers may ask how you write test cases, what fields you include, and how you
avoid over-documentation in Agile.

## 3. Complete Topic Breakdown

### Test scenario

Example:

- Verify user can create a report with valid filters.
- Verify unauthorized user cannot access admin report.
- Verify export respects applied filters.

### Test case

Good fields:

- ID/title.
- Requirement/story link.
- Preconditions.
- Test data.
- Steps.
- Expected result.
- Actual result.
- Status.
- Priority/risk.
- Evidence/notes.

### Checklist

Useful for:

- Exploratory testing.
- UI consistency checks.
- Regression around a small change.
- Release smoke.
- Areas where detailed steps are obvious to experienced QA.

## 4. Manual Testing Angle

Manual cases should cover:

- Happy path.
- Negative path.
- Boundary values.
- Required/optional fields.
- Role and permission.
- Error messages.
- Data persistence.
- Integration impact.
- Browser/responsive behavior when needed.

The case should be detailed enough that another QA can execute it without guessing, but
not so heavy that Agile delivery becomes documentation work only.

## 5. Automation Angle

Automation starts from good manual coverage but should not blindly copy every manual
case.

- Automate stable, repeated, high-value cases.
- Convert repeated data variations into data-driven tests.
- Use API/component level for rule-heavy cases.
- Keep exploratory and judgement-based checks manual.
- Add assertions based on expected result, not only visibility.

## 6. Exact Surface To Remember

- Scenario = high-level what.
- Test case = detailed how.
- Checklist = lightweight coverage reminder.
- Expected result must be observable.
- A case without test data is often incomplete.

## 7. Real Project Example

> "For a new story, I first prepare scenarios from AC and risk. For critical flows I
> write detailed cases with data and expected results. After the behavior stabilizes,
> I select repeated cases for automation and avoid converting every manual checklist
> item into a UI script."

## 8. Common Mistakes / Weak Areas

- Writing steps without expected result.
- Missing test data.
- Writing too many low-value cases.
- Using checklists where traceability is required.
- Automating every manual case blindly.

## 9. 30-Second Answer

Scenarios define what to test, test cases define exactly how to test, and checklists
support lightweight coverage. I use scenarios for coverage thinking, cases for important
repeatable checks, and checklists for exploratory or low-risk areas.

## 10. 2-Minute Answer

I start from acceptance criteria and risk. First I list scenarios: positive, negative,
boundary, role, data, and integration. Then for important scenarios I write test cases
with preconditions, data, steps, expected result, status, and evidence. In Agile, I keep
documentation useful rather than heavy. For automation, I select stable repeated cases,
often pushing validations to API level and keeping only critical journeys at UI level.

## 11. Interview Q&A

### Q: What fields should a test case have?

ID, title, requirement link, preconditions, data, steps, expected result, actual result,
status, priority, and evidence/notes.

### Q: Test case vs checklist?

Test case is detailed and traceable. Checklist is lighter and useful for exploratory or
obvious repeated checks.

## 12. Quick Revision Points

- Scenario = coverage.
- Case = execution.
- Checklist = lightweight reminder.
- Expected result and data are mandatory for strong cases.
