# Testing A RAG Pipeline — Retrieval And Generation Metrics

## 1. What it is

Testing RAG means measuring two things separately and then together:

- **Retrieval quality** — did the system find the right documents/chunks?
- **Generation quality** — given those chunks, was the answer faithful, relevant and complete?

Each gets metrics, a test dataset, and a threshold. This is the core of "how do you test RAG".

## 2. Why it matters in interviews

The expected answer isn't "I'd ask it questions and see." It's: build a labelled dataset, measure retrieval
and generation with named metrics, set thresholds, and run it as regression. Naming the metrics
correctly is the credibility signal.

## 3. 30-second answer

"I build a dataset of questions with ground-truth answers and the source documents they come from.
For retrieval I measure recall@k, precision@k and ranking quality like MRR or nDCG. For generation I
measure faithfulness — is every claim supported by the retrieved context — answer relevancy, and
correctness against the reference. I include unanswerable and adversarial questions, set thresholds, and
run the set in CI whenever the prompt, model, or index changes."

## 4. 2-minute answer

**Step 1: the test dataset.** Each row: *question*, *reference answer* (written or approved by a subject
expert), and *source chunk/document IDs that contain the answer*. Cover categories:

| Category | Why |
|---|---|
| Simple fact lookup | Baseline |
| Multi-chunk / multi-document questions | Tests retrieval breadth and synthesis |
| Paraphrased / synonym / acronym questions | Embedding robustness |
| **Unanswerable** (not in the docs) | Correct behaviour is "I don't know"; catches hallucination |
| Ambiguous questions | Should ask to clarify or state assumption |
| Conflicting or outdated documents | Version handling |
| Permission-restricted content | Access control |
| Adversarial / injection attempts | Safety (note 08) |

**Step 2: retrieval metrics** (given the known correct chunk IDs):

- **Recall@k:** of the relevant chunks, how many appear in the top k? (Definitions vary slightly; some
  use "at least one relevant chunk in top k." State which you use.) Low recall means the answer can't
  be right because the evidence never reached the model.
- **Precision@k:** of the k retrieved chunks, how many are relevant? Low precision means noise that
  distracts the model and wastes tokens.
- **MRR (mean reciprocal rank):** how high the first relevant chunk is ranked; 1st place → 1, 2nd → 0.5.
- **nDCG@k:** ranking quality when relevance is graded; rewards putting the best chunks first.
- **Context precision / context recall** (RAG-evaluation framing, used by Ragas/DeepEval): context
  precision checks that relevant chunks are ranked above irrelevant ones; context recall checks the
  retrieved context contains what's needed to produce the reference answer.

**Step 3: generation metrics** (given question + retrieved context + answer, often judged by an LLM
against a rubric):

- **Faithfulness / groundedness:** are the answer's claims supported by the retrieved context? The
  hallucination detector. A grounded but irrelevant answer can still score high — so pair it with:
- **Answer relevancy:** does the answer actually address the question?
- **Answer correctness:** does it match the reference answer (semantically)?
- **Completeness:** did it cover all parts?
- **Citation accuracy:** do cited sources really contain the claim?
- **Refusal correctness:** for unanswerable questions, did it decline rather than invent?
- **Tone/format:** follows system instructions (length, language, structure).

**Step 4: diagnose with the matrix.**

| Retrieval | Generation | Meaning |
|---|---|---|
| Good | Good | Healthy |
| Good | Bad | Prompt/model problem |
| Bad | Bad (or lucky-good) | Retrieval/ingestion problem |
| Bad | "Good" | Answer from model memory — risky; not grounded |

**Step 5: thresholds and regression.** Example gates: recall@5 ≥ 0.85, faithfulness ≥ 0.9, unanswerable
refusal ≥ 0.95. Pick thresholds with the product owner from a baseline run, then ratchet. Run on
every change to prompt, model, chunking, embedding model, index data.

**Step 6: classical tests alongside:** API contract of the question/answer endpoint, citations present
in the schema, permissions, latency under load, error handling when the vector DB or LLM is down.

**Pitfalls:** testing only happy questions; datasets written by the same people who built the prompt
(bias); relying only on LLM-judged scores without spot checks by humans; no baseline so no way to see a
regression; tiny datasets (scores jump around).

## 5. Real project example (honest bridge)

No production RAG experience — say so. How I'd approach: treat the dataset like a regression suite
(my CIP_LEGACY habit: 300+ scenarios with clear expected results), the thresholds like quality gates
in CI (Azure DevOps/GitHub Actions in BASES_PLAYWRIGHT), and the reporting like Allure — a per-case
table with scores. Load/latency of the answer endpoint is a JMeter job, where my P90/P95 reading
experience directly applies.

## 6. Common interview Q&A

**Q: How would you test a RAG chatbot?** Dataset with references and source IDs; retrieval metrics
(recall@k, precision@k, MRR/nDCG); generation metrics (faithfulness, relevancy, correctness);
unanswerable and adversarial cases; thresholds in CI; classical tests around it.

**Q: Faithfulness vs answer relevancy?** Faithfulness: claims supported by the context. Relevancy: answer
addresses the question. You need both — an answer can be faithful but off-topic, or relevant but invented.

**Q: What's recall@k?** Whether the needed chunk(s) appear in the top k results. Low recall = the model
never sees the evidence.

**Q: How do you test "I don't know" behaviour?** Questions whose answers aren't in the documents; expect a
refusal or a clear statement of insufficient information, not a guess.

**Q: How do you know the judge LLM is reliable?** Calibrate against human-labelled samples, fix
the rubric, use low temperature, check agreement; don't trust unreviewed scores.

## 7. Weak areas to watch

- Using one metric ("accuracy") for everything.
- Forgetting unanswerable questions.
- Not separating retrieval from generation when diagnosing.
- Presenting thresholds as universal — they come from baselines and risk.

## 8. Key takeaway

RAG testing = labelled dataset + retrieval metrics + generation metrics + unanswerable cases + CI
thresholds. Always diagnose retrieval first.
