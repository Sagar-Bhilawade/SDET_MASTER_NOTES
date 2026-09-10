# Security, Usability, Compatibility & Accessibility

## 1. What It Is

These are non-functional quality areas. They check whether the system is safe, usable,
accessible, and reliable across supported browsers/devices.

## 2. Why It Matters In Interviews

Manual + automation QA roles often ask what non-functional checks you can cover. The
right answer is honest: you can cover QA/SDET-level risks and involve specialists for
deep security or performance work.

## 3. Complete Topic Breakdown

| Area | What it checks |
|------|----------------|
| Security | Unauthorized access, unsafe input, data exposure |
| Usability | Ease of completing real user tasks |
| Accessibility | Usability for people with disabilities |
| Compatibility | Supported browsers, OS, devices |
| Responsive testing | Layout and behavior across screen sizes |

### Security basics

| Risk | Meaning | QA check |
|------|---------|----------|
| Authentication | Who are you? | Login/logout/session expiry |
| Authorization | What can you access? | Role permissions |
| IDOR | Access object by changing ID | Try another user's record ID |
| Input validation | Unsafe/invalid input accepted | Boundary/special characters |
| Session handling | Token/session misuse | Back after logout, expiry |
| Data exposure | Sensitive data visible | UI/API/log review where allowed |

### Usability checks

- Is the primary action clear?
- Are labels and messages understandable?
- Can user recover from mistakes?
- Are required fields clear?
- Are error messages useful?
- Does the flow work with realistic data?

### Accessibility checks

- Keyboard navigation.
- Visible focus.
- Proper labels.
- Error messages near fields.
- Color contrast.
- Meaningful accessible names.

## 4. Manual Testing Angle

Manual testing is essential here.

- Security: role checks, session behavior, obvious unauthorized access attempts.
- Usability: human judgement about clarity, effort, confusion, and recovery.
- Accessibility: keyboard flow, focus, labels, contrast observation.
- Compatibility: actual browser/device behavior and layout.
- Responsive: menus, tables, modals, filters, and forms at different sizes.

## 5. Automation Angle

Automation can support these areas, but cannot fully replace judgement.

- Automate stable role-based access checks.
- Automate critical cross-browser smoke flows.
- Add API negative checks for forbidden/unauthorized access.
- Use accessibility scans where available, but treat them as partial coverage.
- Use viewport-based checks for critical responsive flows.

Do not overclaim deep penetration testing. Say:

> "I cover security from a QA/SDET angle: auth/authz, role access, input validation,
> session behavior, and regression for sensitive flows. For deep penetration testing,
> I would involve security specialists."

## 6. Exact Surface To Remember

- Authentication = identity.
- Authorization = permission.
- IDOR = changing object ID should not expose another user's data.
- Accessibility automation catches some issues, not all user experience.
- Usability is mostly human judgement.
- Compatibility should follow supported browser matrix.

## 7. Real Project Example

> "For manual testing, I validate role access, error messages, keyboard usability,
> supported browser behavior, and responsive layout for important flows. For automation,
> I would keep stable permission checks and cross-browser smoke flows. I would not claim
> automation fully proves usability or accessibility."

## 8. Common Mistakes / Weak Areas

- Confusing authentication and authorization.
- Overclaiming security expertise.
- Saying accessibility can be fully automated.
- Testing only Chrome if product supports more browsers.
- Treating usability as cosmetic only.
- Ignoring responsive tables, filters, and modals.

## 9. 30-Second Answer

Security checks whether access and data are protected, usability checks whether users can
complete work easily, accessibility checks whether the product works for users with
different needs, and compatibility checks supported browsers/devices. I cover these from
a QA/SDET angle and automate stable checks where possible.

## 10. 2-Minute Answer

For security, I focus on auth/authz, role access, session handling, input validation, and
data exposure risks. For usability, I check whether the flow is understandable and users
can recover from mistakes. For accessibility, I check keyboard navigation, focus, labels,
contrast, and error messages. For compatibility, I follow the supported browser/device
matrix and prioritize critical flows. Automation can help with stable permission checks,
API negative checks, accessibility scans, and cross-browser smoke, but human judgement is
still required.

## 11. Interview Q&A

### Q: Authentication vs authorization?

Authentication verifies identity. Authorization verifies permission.

### Q: What is IDOR?

Insecure Direct Object Reference: changing an object ID should not allow access to
another user's data.

### Q: Can accessibility be fully automated?

No. Automation catches some rule violations, but keyboard flow, clarity, and real user
experience still need manual judgement.

## 12. Quick Revision Points

- Auth = identity, authz = permission.
- IDOR = object ID access risk.
- Usability/accessibility need human judgement.
- Automate support checks, not full judgement.
