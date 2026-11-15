# LLM Evaluation — Methods, Metrics And LLM-As-Judge

## 1. What it is

"Evaluation" (evals) is how I turn "this answer looks good" into numbers. Because LLM output varies, I
can't compare to one expected string. Instead I use several evaluation methods, each with different
cost, speed and reliability, and combine them.

## 2. Why it matters in interviews

"How do you assert on LLM output?" is the single most common AI-testing question. The answer is a
toolbox: rule-based checks, reference comparisons, semantic similarity, LLM-as-judge with a rubric,
and human review — plus awareness of each one's weakness.

## 3. 30-second answer

"I don't use exact match. I layer: deterministic checks for format and must-have/must-not content;
semantic similarity to a reference; an LLM judge scoring against a clear rubric for things like
faithfulness and relevance; and human review on a sample to calibrate the judge. I run these over a
dataset, report scores, and gate on thresholds."

## 4. 2-minute answer

**Method 1 — Deterministic/code checks (cheapest, most reliable).**
Valid JSON/schema, required fields, length limits, regex for forbidden patterns (card numbers, emails),
must contain a citation, must contain a specific fact ("30 days"), must not contain competitor
names, response language. Fast, free, repeatable. Limited to what's checkable by rule.

**Method 2 — Reference-based text overlap (BLEU, ROUGE, exact match).**
Compare words with a reference. Cheap but poor for free-form answers: a correct paraphrase scores
low. Fine for short factual answers, extraction tasks, or translation-like tasks. Know the names; don't
lean on them.

**Method 3 — Semantic similarity (embeddings).**
Embed the answer and the reference; use cosine similarity. Handles paraphrase. Weakness: similar
topic ≠ same facts — "refund in 30 days" and "refund in 60 days" look very close. Use with a threshold
and a factual check.

**Method 4 — LLM-as-judge.**
A (usually stronger) LLM scores the output against a rubric: "Is every claim supported by the context?
Score 1–5 and explain." Variants: *pointwise* (score one answer), *pairwise* (which of two is better —
good for comparing prompt versions), *reference-guided*. Scales well and handles nuance.

Known weaknesses of judges:
- **Position bias** — prefers the first (or last) answer shown; fix by swapping order and averaging.
- **Verbosity bias** — prefers longer, more fluent answers, even when wrong; fix with rubric items that
  penalise padding.
- **Self-preference** — a model may favour its own style; use a different model family for judging.
- **Variance** — scores change between runs; use low temperature, multiple samples, and track
  distributions.
- **Rewards well-written hallucinations** — fluent wrong answers can score high; check key claims with
  deterministic facts.
- **Prompt sensitivity** — rubric wording changes scores; version the rubric like code.

Reduce risk: explicit rubric with anchors ("5 = every claim traceable to context; 1 = invented facts"),
ask for reasoning before the score, binary or small scales over 1–10, calibrate with human-labelled
examples, measure judge–human agreement (e.g. Cohen's kappa), and spot-check.

**Method 5 — Human evaluation (gold standard, slowest).**
Subject experts rate samples. Use it to build the golden dataset, calibrate judges, and review
failures. Not scalable for every run.

**Method 6 — Production signals.**
User thumbs up/down, escalation rate, edits, complaints — evidence after release.

**Metric families to name:**
- *Quality:* correctness, relevance, completeness, coherence, tone.
- *Grounding:* faithfulness/groundedness, citation accuracy, hallucination rate.
- *Safety:* toxicity, bias, PII leakage, jailbreak success rate, refusal correctness.
- *Operational:* latency, tokens, cost, error rate.

**Handling non-determinism statistically:** run each case n times (e.g. 3–5), report pass rate; compare
score distributions between versions; use thresholds with a margin; need a reasonable dataset size —
tiny sets give noisy scores.

**Classification-style tasks** (sentiment, intent, fraud) use classic ML metrics: accuracy, precision,
recall, F1, confusion matrix — and check fairness across groups.

## 5. Real project example (honest bridge)

The mindset matches my review-gate habits: a test is only useful if the assertion proves something.
"Visibility-only assertions are green but meaningless" has an AI twin: "an LLM judge that rewards
fluency is green but meaningless." In CIP_LEGACY, expected results were exact; here they're criteria,
so my job is writing the criteria (the rubric) with the same rigour as test cases, and checking the
checker.

## 6. Common interview Q&A

**Q: How do you assert on non-deterministic output?** Properties and scores, not equality: deterministic
checks, semantic similarity, LLM-judge rubric, plus human calibration; thresholds over a dataset.

**Q: What is LLM-as-judge and its risks?** An LLM scoring outputs against a rubric. Risks: position and
verbosity bias, self-preference, variance, rewarding fluent errors. Mitigate with rubrics, order
swapping, different judge model, calibration with humans.

**Q: Why not BLEU/ROUGE?** They measure word overlap, penalise valid paraphrases, and say nothing about
factual correctness.

**Q: How do you compare two prompt versions?** Same dataset, pairwise judging with swapped order,
plus metrics; look at distributions and failure categories, not one average.

**Q: How do you trust the judge?** Calibrate against human-labelled samples and track agreement; recheck
when the judge model changes.

## 7. Weak areas to watch

- Presenting LLM-judge scores as ground truth.
- Using the same model to generate and judge.
- Averages hiding bad failure categories (look at the worst cases).
- Not versioning rubrics and datasets.

## 8. Key takeaway

Layer cheap deterministic checks, semantic similarity and a calibrated LLM judge, and verify the
judge itself. Evaluate the evaluator.
