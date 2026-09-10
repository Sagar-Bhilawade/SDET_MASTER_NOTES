# STLC & Test Artifacts

## 1. What It Is

STLC is the testing lifecycle: how QA moves from requirement understanding to planning,
design, execution, defect handling, regression, and closure. Test artifacts are the
work products that make this traceable and repeatable.

## 2. Why It Matters In Interviews

Interviewers ask STLC to check whether you understand QA work beyond writing automation
scripts. A strong answer shows you can handle manual testing, automation planning,
defect communication, and release confidence.

## 3. Complete Topic Breakdown

| Phase                | Purpose                             | Output                          |
| -------------------- | ----------------------------------- | ------------------------------- |
| Requirement analysis | Understand scope, AC, risk, gaps    | Questions, scenarios, risk list |
| Test planning        | Decide scope, approach, environment | Test plan/sprint test approach  |
| Test design          | Convert requirements into checks    | Scenarios, cases, data, RTM     |
| Environment setup    | Prepare build, data, access         | Ready environment and accounts  |
| Execution            | Run manual/automation checks        | Results, evidence, defects      |
| Defect retest        | Verify fixes                        | Closed/reopened defects         |
| Regression           | Check impact area                   | Regression result               |
| Closure              | Summarize release confidence        | Test summary/sign-off           |

### Key artifacts

| Artifact          | Meaning                                               |
| ----------------- | ----------------------------------------------------- |
| Test scenario     | High-level condition to validate                      |
| Test case         | Steps, data, expected result, preconditions           |
| Test data         | Users, roles, records, boundary values                |
| RTM               | Requirement-to-test mapping                           |
| Defect report     | Repro, expected/actual, evidence, impact              |
| Test summary      | Passed/failed/blocked, risks, recommendation          |
| Automation report | Suite result, failed specs, environment/build details |

## 4. Manual Testing Angle

Manual QA work in STLC includes:

- Asking requirement questions early.
- Identifying testable and untestable acceptance criteria.
- Creating scenarios and cases with clear expected results.
- Preparing data and environment access.
- Executing manual, exploratory, regression, and retesting.
- Logging defects with evidence.
- Giving release risk summary.

In Agile, artifacts can be lightweight, but they cannot be absent. Enough documentation
must exist so coverage, defects, and release risk are clear.

## 5. Automation Angle

Automation is planned inside STLC, not after it.

- Requirement analysis: identify automation candidates and impacted suites.
- Test design: choose UI/API/component level.
- Environment setup: prepare accounts, data, config, CI.
- Execution: run smoke/regression suites.
- Defect triage: separate product bug from script/data/environment issue.
- Closure: include automation results and known unstable/blocked checks.

## 6. Exact Surface To Remember

- Test scenario = what to test.
- Test case = how exactly to test it.
- Test plan = scope, approach, resources, schedule, risks.
- Test strategy = broader approach across project/org.
- RTM = mapping requirements to tests.
- Entry criteria = when testing can start.
- Exit criteria = when testing is good enough to stop/recommend release.

## 7. Real Project Example

> "In sprint work, I start with requirement analysis and manual scenario design. Once
> behavior is clear, I decide which checks belong in automation. Smoke and regression
> are automated where stable, while exploratory and new-feature validation stay manual
> first. At closure, I would speak in terms of what passed, what failed, what was blocked,
> and what risk remains."

## 8. Common Mistakes / Weak Areas

- Reciting waterfall STLC as if Agile has no structure.
- Confusing scenario and test case.
- Saying RTM is always heavy documentation.
- Forgetting test data and environment setup.
- Treating automation as separate from STLC.
- Giving sign-off without mentioning risk.

## 9. 30-Second Answer

STLC covers requirement analysis, planning, test design, environment setup, execution,
defect retesting, regression, and closure. In Agile it runs inside each sprint. I use it
for both manual and automation work: manual scenarios and exploratory checks first, then
stable repeated checks move into automation.

## 10. 2-Minute Answer

For me STLC starts before a build is ready. I review the story, acceptance criteria,
roles, data, and integrations. Then I create scenarios and test cases, prepare data and
environment access, execute manual and exploratory checks, log defects, retest fixes,
run regression, and summarize release risk.

Automation fits into the same lifecycle. During planning, I identify automation
candidates and impacted suites. During design, I decide whether the check belongs at UI,
API, or component level. During execution, automation gives repeatable regression
confidence, but manual testing still handles new behavior and judgement-based checks.

## 11. Interview Q&A

### Q: Walk me through STLC.

Requirement analysis -> planning -> design -> environment setup -> execution -> defect
retest -> regression -> closure.

### Q: Scenario vs test case?

Scenario is high-level what to test. Test case is detailed steps, data, expected result,
and preconditions.

### Q: Test plan vs test strategy?

Strategy is high-level long-term approach. Plan is specific to project/release/sprint.

### Q: What is RTM?

Requirement Traceability Matrix maps requirements to test cases so gaps are visible.

## 12. Quick Revision Points

- STLC exists in Agile too.
- Scenario = what; case = how.
- Manual and automation both sit inside STLC.
- Closure means risk summary, not only "all passed".
