# AI Basics For QA — LLMs, Tokens, Context, Hallucination

## 1. What it is

An LLM (large language model) is a model trained on huge amounts of text to predict the next
piece of text. That's the whole trick: it completes patterns. When I give it a Jira story and
ask for test cases, it isn't "understanding the product" — it is producing the most likely
useful continuation of my prompt. That is powerful for drafting and weak for hidden business
rules it was never told.

The vocabulary an SDET should be able to say in plain words:

| Term | Plain meaning | Why a tester cares |
|---|---|---|
| **Token** | A chunk of text (word piece). Models read and bill in tokens | Long prompts cost more and can get cut off |
| **Context window** | How much text the model can look at in one go (prompt + answer) | Past the limit, it silently forgets the start |
| **Prompt** | Everything I send: instructions + context + examples + question | Output quality is mostly prompt quality |
| **System prompt** | Hidden standing instructions set by the app/developer | Where "you are a QA assistant, never invent data" lives |
| **Temperature** | Randomness dial. Low = steadier, high = more varied | Same prompt, different answers — non-determinism |
| **Hallucination** | Confident output that is false or unsupported | The #1 risk in QA drafts (invented steps, fields, APIs) |
| **Grounding** | Making the answer rest on supplied facts (story, logs, docs) | Cure for hallucination: give it the source, tell it to stay in it |
| **Embedding** | A list of numbers representing the meaning of text | Powers search by meaning (used in RAG — section 12) |
| **Fine-tuning vs prompting** | Retrain the model vs just instruct it | Most QA use is prompting; no training involved |

## 2. Why it matters in interviews

Interviewers use this to check whether I'm a practical user or a buzzword user. "We use AI for
test cases" is weak. A strong answer shows I know the failure modes: it hallucinates, it forgets
beyond the context window, it gives different answers each run, so I supply context, constrain
the output, and review everything.

## 3. 30-second answer

"An LLM predicts likely text from the prompt, so it's great at drafting and pattern work and weak
at facts it wasn't given. It can hallucinate, it has a limited context window, and its output varies
between runs. So I give it scoped context — story, expected behaviour, an existing test as a
style example — constrain the output format, and review everything before it goes into the suite."

## 4. 2-minute answer

Three properties drive almost every decision:

1. **It completes patterns, it doesn't verify truth.** A hallucinated locator or an invented API
   field looks exactly as confident as a correct one. So AI output is a draft, never evidence.
2. **It only knows what is in the context.** The model has general knowledge, but not my app's
   rules. Anything business-specific has to be in the prompt. Too much irrelevant context also
   hurts — the answer gets generic. Scoped, high-quality context beats a giant paste.
3. **It's non-deterministic.** Run the same prompt twice and get two answers. For drafting that's
   fine. It matters later when AI is *inside* the product I'm testing (section 12), because the
   same input may give different valid outputs and I can't just assert equality.

What I do about it: give role + task + context + constraints + output format; ask it to list
assumptions and questions instead of guessing; keep temperature low when I want stable output
(where the tool allows it); verify by running the code and checking assertions.

Model choice, at a high level: a faster/cheaper model for boilerplate, refactoring, and
explanations; a stronger reasoning model for strategy, impact analysis, and debugging. I don't
claim model internals — I treat model choice as a cost/quality trade-off.

## 5. Real project example

On BASES_PLAYWRIGHT I use GitHub Copilot Chat and Playwright CLI workflows to draft and maintain
tests, which gave about a **30% efficiency gain** (verified metric). The gain is on drafting
and maintenance — boilerplate, locator candidates, refactors. It is not on deciding what to test or
whether the result is right. The review gate (note 07) is what makes the 30% safe.

## 6. Common interview Q&A

**Q: What is an LLM, simply?** A model that predicts the next text from context; good at language
patterns, not a source of truth.

**Q: Why does AI give wrong answers confidently?** It generates plausible text; there's no built-in
fact check. That's hallucination. Mitigation: ground it in supplied facts and verify.

**Q: What is a context window and why care?** The limit on how much it can consider at once.
Overflow means it drops earlier content; irrelevant content dilutes the answer. I send only what
matters.

**Q: Why is AI output different each time?** Sampling randomness (temperature). Fine for drafts;
the reason AI features are hard to test with exact-match assertions.

**Q: Fine-tuning or prompting?** For QA assistance, prompting with good context. Fine-tuning is a
heavier choice I haven't needed.

## 7. Weak areas to watch

- Saying the model "understands" the application. Say "it infers from the context I give it".
- Claiming knowledge of model internals (attention, training). Stay at behaviour level.
- Treating a long AI-generated test list as good coverage. Good QA is deletion.
- Pasting internal or customer data into a public AI tool (see note 18).

## 8. Key takeaway

AI drafts fast and is confidently wrong sometimes. My value is the context I give it and the review
I apply — that's what turns a 30% faster draft into a trustworthy test.
