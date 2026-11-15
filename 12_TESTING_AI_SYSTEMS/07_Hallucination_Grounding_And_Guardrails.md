# Hallucination, Grounding And Guardrails

## 1. What it is

**Hallucination** is when an LLM produces fluent, confident output that is false or unsupported. It's the
headline risk of AI features. **Grounding** is making the answer depend on real, supplied sources.
**Guardrails** are checks placed before and after the model that block or fix unsafe, off-topic, or
malformed input and output.

Two kinds worth naming:
- **Fabrication (extrinsic):** invents facts not in any source — a fake policy, a non-existent API, a made-up
  citation.
- **Contradiction (intrinsic):** says something that conflicts with the supplied context.

## 2. Why it matters in interviews

"How do you test for hallucinations?" is a standard question. The strong answer has detection
(compare claims to sources), prevention (grounding, "I don't know"), and guardrail testing with both
false-negative and false-positive thinking.

## 3. 30-second answer

"I test hallucination by checking whether each claim in the answer is supported by the supplied
context — faithfulness — using unanswerable questions, conflicting-source questions, and fake-premise
questions, and by checking citations actually contain the claim. Prevention is grounding, an
explicit 'say you don't know' rule, good retrieval and constrained output. For guardrails I test
both that they block bad cases and that they don't block good ones."

## 4. 2-minute answer

**Why models hallucinate.** They predict plausible text; when the answer isn't in context, a confident
guess is the easiest continuation. Also: weak retrieval gives wrong evidence, long contexts lose
details, and leading questions ("Why is the refund 90 days?") invite agreement with a false premise.

**Test cases that expose it**
- **Unanswerable questions:** answer isn't in the documents. Expected: refusal or "not found."
- **False-premise questions:** "Why does the product support feature X?" (it doesn't). Expected:
  correction.
- **Conflicting documents:** old and new policy. Expected: use the latest or flag conflict.
- **Numbers, dates, names, IDs:** high-risk facts must match source exactly.
- **Citation checks:** each cited source actually contains the claim; no invented sources/URLs.
- **Out-of-scope questions:** medical/legal/financial advice the system shouldn't give.
- **Long-context stress:** answer buried in the middle of many chunks.
- **Consistency:** ask the same thing several ways; answers shouldn't contradict each other.

**Detecting hallucination**
- *Faithfulness metrics* (note 04/06): break the answer into claims, check each against the context.
- *Source-grounded checks:* required facts present, forbidden claims absent.
- *Self-consistency:* sample multiple answers; large disagreement signals uncertainty.
- *Human review* on a sample, especially for high-stakes domains.

**Prevention and mitigation (what the dev team does; what I verify)**
- Strong retrieval; answer only from context; explicit "if not in context, say you don't know."
- Show citations; low temperature for factual tasks; structured output; limit scope.
- Human-in-the-loop for high-risk actions.

**Guardrails**
Input guardrails: block prompt injection patterns, PII, off-topic or abusive input, oversized input.
Output guardrails: moderation (toxicity/self-harm/hate), PII redaction, schema validation, "must cite
sources", banned-topic filters, factuality check against context.

Testing guardrails needs both sides:
- **False negatives (bad content gets through):** attack/unsafe set — should be blocked.
- **False positives (good content blocked):** normal, legitimate set — should pass. Over-blocking is a
  product defect too ("over-refusal").
- Track as precision/recall of the guardrail, per category.
- Also test the failure path: what does the user see when something is blocked? Is it logged? Is the
  fallback message safe and helpful?

**Severity thinking:** a hallucinated joke is low; a hallucinated refund policy, medical dose, or legal
claim is high. Risk-rank hallucination tests by domain impact — the same instinct as risk-based
regression.

## 5. Real project example (honest bridge)

I haven't shipped a guardrail suite. The method mirrors my test design habits: equivalence classes
(normal, edge, malicious inputs), boundary cases (very long inputs), and negative testing — plus the
classic QA instinct that testing only "blocked correctly" misses the defect where valid users get blocked.
In CIP_LEGACY, validating large datasets taught me to check precision and recall of rules, not just "it
caught something."

## 6. Common interview Q&A

**Q: How do you test for hallucinations?** Unanswerable and false-premise questions, claim-level
faithfulness against the context, citation verification, high-risk facts matched to source, and
consistency checks — plus human spot checks.

**Q: How do you reduce hallucinations?** Grounding via RAG with good retrieval, "don't know" rule, constrained output, low temperature,
citations, scope limits, human review for high-risk — I verify these work.

**Q: What is a guardrail?** A check before/after the model that blocks or fixes unsafe or malformed input
and output.

**Q: How do you test guardrails?** Unsafe set must be blocked, normal set must pass, measure both error
types, and check the user-facing fallback.

**Q: What is over-refusal?** The system refuses legitimate requests due to overly strict rules — a
usability defect.

## 7. Weak areas to watch

- Only testing happy-path answers.
- Forgetting false positives in guardrails.
- Treating all hallucinations as equal; rank by impact.
- Claiming zero hallucinations is achievable; it's about measured rates and mitigations.

## 8. Key takeaway

Test hallucination with questions designed to tempt it, measure faithfulness against sources, and test
guardrails for both leaking and over-blocking.
