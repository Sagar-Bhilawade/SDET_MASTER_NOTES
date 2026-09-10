# White-Box, Black-Box & Grey-Box Testing

## 1. What It Is

Black-box testing checks behavior from inputs and outputs without internal code
knowledge. White-box testing uses internal code/logic knowledge. Grey-box testing uses
some internal knowledge, such as APIs, DB, logs, configs, and architecture.

## 2. Why It Matters In Interviews

SDET interviews expect you to understand more than UI behavior. You do not need to claim
developer ownership of all unit tests, but you should understand coverage, code paths,
and how internal knowledge improves testing.

## 3. Complete Topic Breakdown

| Approach | Knowledge used | Example |
|----------|----------------|---------|
| Black-box | Requirement, inputs, outputs | Validate login error for invalid password |
| White-box | Code structure and branches | Unit test if/else calculation paths |
| Grey-box | Partial internals | Use API + DB + logs to validate UI behavior |

### Coverage types

| Coverage | Meaning | Limitation |
|----------|---------|------------|
| Statement | Each statement executed | Can miss false branch |
| Branch/decision | Each decision outcome covered | Stronger than statement |
| Condition | Each boolean condition true/false | Useful for compound logic |
| Path | Different execution paths | Can explode quickly |
| Loop | Zero, one, many iterations | Catches boundary/empty-list issues |

## 4. Manual Testing Angle

Manual QA mostly works black-box, but strong QA often works grey-box.

- Check UI behavior against requirements.
- Use API responses to understand actual backend behavior.
- Check DB state when relevant and allowed.
- Use logs or network data to improve defect reports.
- Think about branches: valid, invalid, empty, duplicate, permission, timeout.

Grey-box thinking improves manual testing because it helps isolate where the issue may
be: UI, API, DB, config, data, or integration.

## 5. Automation Angle

Automation level selection is tied to this topic.

- Unit tests are good for small logic branches.
- API/component tests are good for business rules.
- UI tests are good for end-user confidence.
- Do not put all branch combinations in E2E.
- Code coverage is a signal, not proof of quality.

Important line:

> 100% statement coverage only says code was executed. It does not prove the right
> assertions were made.

## 6. Exact Surface To Remember

- Black-box = behavior without internals.
- White-box = internal code logic.
- Grey-box = practical SDET testing with partial internals.
- Statement coverage can miss branches.
- Branch coverage checks decision outcomes.
- Coverage percentage is not the same as quality.

## 7. Real Project Example

> "In day-to-day QA/SDET work, I mostly validate behavior like black-box testing, but I
> also use grey-box signals such as API responses, DB state, logs, and configs. That helps
> me isolate defects better and decide whether a check belongs in UI automation, API
> automation, or should stay manual."

## 8. Common Mistakes / Weak Areas

- Saying 100% coverage means no bugs.
- Not knowing statement vs branch coverage.
- Thinking QA never uses internal knowledge.
- Using UI automation for every logic branch.
- Confusing code coverage with requirement coverage.

## 9. 30-Second Answer

Black-box testing validates behavior from outside. White-box testing uses internal code
knowledge. Grey-box testing uses partial internal knowledge like APIs, DB, logs, and
configs. As an SDET, grey-box thinking helps me isolate defects and choose the right test
level.

## 10. 2-Minute Answer

In real projects, QA does not stay purely black-box all the time. I validate behavior
from the user side, but I also use API responses, DB state, logs, and environment config
to understand failures. White-box concepts like statement and branch coverage help me
understand what unit/component tests should cover. But I do not treat coverage percentage
as quality proof. A line can execute without a meaningful assertion. The practical
decision is to test each risk at the cheapest reliable level.

## 11. Interview Q&A

### Q: Statement vs branch coverage?

Statement coverage means lines executed. Branch coverage means each decision outcome,
like true and false, was exercised.

### Q: Is 100% coverage enough?

No. It may still have weak assertions, missing requirements, and untested data paths.

### Q: How does grey-box help QA?

It helps isolate whether failure is UI, API, DB, data, config, or integration related.

## 12. Quick Revision Points

- Black-box = outside behavior.
- White-box = code internals.
- Grey-box = practical SDET reality.
- Coverage is useful, but not proof.
