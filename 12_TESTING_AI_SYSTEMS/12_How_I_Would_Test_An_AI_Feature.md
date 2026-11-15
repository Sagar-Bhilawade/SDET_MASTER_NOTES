# How I Would Test An AI Feature — Walkthroughs And Q&A Bank

## 1. What it is

The interview version of section 12: a repeatable framework plus three worked examples, so I can answer
"how would you test X?" for an AI feature in a structured way — and be honest about experience.

## 2. Why it matters in interviews

This is the actual question. Interviewers may give me a feature ("we have a support chatbot") and watch me
structure the problem. Structure, trade-offs, and honesty are what get scored.

## 3. 30-second answer (the umbrella)

"I'd start with the risk: what's the worst thing this feature could do wrong. Then test in layers —
classical tests on the APIs, permissions and error handling; component tests with a mocked model; an
evaluation dataset with metrics for quality and grounding; safety and red-team tests; performance and cost;
and monitoring after release. I assert on properties and scores with thresholds, not exact text, and I
re-run the set whenever prompt, model, or data change."

## 4. 2-minute answer — the framework

1. **Understand the feature and the risk.** Who uses it, what decisions depend on it, what's the
   worst failure (wrong medical/financial/legal answer, data leak, harmful content)? Risk sets depth.
2. **Define "good".** With the product owner write acceptance criteria as measurable properties:
   correct, grounded, relevant, safe, polite, within scope, fast enough, within budget.
3. **Map the architecture** (note 02/03): prompt, model, retrieval, tools, guardrails, output handling.
4. **Layer the tests:**
   - Classical: API contract, auth/permissions, validation, error handling, logging/PII.
   - Component: prompt building, parsers, retrieval for known queries, guardrail rules; mock the LLM.
   - Evaluation dataset: normal, edge, unanswerable, adversarial; metrics + thresholds (notes 04–06).
   - Safety/security: injection, leakage, jailbreaks, agency (note 08).
   - Non-functional: load, latency, TTFT, tokens/cost, rate limits (note 11).
   - Exploratory/human review for tone and nuance.
5. **Automate and gate:** smoke on PR, full nightly, versioned dataset/prompt/judge (note 10).
6. **Monitor in production:** feedback, drift, cost, failure rate; loop failures into the dataset.
7. **Communicate limits:** AI quality is statistical — report rates, categories and residual risk, not "100%".

### Walkthrough A — Customer-support RAG chatbot
- *Risk:* wrong policy answers, leaking another customer's data.
- *Classical:* auth on the chat API, conversation storage, PII in logs.
- *Retrieval:* question→source set; recall@k, precision@k; stale/removed doc checks.
- *Generation:* faithfulness, relevancy, correctness; unanswerable → "I don't know/hand off to human".
- *Safety:* injection via uploaded docs, system prompt extraction, cross-user data.
- *Load:* JMeter with stubbed LLM, then small real runs; 429 handling.
- *Gate:* critical-category thresholds, release blocked on any safety failure.

### Walkthrough B — Text summariser
- *Risk:* omits key facts, adds facts, distorts numbers.
- *Checks:* key-fact coverage (required facts list), no new facts (faithfulness), length/format limits,
  number/date exactness (deterministic), tone; test on long, messy, multilingual, empty, and
  adversarial inputs; consistency across runs.
- *Metrics:* faithfulness, coverage, human spot-check; thresholds.

### Walkthrough C — AI test-case generator (a tool for QA teams)
- *Risk:* plausible but invalid cases, missing risks, duplicates.
- *Checks:* given known stories with known defect-prone areas, does it surface them (recall of known
  scenarios)? Are generated cases traceable to the story (no invention)? Duplicate rate? Executable
  steps? Human review sample rated against a rubric.
- *Link to note 11:* the same "AI gives breadth, QA gives depth" principle, now measured.

### Honest-positioning script
"I haven't owned testing of a production LLM feature yet. My AI experience is using AI-assisted QA —
Copilot Chat and Playwright CLI workflows, about 30% faster on drafting and maintenance — and I've gone
deep on how to evaluate AI products: RAG metrics, DeepEval/Ragas, guardrails and red-teaming.
My engineering base — API testing, CI quality gates, JMeter performance work — transfers directly.
Here's how I'd approach your feature..."

## 5. Real project example (how my background maps)

| Their need | What I already do |
|---|---|
| Contract and permission tests on AI endpoints | API testing (RestAssured, Playwright request, Postman) |
| Quality gates in CI | Azure DevOps / GitHub Actions, 40→12 min feedback loop |
| Large regression set discipline | 300+ scenarios, 90% regression coverage |
| Load and latency analysis | JMeter, 60+ APIs, P90/P95 |
| Data validation | SQL with Oracle/MySQL |
| AI workflow judgement | AI-assisted QA with review gates, ~30% drafting/maintenance gain |

## 6. Common interview Q&A bank

**Q1. How would you test an AI chatbot?** Framework above: risk → measurable criteria → layered tests
(classical, component, eval dataset, safety, load) → CI gates → monitoring.

**Q2. How do you know the answer is correct?** Reference answers and source grounding, faithfulness
checks, rubric-based judging calibrated by humans; for critical facts, deterministic checks.

**Q3. What is RAG and how do you test it?** Retrieve-then-generate; test retrieval (recall@k/precision@k/MRR)
and generation (faithfulness/relevancy/correctness) separately, plus permissions and freshness.

**Q4. How do you handle non-determinism?** Assert on properties/scores; low temperature; multiple runs and
pass-rate; thresholds with margin; pinned versions.

**Q5. What is LLM-as-judge?** An LLM scoring outputs against a rubric; calibrate against humans and watch
position/verbosity bias.

**Q6. Which tools?** DeepEval, Ragas, promptfoo, garak, plus tracing like LangSmith/Langfuse — studied,
not used in production.

**Q7. How do you test for hallucination?** Unanswerable and false-premise questions, claim-level
faithfulness, citation checks, high-risk fact matching.

**Q8. Prompt injection?** Direct and indirect; layered defences; test with attack sets; treat model output as untrusted.

**Q9. How do you test an agent?** Outcome, trajectory, tool-call accuracy, safety; mock tools; fault
injection; repeated runs.

**Q10. Performance for LLM apps?** JMeter on the service, stubbed model for scale, small real runs for
latency/cost, TTFT, 429 handling.

**Q11. What would you automate vs keep manual?** Automate regression on the dataset, deterministic checks,
safety attack set, API tests. Keep exploratory testing, tone review, and calibration of judges human.

**Q12. Who decides the quality bar?** I propose metrics and thresholds from a baseline; product and risk
owners agree them; I report rates and residual risk.

**Q13. Have you tested an AI product?** (Honest) Not an LLM feature in production; I've used AI-assisted
QA daily and prepared the evaluation approach in depth. Offer the walkthrough.

**Q14. What would you do first on day one?** Map architecture and risks, build a small golden set of 30–50
cases with the team, baseline the metrics, wire a smoke run into CI.

## 7. Weak areas to watch

- Skipping the honest-positioning line and being caught over-claiming.
- Starting with tools instead of risk.
- Forgetting the classical tests around the model.
- Promising 100% correctness.

## 8. Key takeaway

Risk first, measurable criteria, layered tests, thresholds in CI, monitoring after release — and say
clearly what I've done versus what I've studied.
