# LLM Fundamentals For Testers

## 1. What it is

To test an LLM application I need a working picture of how it's built, the same way I need to
understand a web app's layers before testing it. This note is the minimum technical model: what's
sent to the model, what controls the output, how "search by meaning" works, and what a typical LLM
app looks like.

## 2. Why it matters in interviews

Questions like "what's a token?", "what does temperature do?", "what is an embedding?" are quick
credibility checks. If I can't answer them, my testing answers sound shallow.

## 3. 30-second answer

"An LLM app sends the model a prompt made of a system instruction, the user's question, and any extra
context. The model returns text sampled token by token, shaped by settings like temperature. For search
by meaning it uses embeddings — numeric vectors of text — stored in a vector database. As a tester I care
about what goes into the prompt, what settings are used, and what the app does with the output."

## 4. 2-minute answer

**Tokens.** The model reads and writes tokens (pieces of words). Limits and cost are measured in
tokens. Roughly, a token is a few characters. Test implication: very long inputs can be truncated or
rejected; long outputs cost more and take longer.

**Context window.** The maximum tokens (input + output) the model can handle at once. Beyond it, content
is dropped or the call fails. Test implication: long chat histories, big documents, many retrieved
chunks.

**Message roles.** A request is typically a list of messages: *system* (standing rules: "You are a
support assistant; answer only from the provided documents"), *user* (the question), *assistant*
(previous replies), and sometimes tool results. The system prompt is part of the product and must be
tested — and should be treated as potentially extractable by users.

**Sampling settings.**
- **Temperature:** low → steadier, more predictable; high → more varied/creative.
- **Top-p:** another way to limit which tokens can be chosen.
- **Max tokens:** cap on response length (cuts answers mid-sentence if too low).
- **Stop sequences:** text that ends generation.
Test implication: tests are steadier at low temperature, but even then results aren't guaranteed
identical; production settings should be tested too.

**Structured output / function calling.** Apps often ask the model for JSON that matches a schema or to
"call a tool". Testers validate the JSON against the schema, handle malformed output, and check the
chosen tool and arguments.

**Embeddings.** A model turns text into a vector (a long list of numbers) so that texts with similar
meaning sit close together. "How do I get my money back?" lands near "Refund policy". Closeness is
measured by cosine similarity. This is how semantic search works — and it is how RAG finds the
right documents.

**Vector database.** Stores embeddings plus the original text and metadata, and answers "give me the k
nearest chunks to this query vector." Examples include FAISS, Chroma, pgvector, Pinecone — I only need
the concept.

**Prompting vs RAG vs fine-tuning.**
- *Prompting:* instruct the model with good context. Cheapest, most common.
- *RAG:* fetch relevant documents at question time and put them in the prompt. For private or fresh
  knowledge. (Notes 03–04.)
- *Fine-tuning:* retrain on examples to change style/behaviour. Heavier, less common for knowledge.

**Typical LLM app architecture:**

```
User → App/API → (guardrails: input checks)
     → build prompt (system + history + retrieved context + question)
     → LLM call (model, temperature, max tokens)
     → (guardrails: output checks, schema validation)
     → response to user (+ logging, feedback)
```

Every arrow is a test point: input validation, prompt building, model call failures, output parsing,
logging of PII, and permissions on the data fed into the prompt.

**Where non-determinism enters:** the sampling step, plus any retrieval that changes when the index
changes, plus model updates by the provider.

## 5. Real project example (honest bridge)

I haven't built an LLM app, but the testing mindset maps directly: the "build prompt" and "parse
output" stages are plain code I can unit test; the "call the LLM" stage is an external dependency I can mock
(like stubbing an API in RestAssured tests); and the service endpoints get API tests. My JMeter
experience applies to the endpoint latency and concurrency questions.

## 6. Common interview Q&A

**Q: What is a token?** A chunk of text the model processes; limits and cost are counted in tokens.

**Q: What does temperature do?** Controls randomness of generation: low = more consistent, high = more
varied. For tests I prefer low, but I still don't assume exact repeatability.

**Q: What's an embedding?** A numeric vector representing meaning; similar texts are close, used for
semantic search.

**Q: Where are the test points in an LLM app?** Input validation, prompt construction, the model call
(timeouts, 429s), output parsing/schema, guardrails, logging, and permissions on retrieved data.

**Q: Why mock the LLM in some tests?** Speed, cost, determinism — to test my code around it; real-model
tests are a smaller separate evaluation set.

## 7. Weak areas to watch

- Overstating knowledge of model internals; stay at behaviour/system level.
- Forgetting that the system prompt itself is something to test.
- Forgetting max-token truncation and context overflow cases.

## 8. Key takeaway

An LLM app is normal code around a probabilistic model call. Know the prompt, the settings, the
retrieval and the output handling — each is a test point.
