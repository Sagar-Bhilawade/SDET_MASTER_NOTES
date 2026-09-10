# Test Data & Environment

## 1. What It Is

Test data is the input and system state needed to execute a check. Test environment is
the build, configuration, services, browser, database, roles, and integrations where
testing happens.

## 2. Why It Matters In Interviews

Many real QA failures are not because testers forgot a definition. They happen because
data is dirty, roles are wrong, services are down, or environments differ. Good SDETs
know this and plan for it.

## 3. Complete Topic Breakdown

### Test data types

- Valid data.
- Invalid data.
- Boundary data.
- Duplicate data.
- Empty/null data.
- Role-based data.
- Large-volume data.
- Old/migrated data.
- Dependent/integrated data.

### Environment checks

- Correct build/version.
- Correct URL.
- Feature flags/config.
- Browser/device support.
- User roles and permissions.
- API/service availability.
- Database connectivity.
- Test accounts not locked/expired.
- Stable data reset/cleanup approach.

## 4. Manual Testing Angle

Manual testers must prepare and protect test data.

- Know which users and records are safe to change.
- Avoid corrupting shared data.
- Capture data used in defect reports.
- Reproduce defects with the same data where possible.
- Check behavior with realistic data volume, not only perfect demo data.

## 5. Automation Angle

Automation needs predictable data.

- Prefer isolated test data where possible.
- Create data through API/setup when stable.
- Clean up after tests when needed.
- Avoid depending on data that another tester/job can change.
- Separate product bug from data/environment failure.

Common automation failure categories:

- Data missing.
- Data already used.
- Role does not have permission.
- Environment down.
- Feature flag off.
- Build mismatch.

## 6. Exact Surface To Remember

- Test data must match the scenario.
- Environment issue is not the same as product defect.
- Shared data creates flaky tests.
- Clean setup and cleanup improve automation reliability.

## 7. Real Project Example

> "For manual testing I note the exact user, role, record, and environment used. For
> automation I prefer stable or isolated data because shared data can cause false
> failures. When a test fails, I triage whether it is product behavior, test data,
> environment, or script issue."

## 8. Common Mistakes / Weak Areas

- Not recording data in bug reports.
- Testing only with perfect data.
- Using shared data for automation without cleanup.
- Logging environment issues as product defects.
- Ignoring feature flags/config.

## 9. 30-Second Answer

Test data and environment are core to reliable testing. I prepare valid, invalid,
boundary, role-based, and realistic data, and I verify build, config, services, and
permissions before execution. For automation, stable data is critical to avoid flaky
false failures.

## 10. 2-Minute Answer

Before testing, I confirm the environment and data are suitable: correct build, URL,
role, feature flag, API/service availability, and required records. Manual execution
needs realistic data and clear evidence. Automation needs even more control because
shared or changing data can make tests fail randomly. So I separate failures into
product bug, data issue, environment issue, and script issue before logging or fixing.

## 11. Interview Q&A

### Q: Why do automation tests fail due to data?

Because the expected record may be missing, already used, changed by another test, or
not valid for the user's role.

### Q: What do you include in defect evidence?

Environment, build, user, role, data used, steps, expected/actual, and screenshots/logs
or API evidence where relevant.

## 12. Quick Revision Points

- Bad data gives false defects.
- Shared data gives flaky automation.
- Always note build, environment, user, role, and record.
