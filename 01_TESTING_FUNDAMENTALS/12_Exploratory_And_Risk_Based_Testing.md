# Exploratory & Risk-Based Testing

## 1. What It Is

Exploratory testing is structured investigation where learning, test design, and
execution happen together. Risk-based testing means prioritizing effort based on impact
and likelihood of failure.

## 2. Why It Matters In Interviews

This is where manual QA maturity shows. Many candidates say "manual testing" but only
describe executing test cases. Strong QA can explore, prioritize, and explain risk.

## 3. Complete Topic Breakdown

### Exploratory testing

It is not random clicking. It uses:

- Charter.
- Time box.
- Focus area.
- Notes.
- Observations.
- Defects.
- Follow-up questions.

Example charter:

> Explore report filter behavior for invalid, empty, large, and role-restricted data for
> 45 minutes. Focus on incorrect results, confusing messages, and export mismatch.

### Risk-based testing

Prioritize using:

- Business criticality.
- Customer impact.
- Changed code area.
- Defect history.
- Complexity.
- Integration dependency.
- User frequency.
- Regulatory/security sensitivity.
- Time available.

## 4. Manual Testing Angle

Manual testing is strongest for risk discovery.

- Explore new or changed features.
- Follow unexpected behavior.
- Compare UI, API, DB, and reports when needed.
- Find requirement gaps.
- Look for side effects.
- Capture notes and convert findings into defects, questions, or regression candidates.

## 5. Automation Angle

Automation benefits from exploratory testing.

- Exploratory findings identify future regression candidates.
- Risk-based testing decides which tests enter smoke/regression suites.
- Automation should focus on high-risk repeated behavior.
- Do not automate uncertain exploratory paths until expected behavior is clear.

## 6. Exact Surface To Remember

- Exploratory = charter-based learning and testing.
- Risk = impact x likelihood.
- Risk-based testing = test most important things first.
- Time-box keeps exploration focused.

## 7. Real Project Example

> "When time is limited, I do not try to test everything equally. I start with changed
> areas, critical flows, defect-prone modules, and integrations. I use exploratory
> charters for new behavior, then convert stable repeated findings into regression or
> automation candidates."

## 8. Common Mistakes / Weak Areas

- Calling exploratory testing random.
- Exploring without notes or charter.
- Testing low-risk cosmetic areas before critical flows.
- Ignoring past defects.
- Automating exploratory checks before expected result is agreed.

## 9. 30-Second Answer

Exploratory testing is structured investigation with a charter, time box, notes, and
follow-up. Risk-based testing prioritizes based on business impact and failure likelihood.
Together they help manual QA find unknown risks and help automation focus on the most
valuable regression checks.

## 10. 2-Minute Answer

When a feature is new, scripted cases are not enough because they only cover what we
already predicted. I use exploratory testing with a charter, such as checking filter
behavior with invalid, large, empty, and role-restricted data. I take notes, capture
defects, and identify questions. Then I use risk-based thinking to decide what matters
first: changed area, business-critical flow, defect history, integrations, and customer
impact. Stable repeated risks can later become automation.

## 11. Interview Q&A

### Q: Is exploratory testing random testing?

No. It is structured investigation with a charter, time box, notes, and learning.

### Q: How do you test when time is short?

Prioritize changed area, critical business flows, integrations, defect-prone modules,
and high-impact negative cases.

## 12. Quick Revision Points

- Exploratory = structured, not random.
- Risk = impact x likelihood.
- Automation gets candidates from repeated exploratory findings.
