# DeepEval And Ragas — Evaluation Frameworks

> Honest status: I've studied these, not run them in production. The code below follows the
> **official docs** (deepeval.com getting-started and Faithfulness pages; docs.ragas.io RAG-eval guide),
> checked on 2026-11-15. It was **not executed** by me. Libraries change often, so pin versions and
> re-check the docs before copying. In an interview say "I've explored them"; don't claim project use.

## 1. What it is

**DeepEval** and **Ragas** are open-source Python libraries that run the evaluation methods from note
05 for you. They provide ready-made metrics (many use an LLM as the judge) and let you run them over a
dataset.

- **DeepEval** feels like **pytest for LLMs**: you write test cases, attach metrics with thresholds,
  and assert. It plugs into CI the way my Playwright/TestNG suites do.
- **Ragas** is built around **evaluating RAG pipelines**: you give it a dataset of
  question/context/answer/reference and it returns metric scores per row and on average.

Both need an LLM (the "judge") configured — often an API key — and some metrics also use embeddings.

## 2. Why it matters in interviews

"What tools would you use to test an LLM app?" Naming DeepEval/Ragas (plus promptfoo for prompt
comparison and red-teaming, and garak as a vulnerability scanner) and explaining how they fit shows
current knowledge. Interviewers also test that I know their limits.

## 3. 30-second answer

"DeepEval gives me a pytest-style way to assert on LLM output with metrics like faithfulness, answer
relevancy, contextual precision/recall, hallucination and custom G-Eval criteria, with thresholds so a
run can fail CI. Ragas is more dataset-oriented for RAG: it scores faithfulness, answer relevancy and
context precision/recall across a test set. Both rely on an LLM judge, so I calibrate and watch cost and
variance."

## 4. 2-minute answer

**DeepEval — pytest style**

Core objects: a *test case* (input, actual output, optionally expected output and retrieval context), a
*metric* (with a threshold), and an assert. Typical shape:

```python
# test_rag.py  (shape follows the official DeepEval docs)
from deepeval import assert_test
from deepeval.test_case import LLMTestCase
from deepeval.metrics import FaithfulnessMetric, AnswerRelevancyMetric

def test_refund_question():
    answer, chunks = my_rag_app("What is the refund window?")   # my app under test
    case = LLMTestCase(
        input="What is the refund window?",
        actual_output=answer,
        retrieval_context=chunks,          # list of strings; required for Faithfulness
    )
    assert_test(case, [
        FaithfulnessMetric(threshold=0.8),       # defaults per docs: threshold 0.5, include_reason=True
        AnswerRelevancyMetric(threshold=0.7),
    ])
```

Run with the DeepEval CLI: `deepeval test run test_rag.py` (this exact command is in the official
getting-started page). It runs like a test run and reports scores. A custom-criteria metric looks like
`GEval(name=..., criteria=..., evaluation_params=[...], threshold=0.5)`; newer docs name the params
enum `SingleTurnParams` (older versions used `LLMTestCaseParams`) - another reason to pin versions. Each metric returns a score between 0 and 1 and passes if it meets the
threshold.

Metrics DeepEval commonly lists: **Answer Relevancy, Faithfulness, Contextual Precision, Contextual
Recall, Hallucination, G-Eval, Summarization, Toxicity, Bias**, plus a RAGAS-style composite. **G-Eval** is
the flexible one: you write the criteria in plain English ("The answer must not give legal advice and
must cite the policy") and it scores against that.

**Ragas — dataset style**

You prepare rows with the user's question, the retrieved contexts, the model's response, and a reference
answer, and call an evaluate function with a list of metrics:

```python
# follows the official Ragas RAG-evaluation guide; needs an LLM (and embeddings for some metrics)
from langchain_openai import ChatOpenAI
from ragas import EvaluationDataset, evaluate
from ragas.llms import LangchainLLMWrapper
from ragas.metrics import LLMContextRecall, Faithfulness, FactualCorrectness

rows = [{
    "user_input": "What is the refund window?",
    "retrieved_contexts": chunks,            # list of strings from my retriever
    "response": answer,                      # what my RAG app answered
    "reference": "30 days from delivery",    # the expected answer
}]
evaluation_dataset = EvaluationDataset.from_list(rows)
evaluator_llm = LangchainLLMWrapper(ChatOpenAI(model="gpt-4o"))   # the judge

result = evaluate(
    dataset=evaluation_dataset,
    metrics=[LLMContextRecall(), Faithfulness(), FactualCorrectness()],
    llm=evaluator_llm,
)
# example output in the docs: {'context_recall': 1.0, 'faithfulness': 0.857, 'factual_correctness': 0.728}
```

(The official page lists the RAG metrics as Faithfulness, Response Relevancy, Context Precision and
Context Recall; the guide's own example runs the three classes above. Class names for the others -
e.g. response relevancy needs an embeddings model too - should be confirmed in the docs for your version.)

The four core RAG metrics, in plain words:
- **Faithfulness:** claims in the answer are supported by the retrieved context (hallucination check).
- **Answer relevancy:** the answer addresses the question.
- **Context precision:** relevant chunks are ranked above irrelevant ones (retrieval noise/ordering).
- **Context recall:** the retrieved context contains what's needed for the reference answer.

**DeepEval vs Ragas:** DeepEval → test-suite and CI ergonomics, broader metric set (safety too).
Ragas → quick RAG-focused scoring and synthetic test-set generation. They overlap; teams often pick
one. Neither replaces human review.

**Related tools worth naming (concept only; checked against official pages):** **promptfoo** — open-source
evals and red teaming; red-team 'plugins' generate adversarial inputs and it can run in CI/CD (commands in note 08);
**garak** (NVIDIA) — LLM vulnerability scanner with probes for prompt injection, leakage, jailbreaks, toxicity, hallucination; **LangSmith/Langfuse/Arize Phoenix** — tracing and monitoring of LLM apps in dev and prod.

**Practical cautions**
- **Cost:** judge metrics make several LLM calls per case; a big dataset on every commit is expensive.
  Run a small smoke subset on PRs and the full set nightly.
- **Variance:** scores shift between runs; use thresholds with margin and multiple samples.
- **Small datasets:** under ~10 cases the numbers are unreliable.
- **Judge dependence:** change the judge model and scores shift; pin and record it.
- **Version drift:** APIs change often — pin versions.
- **Privacy:** judge calls send my data to a provider; use synthetic/masked data or an approved
  endpoint.

## 5. Real project example (honest bridge)

How it would slot into BASES_PLAYWRIGHT-style delivery: tests live in the repo, run in GitHub Actions
or Azure DevOps like Playwright tests, produce a report, and fail the pipeline below threshold.
The skill I already have is the pipeline/gating side; the new part is the metric choice and threshold
setting. I have not run DeepEval or Ragas on a real product.

## 6. Common interview Q&A

**Q: What tools do you know for evaluating LLM apps?** DeepEval (pytest-style metrics and thresholds),
Ragas (RAG metrics over datasets), promptfoo (prompt comparison + red-teaming), garak (vulnerability
scanning), and tracing tools like LangSmith/Langfuse. I've studied them; my pipeline/CI experience
carries over.

**Q: Have you used DeepEval/Ragas?** (Honest) I've explored how they work and the metrics; I haven't
used them in production. Here's how I'd set it up...

**Q: What is G-Eval?** A custom criteria metric: describe in plain language what good looks like and an
LLM scores against it.

**Q: What are the downsides?** Cost, variance, judge bias, and dependence on the judge model.

**Q: How do you fit it in CI?** Small smoke set per PR, full set nightly, threshold-based fail, report
artifacts, pinned versions and judge model.

## 7. Weak areas to watch

- Claiming hands-on use without having done it.
- Quoting exact imports/params from memory — say "check the version's docs".
- Treating a single score as truth.
- Ignoring cost and privacy of judge calls.

## 8. Key takeaway

DeepEval = pytest for LLM outputs; Ragas = RAG scoring over datasets. Both are judge-based, so
calibrate, control cost, pin versions, and be honest about experience.
