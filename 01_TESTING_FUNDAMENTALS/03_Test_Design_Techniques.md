# Test Design Techniques

## 1. What It Is

Test design techniques help decide **which test cases are worth writing**. They prevent
random testing and give maximum meaningful coverage with minimum duplicate cases.

## 2. Why It Matters In Interviews

Because ISTQB is on the resume, EP, BVA, decision tables, and state transition testing
are fair questions. At 3.8+ years, the answer must include examples and automation
conversion, not only definitions.

## 3. Complete Topic Breakdown

| Technique | Use when | Main idea |
|-----------|----------|-----------|
| Equivalence partitioning | Inputs behave in groups | Test one representative from each group |
| Boundary value analysis | Numeric/date/length/range fields | Test below, on, and above boundaries |
| Decision table | Conditions combine | Cover combinations and expose missing rules |
| State transition | Status changes over time | Test valid and invalid transitions |
| Use-case testing | Business user journeys | Cover end-to-end user goal |
| Pairwise testing | Many configuration combinations | Reduce combinations intelligently |
| Error guessing | Known failure patterns | Add experience-based cases |
| Exploratory testing | New or unclear behavior | Investigate with a charter |

### Example: age field accepts 18 to 65

Partitions:

- Below range: 17
- Valid range: 30
- Above range: 66
- Blank
- Non-numeric
- Decimal
- Negative

Boundaries:

- 17, 18, 19
- 64, 65, 66

Exact line:

> Partitioning gives the groups; boundary analysis chooses the most risky values at
> the edges of those groups.

Important interview point:

BVA does not replace EP. EP first tells us **what behavior groups exist**. BVA then
chooses stronger values near the edges of those groups. If we jump directly to boundary
values without partitioning, we may miss non-boundary classes like blank input,
non-numeric input, decimal value, negative value, special characters, permission-based
variation, or a business category that has no numeric edge.

So the practical order is:

1. Use EP to identify all behavior groups.
2. Use BVA only for partitions that have meaningful boundaries.
3. Add one normal middle value when it gives confidence that the full valid partition
   works, not only the edge.

### Example: state transition

Order flow:

- Created -> Paid -> Packed -> Shipped -> Delivered
- Created -> Cancelled
- Paid -> Cancelled if business allows
- Delivered -> Cancelled should be blocked
- Refunded -> Paid should be blocked

Invalid transitions matter because developers often implement happy paths carefully and
miss blocked paths.

## 4. Manual Testing Angle

Manual test design turns requirements into coverage.

- Read story and acceptance criteria.
- Identify input fields, business rules, roles, statuses, and integrations.
- Derive scenarios first, then detailed cases.
- Include positive, negative, boundary, permission, data, and error handling cases.
- Use exploratory charters when behavior is new or uncertain.
- Use decision tables to ask requirement questions before code is finished.

Manual test design is also communication: if a decision table has an empty cell, that is
not just a testing gap; it is a requirement gap.

## 5. Automation Angle

Automation should come after good test design.

- Convert stable partitions and boundaries into data-driven tests.
- Prefer API/component level for pure validation rules.
- Use UI automation for critical business journeys, not every combination.
- Automate decision table combinations only when rules and expected results are stable.
- Add regression for repeated real defects, but do not automate every one-off bug.

## 6. Exact Surface To Remember

- EP = divide inputs into behavior groups.
- BVA = test below/on/above boundaries.
- Decision table = conditions vs expected outcomes.
- State transition = valid and invalid movement between states.
- Pairwise = reduce configuration combinations.
- Error guessing = experience-based complement, not replacement.
- Good test design = maximum coverage with minimum meaningful cases.

## 7. Real Project Example

> "For manual testing, I use test design techniques to create focused coverage instead
> of random cases. For automation, I then choose the cheapest reliable level. For example,
> validation rules derived from EP/BVA are better as API or data-driven checks, while
> only critical business journeys should become UI E2E tests. That keeps the suite useful
> without making it slow and fragile."

## 8. Common Mistakes / Weak Areas

- Giving definitions without examples.
- Testing only valid partitions.
- Forgetting invalid state transitions.
- Automating all combinations through UI.
- Calling exploratory testing random testing.
- Writing too many cases that do not add new coverage.

## 9. 30-Second Answer

I use test design techniques to select meaningful cases. EP groups similar inputs, BVA
tests the risky edges, decision tables handle combined conditions, and state transition
testing checks valid and invalid status movement. These techniques help manual testing
coverage and also help decide which cases are worth automating.

## 10. 2-Minute Answer

Good test design starts from the requirement and risk. For input rules, I use
equivalence partitioning to avoid duplicate cases and boundary value analysis because
many bugs happen at edges. For combined business rules, I build a decision table because
it exposes missing requirement combinations. For workflows, I use state transition
testing and include invalid transitions, not only happy paths.

After deriving coverage, I decide execution level. Pure validation rules should not all
be UI tests. They can usually be API or data-driven checks. UI automation should focus on
critical user journeys. This keeps both manual and automation coverage strong.

## 11. Interview Q&A

### Q: How do you decide test cases?

By requirement, risk, changed area, business criticality, and test design techniques:
EP, BVA, decision tables, state transitions, exploratory testing, and past defects.

### Q: Why BVA if EP exists?

EP gives behavior groups. BVA focuses on the edges of those groups where off-by-one
defects commonly happen.

### Q: If BVA already covers boundary values and one value around each side, why do we still need equivalence partitioning?

Because BVA is not a separate replacement for EP; it is usually applied **after** EP.
EP identifies the classes of input that should behave similarly. BVA then selects the
highest-risk values from the numeric or ordered classes.

For example, if age accepts 18 to 65, BVA gives 17, 18, 19, 64, 65, 66. That is strong
for range defects. But EP also reminds us to check other classes such as blank,
non-numeric, decimal, negative, and maybe extremely large values. Those are not
boundary cases of the valid range, but they are still different behavior groups.

In interviews I would answer it like this:

> "I don't treat BVA and EP as competing techniques. EP tells me the partitions; BVA
> sharpens the risky edges inside those partitions. If BVA already gives enough
> confidence for a simple numeric range, I may not add many extra middle values. But I
> still use EP thinking to make sure I have not missed invalid or special input classes."

Trade-off: adding both blindly can create duplicate cases. So I combine them: one
normal representative for the valid class if needed, boundary values for edge risk,
and separate representatives for invalid classes.

### Q: When use decision table?

When multiple conditions combine, such as role, status, amount, coupon, or permission.

### Q: What should be automated from these?

Stable, repeated cases with clear expected results. Prefer API/component for logic and
UI only for critical user journeys.

## 12. Quick Revision Points

- EP = groups.
- BVA = edges.
- Decision table = combinations.
- State transition = valid + invalid status movement.
- Manual derives coverage; automation preserves stable repeated checks.
