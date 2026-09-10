# Testing Principles & Terminology

## 1. What It Is

Testing terminology gives exact language for explaining quality work. Testing principles
explain the mindset behind good testing: reduce risk, test early, prioritize intelligently,
and never claim perfection.

## 2. Why It Matters In Interviews

These questions reveal whether you think like a QA engineer or only like a script
executor. Interviewers expect ISTQB-level clarity because ISTQB CTFL is on the resume.

Strong signal:

- You distinguish QA, QC, and testing.
- You know error, defect, and failure.
- You explain severity vs priority clearly.
- You never say testing proves software is bug-free.
- You connect principles to manual and automation decisions.

## 3. Complete Topic Breakdown

### Core terms

| Term         | Meaning                          | Example                                |
| ------------ | -------------------------------- | -------------------------------------- |
| Error        | Human mistake                    | Developer misunderstands discount rule |
| Defect/Bug   | Fault in work product            | Wrong discount logic in code           |
| Failure      | Visible incorrect behavior       | User gets wrong discount               |
| Verification | Are we building it right?        | Review story, design, test cases       |
| Validation   | Are we building the right thing? | Feature solves user/business need      |
| QA           | Process to prevent defects       | Better reviews, standards, testability |
| QC/Testing   | Product checks to find defects   | Execute tests, log defects             |
| Severity     | Impact of the defect             | How bad is it?                         |
| Priority     | Urgency to fix                   | How soon should it be fixed?           |
| Test basis   | Source of expected behavior      | Story, AC, API contract, design        |
| Test oracle  | How pass/fail is known           | Requirement, DB value, API response    |
| Coverage     | What risk is exercised           | Requirement/risk to checks mapping     |

### Seven testing principles

| Principle                         | Practical meaning                                   |
| --------------------------------- | --------------------------------------------------- |
| Testing shows presence of defects | Do not claim bug-free software                      |
| Exhaustive testing is impossible  | Use risk, priority, and techniques                  |
| Early testing saves cost          | Review requirements before coding finishes          |
| Defect clustering                 | Focus on risky/changing/problem modules             |
| Pesticide paradox                 | Refresh suites or they stop finding new defects     |
| Testing is context dependent      | Testing depth depends on product risk               |
| Absence-of-errors fallacy         | Passing tests is useless if wrong product was built |

## 4. Manual Testing Angle

Manual QA uses these principles before automation even begins.

- During requirement review, identify ambiguity, missing validation, role gaps, and untestable acceptance criteria.
- During test design, choose cases using risk and coverage, not random intuition.
- During execution, log defects with severity, priority, evidence, and impact.
- During release discussion, speak in terms of risk reduced, risk remaining, and confidence level.

Manual QA value is not only finding bugs. It is preventing defects through questions,
clarity, coverage, and risk judgement.

## 5. Automation Angle

Automation applies the same principles at scale.

- Exhaustive testing is impossible, so automation suite selection must be risk-based.
- Defect clustering tells us which modules deserve stronger regression coverage.
- Pesticide paradox means old suites must evolve with new defects and new features.
- Absence-of-errors fallacy means a green pipeline is not enough if assertions are weak.
- Testing is context dependent, so not every project needs the same UI/API/DB split.

## 6. Exact Surface To Remember

- QA = prevention/process.
- Testing/QC = detection/product check.
- Error = human mistake.
- Defect = fault in work product.
- Failure = visible wrong behavior.
- Severity = impact.
- Priority = urgency.
- Verification = building it right.
- Validation = building the right thing.

## 7. Real Project Example

> "In my work I use these principles while handling both manual and automation testing.
> For manual work, I review stories and acceptance criteria early so requirement gaps
> are caught before execution. For automation, I do not treat a green suite as final
> quality proof. I check whether the suite covers the right risks, whether assertions
> are meaningful, and whether new defects should update regression coverage."

## 8. Common Mistakes / Weak Areas

- Saying QA and testing are the same.
- Saying testing proves there are no bugs.
- Mixing error, defect, and failure.
- Mixing severity and priority.
- Treating automation coverage as complete product coverage.
- Forgetting that context changes testing depth.

## 9. 30-Second Answer

Testing is risk reduction, not proof that software is perfect. QA prevents defects
through process and early clarification, while testing finds defects in the product.
Important terms are error, defect, failure, verification, validation, severity, and
priority. The main principles I apply are risk-based selection, early testing, defect
clustering, and refreshing suites to avoid pesticide paradox.

## 10. 2-Minute Answer

I use testing principles to decide where effort gives the most value. Exhaustive testing
is impossible, so I prioritize based on business risk, changed areas, defect-prone
modules, and critical user flows. Early testing means I do not wait for a build; I review
requirements and acceptance criteria first. Defect clustering tells me where to go deeper.
Pesticide paradox reminds me that automation suites must be updated, because the same
checks eventually stop finding new problems.

For terminology, I keep the wording clean: error is the human mistake, defect is the
fault in the code or requirement, and failure is the visible wrong behavior. Severity is
impact; priority is urgency. Verification asks whether we built it right; validation asks
whether we built the right thing.

## 11. Interview Q&A

### Q: QA vs testing?

QA is prevention-focused: improving the process so defects are less likely. Testing is
detection-focused: executing checks to find defects and provide release information.

### Q: Error vs defect vs failure?

Error is a human mistake, defect is the fault introduced into the product, and failure is
the visible incorrect behavior when that defect is executed.

### Q: Severity vs priority?

Severity is how bad the impact is. Priority is how urgently it should be fixed.

### Q: What is pesticide paradox?

If the same tests are repeated again and again, they find fewer new defects. Suites must
be reviewed and refreshed with new risks and real defects.

## 12. Quick Revision Points

- Testing reduces risk; it does not prove perfection.
- QA prevents; testing detects.
- Severity = impact, priority = urgency.
- Error -> defect -> failure.
- Green automation is useful only when coverage and assertions are meaningful.
