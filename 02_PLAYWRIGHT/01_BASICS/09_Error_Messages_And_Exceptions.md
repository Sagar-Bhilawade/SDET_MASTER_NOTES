# Error Messages And Exceptions

## 1. What It Is

Playwright errors are diagnostic signals. The goal is not memorizing every stack trace;
the goal is hearing the error family and knowing where to investigate.

## 2. Why It Matters In Interviews

Interviewers ask errors to check whether you debug systematically. Strong SDETs classify
the failure before increasing timeout or adding `force`.

## 3. Complete Topic Breakdown

| Error family | Meaning | First check |
|---|---|---|
| Strict mode violation | Locator matched multiple elements | Scope/filter locator |
| Timeout | Expected state/action did not complete | App state, data, wait type |
| Actionability | Element not ready for action | Visible/stable/enabled/receives events |
| Assertion failure | Expected result not met | Requirement, data, assertion |
| Page/context/browser closed | Test acted after cleanup/close | Missing await, teardown, popup handling |
| 401 | Not authenticated | Token/session/request mode |
| 403 | Authenticated but not authorized | Role/permission |
| Browser missing | Browsers not installed | `npx playwright install` |
| No tests found | Pattern/project mismatch | `testDir`, `testMatch`, command |
| `test.only` blocked | Focused test in CI | `forbidOnly: !!process.env.CI` (explained in `02_FRAMEWORK/05_Config_Files`) |

## 4. Manual Testing Angle

Manual QA also classifies before reporting: product issue, data issue, environment issue,
or misunderstood requirement. Playwright debugging should follow the same discipline.

## 5. Automation Angle

Do not fix errors blindly:

- strict mode -> improve locator,
- timeout -> wait for correct state,
- actionability -> inspect UI state,
- assertion -> verify expected behavior,
- 401/403 -> check auth/session/role,
- closed page/context -> check async and cleanup.

## 6. Exact Surface To Remember

Common wording:

```text
strict mode violation
Timeout 30000ms exceeded
locator.click: Target closed
element is not visible
element is outside of the viewport
subtree intercepts pointer events
browserType.launch: Executable doesn't exist
Error: No tests found
```

Useful commands:

```bash
npx playwright test --debug
npx playwright show-trace trace.zip
npx playwright install --with-deps
```

## 7. Real Project Example

> "In BASES_PLAYWRIGHT, if a test fails with strict mode, I do not add `first()` by
> default. I check why the locator is ambiguous. If it is a 401, I check whether the test
> used isolated `request`, `page.request`, or a bearer-token API fixture."

## 8. Common Mistakes / Weak Areas

- Treating every timeout as slow app.
- Adding hard wait without checking state.
- Using `force` for actionability failures.
- Adding `first()` to hide strict mode.
- Confusing 401 and 403.
- Not checking missing browser install in fresh CI.

## 9. 30-Second Answer

I classify Playwright errors first. Strict mode means locator ambiguity, timeout means
the expected action/state did not complete, actionability means the element was not ready
for user interaction, closed page/context often means async or cleanup issue, and 401/403
points to auth or permission.

## 10. 2-Minute Answer

Playwright errors are usually helpful if you read them by category. For strict mode, I
fix locator specificity. For timeouts, I check what was waiting: action, assertion, URL,
event, or API state. For actionability, trace viewer shows if an overlay, disabled state,
animation, or wrong target blocked the action. For auth failures, 401 means unauthenticated
and 403 means authenticated but not permitted.

The trade-off is not to silence errors. A strict mode failure may be exposing a real
ambiguous UI. An actionability failure may show the user also cannot click the element.

## 11. Interview Q&A

### Q: What is strict mode violation?

The locator matched more than one element for an action that expects one. Fix by scoping,
filtering, or using better locator.

### Q: What do you do when click times out?

Open trace and check visibility, stability, enabled state, overlay, viewport, locator
target, and app/data state.

### Q: Difference between 401 and 403?

401 means not authenticated. 403 means authenticated but not authorized.

### Q: Why no stale element like Selenium?

Playwright locators re-resolve elements at action/assertion time instead of keeping an
old WebElement reference.

## 12. Quick Revision Points

- Classify error family.
- Strict mode = ambiguity.
- Timeout = state did not happen.
- Actionability = user-like interaction blocked.
- 401 auth, 403 permission.
