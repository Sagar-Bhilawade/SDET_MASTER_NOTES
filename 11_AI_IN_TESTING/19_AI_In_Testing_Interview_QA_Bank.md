# AI In Testing — Interview Q&A Bank

Short, speakable answers. Tied to what I actually did: practical GitHub Copilot Chat and Playwright
CLI workflows on BASES_PLAYWRIGHT, about a 30% gain on drafting and maintenance, with review gates.
No self-healing, no autonomous Planner/Generator/Healer in production.

## 1. What it is

A bank of the questions interviewers ask about AI in QA, with the answer shape I use. Read it after
notes 01–18; rehearse aloud.

## 2. Why it matters

AI questions are now in most SDET loops. The score is on honesty, boundaries and judgement, not
tool name-dropping.

## 3. 30-second answer (the umbrella)

"I use AI as an assistant across the lifecycle — story analysis, test ideas, drafting Playwright tests,
locator exploration, debugging evidence, bug-report cleanup — with scoped context and a review gate.
About 30% faster on drafting and maintenance. Judgement — what to test, assertions, release confidence —
stays with me."

## 4. 2-minute answer

Lead with the workflow (story → risk-ranked scenarios → manual-to-automation draft → hardening →
review gate → CI). Then the boundaries (no data leakage, no AI-owned severity, no self-healing claim).
Then the metric (30%). Then one example from BASES_PLAYWRIGHT.

## 5. Real project example

Manual flow → AI draft in our fixtures/locator style → I harden assertions, data and waits → review
gate → run in CI. That's the example I give for almost every question below.

## 6. Common interview Q&A

**Q1. How do you use AI in your testing?** Analysis and drafting: story/PR analysis, scenario
brainstorming, Playwright drafts, locator exploration, failure triage, bug-report cleanup. Reviewed
before it enters the suite.

**Q2. What does AI do badly?** Hidden business rules, real-environment facts, prioritisation, and
it can hallucinate. It also pads: long lists, weak assertions.

**Q3. How do you review AI-generated tests?** Six questions: does it prove business behaviour? are
assertions meaningful? are locators stable? is data isolated? parallel-safe? does it add regression
value or just maintenance?

**Q4. What is an agent?** A goal-driven AI workflow that can use tools and context to do a bounded task. I
use agent-style CLI workflows for browser exploration and drafting; I don't run an autonomous agent
that changes the suite on its own.

**Q5. Copilot modes?** Ask (questions), Edit (scoped changes), Agent (multi-step in the workspace),
Plan (think first). I choose by risk: Ask/Plan for analysis, Edit for small refactors, Agent only with
tight scope and review.

**Q6. Playwright CLI vs MCP?** CLI is token-efficient — snapshots saved locally, refs to act on
elements; MCP streams more context into the model. I describe my work as Playwright CLI agent-style
workflow, not MCP.

**Q7. Self-healing locators?** I don't run self-healing in production. I prefer stable user-facing
locators and a review process; self-healing can hide real UI regressions.

**Q8. Does AI replace testers?** No. It removes boilerplate; risk judgement, domain knowledge,
exploratory thinking and release confidence remain human.

**Q9. How do you measure the gain?** Drafting and maintenance time against baseline — about 30% on
BASES_PLAYWRIGHT. I don't claim it improves product quality by itself.

**Q10. Data privacy?** No production/customer/internal confidential data in external tools; synthetic or
masked data; approved tools.

**Q11. How do you stop hallucinated steps?** Ground in the story/PR, forbid invention, list unknowns as
questions, run the code.

**Q12. Biggest risk of AI-generated tests?** Green but meaningless — visibility-only assertions and
duplicates. The fix is the review gate.

**Q13. Would you test an AI feature?** Yes — differently: no exact-match assertions, use evaluation sets,
metrics for grounding/relevance, safety tests. That's section 12.

**Q14. Which model do you use?** I focus on context quality, not model names; faster model for
boilerplate, stronger reasoning for analysis/debugging.

**Q15. Show me a prompt.** Role + task + only-relevant context + constraints ("don't invent, list unknowns")
+ output table + one existing test as style example.

## 7. Weak areas to watch

- Name-dropping tools without a workflow.
- Overclaiming (agents, self-healing, MCP as a resume skill).
- Forgetting the review gate and privacy in the answer.
- No number: always have "about 30% on drafting/maintenance" ready, and say what it measures.

## 8. Key takeaway

Every AI answer has four beats: workflow, boundary, review gate, honest number.
