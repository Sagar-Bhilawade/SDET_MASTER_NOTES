# Testing AI Systems vs Traditional Testing

> Section 12 is about **testing products that have AI inside** (chatbots, RAG assistants, summarisers,
> agents). Section 11 is about **using AI to do QA**. Different skill, different interview questions.
> Honest framing for this whole section: this is knowledge I've studied, not production experience.
> My hands-on base is API/UI automation, JMeter load testing, and AI-assisted QA. I connect to those,
> and I never claim DeepEval/Ragas/red-teaming work I haven't done.

## 1. What it is

Traditional software is **deterministic**: same input → same output. I write an expected value, run the
test, compare. AI systems built on LLMs are **probabilistic**: the same question can produce
different, equally acceptable wordings, and sometimes a wrong one. There is often no single "correct"
string. That breaks exact-match assertions and changes how I design tests.

Think of it like testing a human employee answering customer questions, not a calculator. I don't check
for identical words; I check for **correctness, relevance, safety, and consistency**, across many examples.

## 2. Why it matters in interviews

Companies are shipping AI features and need testers who can say how they'd verify them. A candidate who
only says "I'd write Selenium tests" for a chatbot sounds dated. A strong answer explains the oracle
problem, evaluation datasets, metrics with thresholds, safety testing, and what stays classical.

## 3. 30-second answer

"AI features are non-deterministic, so I can't assert exact outputs. I test them with evaluation
datasets and metrics — correctness, relevance, groundedness, safety — with thresholds, run across many
cases, and track them in CI like regression. The deterministic parts around the model — APIs, auth,
validation, error handling — I still test the classical way."

## 4. 2-minute answer

**What changes**

| Traditional | AI / LLM system |
|---|---|
| Deterministic output | Varies run to run |
| Expected value is exact | Expected behaviour is criteria ("grounded, relevant, polite, no PII") |
| Pass/fail per test | Score per case; pass if aggregate meets a threshold |
| One input per test | A **dataset** of inputs (tens to hundreds) |
| Bugs from code changes | Behaviour also shifts from **prompt, model version, retrieval data** changes with no code change |
| Functional defects | Plus hallucination, bias, toxicity, prompt injection, data leakage, cost/latency |

**The oracle problem.** How do I know the answer is right? Options: a reference answer written by a
human; facts in the retrieved source (grounding); a rubric checked by a human or an "LLM judge";
rule-based checks (valid JSON, must cite a source, must refuse X). I usually combine them.

**Where classical testing still applies** (and this is most of an AI product): authentication and
permissions, API contracts and status codes, input validation, data storage, UI behaviour, error
handling, rate limits, logging, performance. An AI feature is a normal application with one
probabilistic component in the middle. I isolate that component and test it differently.

**A layered approach** (like a test pyramid):

1. **Unit** — prompt templates build correctly, output parsers handle bad JSON, chunking logic,
   guardrail rules. Deterministic. Fast.
2. **Component** — retriever returns the right documents for known queries; model call wrapper handles
   timeouts, 429s, empty responses (mock the LLM).
3. **Evaluation** — dataset of questions scored with metrics (the AI-specific layer).
4. **End-to-end** — a few real user journeys through UI/API, including safety cases.
5. **Production monitoring** — drift, user feedback, cost, latency, failure rate.

**Non-determinism handling:** lower temperature for tests where possible, assert on properties not
words, run each case several times and require a pass-rate, and track score distributions rather than
single runs.

**Types of AI system, quick map:**

- **Classifier/ML model** (fraud, sentiment): test with labelled data — accuracy, precision, recall, F1,
  confusion matrix, bias across groups.
- **LLM chat / generation:** relevance, correctness, tone, safety.
- **RAG assistant:** retrieval quality + grounded generation (notes 03–04).
- **Agent:** tool choice, multi-step trajectories, side effects (note 09).

## 5. Real project example (how I'd connect it to my background)

I haven't tested a production LLM feature, so I'd be honest about that. What transfers: API testing
(RestAssured/Playwright request) for the service wrapping the model; SQL checks on stored
conversations; JMeter load testing for latency under traffic (note 11); and my review-gate
mindset from AI-assisted QA — "green doesn't mean right." In BASES_PLAYWRIGHT, an AI chat feature
would get API-level contract tests plus a small evaluation set run in CI.

## 6. Common interview Q&A

**Q: How is testing an AI feature different?** Output isn't deterministic and there's no single right
answer, so I test with datasets, metrics and thresholds on properties like grounding and safety instead
of exact matches. The classical layers around the model still get classical tests.

**Q: How do you assert on an LLM response?** Not equality. Check properties: contains required facts,
cites the source, valid schema, no forbidden content, passes a semantic or rubric-based score above a
threshold.

**Q: What can change behaviour without a code change?** Model version updates, prompt edits, retrieval
index/data changes, temperature/config. So I re-run the evaluation set on any of those.

**Q: What stays the same?** API contracts, auth, validation, performance, error handling, UI.

## 7. Weak areas to watch

- Claiming I've used DeepEval/Ragas/red-team tools professionally. I haven't; say "studied".
- Trying to force exact-match assertions.
- Forgetting the classical tests around the model.
- Treating one run as proof; use several runs and a dataset.

## 8. Key takeaway

Testing AI = classical testing around the model + evaluation datasets with metrics and thresholds on
the probabilistic part. Assert on properties, not words.
