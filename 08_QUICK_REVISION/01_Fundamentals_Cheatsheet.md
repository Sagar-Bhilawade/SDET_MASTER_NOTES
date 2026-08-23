# Fundamentals Cheat Sheet — Warm-Up Round

> Dense recall layer for `01_TESTING_FUNDAMENTALS`. Read the full notes to learn; read
> this the morning of an interview. One-line answers you can fire back without hesitation.

---

## The distinctions they love to test

| Pair | The one-liner |
|------|---------------|
| **Smoke vs Sanity** | Smoke = build is stable enough to test (broad, shallow). Sanity = a specific fix works (narrow, deep). |
| **Regression vs Retesting** | Retesting = re-run the exact failed test after a fix. Regression = re-run others to check nothing else broke. |
| **Severity vs Priority** | Severity = technical impact (how bad). Priority = business urgency (how soon). A typo on the homepage: low severity, high priority. |
| **Verification vs Validation** | Verification = building it right (reviews, static). Validation = building the right thing (actual testing). |
| **Error / Defect / Failure** | Human makes an **error** → introduces a **defect** in code → defect executes → **failure** in behavior. |
| **QA vs QC** | QA = process-oriented, prevents defects. QC = product-oriented, finds defects. |
| **Load vs Stress vs Spike** | Load = expected volume. Stress = past the limit to find the breaking point. Spike = sudden surge. |

---

## The 7 ISTQB principles (know 4-5 by name)

1. Testing shows presence of defects, not absence
2. Exhaustive testing is impossible
3. Early testing (shift-left) saves cost
4. Defects cluster (Pareto — most bugs in few modules)
5. Pesticide paradox (same tests stop finding bugs)
6. Testing is context-dependent
7. Absence-of-errors fallacy (bug-free ≠ useful)

---

## Test design techniques (the ones they ask you to apply)

- **Equivalence Partitioning (EP):** group inputs that behave the same, test one per group.
- **Boundary Value Analysis (BVA):** bugs hide at edges — test min, min-1, min+1, max, max-1, max+1.
- **Decision Table:** combinations of conditions → actions. For complex business logic.
- **State Transition:** for state machines (login attempts → lockout).

*If asked "test a field that accepts 1-100": EP (valid 1-100, invalid <1, invalid >100) + BVA (0,1,2,99,100,101).*

---

## STLC & artifacts

- **Phases:** Requirement analysis → Test planning → Test design → Environment setup → Execution → Closure.
- **Strategy vs Plan:** Strategy = org-level approach (rarely changes). Plan = project-specific (scope, schedule, resources).
- **Scenario vs Case:** Scenario = *what* to test (high-level). Case = *how*, with steps + expected result.
- **RTM (Requirement Traceability Matrix):** maps requirements ↔ test cases. Proves coverage, finds gaps.

---

## Defect lifecycle

New → Assigned → Open → Fixed → Retest → Closed (or Reopened). Also: Deferred, Rejected, Duplicate.

**A good bug report:** title, steps to reproduce, expected vs actual, environment, severity/priority, evidence (screenshot/log/trace). *"Reproducible + specific = fixed fast."*

---

## White-box quick hits

- **Statement coverage** < **Branch coverage** < **Path coverage** (increasing rigor).
- Why 100% statement coverage is weak: it executes every line but not every *branch outcome* — an untested `else` still counts as covered.

---

## Security / usability / accessibility (what automates vs needs humans)

- **IDOR:** changing an ID in a URL to access someone else's data. Automatable.
- Automatable: injection checks, broken-auth flows, accessibility scans (axe), broken links.
- Needs humans: usability, visual polish, exploratory security, judgment calls.

---

## The 30-second framings to have ready

- **"What's your testing approach?"** → Risk-based. Cover what's most likely to break and most costly if it does, not everything equally.
- **"How do you decide what to automate?"** → Stable, repetitive, high-value, run often. Not one-offs or rapidly-changing UI. See `06/04`.
- **"How much coverage is enough?"** → Coverage is a risk conversation, not a number. 90% on critical paths beats 100% counting trivial getters.

---

## Key takeaway

> **These are cheap to answer and expensive to fumble — they're the first thing asked and
> they set the tone.** ISTQB is on your resume, so every distinction here is fair game.
> Fire them back in one line, then let the interviewer move to the real technical rounds.
