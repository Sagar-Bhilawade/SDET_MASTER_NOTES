# RAG — Architecture And Failure Modes

## 1. What it is

**RAG = Retrieval-Augmented Generation.** An LLM on its own only knows what it was trained on. It
doesn't know my company's policies, last week's release notes, or a private knowledge base. RAG fixes
that: when a user asks a question, the app first **retrieves** the most relevant documents from my own
data, puts them in the prompt, and asks the model to **generate** an answer using them.

Analogy: an open-book exam. The model is the student; the retrieval step finds the right pages of the
textbook; the answer should come from those pages, not from memory.

Why companies use it: private/fresh knowledge, fewer hallucinations, answers with source citations,
no retraining.

## 2. Why it matters in interviews

RAG is the most common enterprise AI pattern, so it's the most likely thing a QA role will test.
Interviewers want to see that I can break it into stages and name what fails in each.

## 3. 30-second answer

"RAG has two phases. Ingestion: load documents, split them into chunks, embed them, store them in a
vector database with metadata. Query time: embed the question, retrieve the top-k similar chunks,
optionally rerank, build a prompt with those chunks, and have the LLM answer, ideally with citations.
I test each stage separately — chunking, retrieval, prompt, generation — because a bad answer can come
from retrieving the wrong thing or from the model ignoring the right thing."

## 4. 2-minute answer

**Phase A — Ingestion (offline)**

1. **Load** documents (PDF, wiki, tickets, DB rows).
2. **Clean** (strip headers/footers, fix encoding).
3. **Chunk** into pieces (e.g. a few hundred tokens, often with overlap) so each is small enough to
   retrieve precisely but big enough to make sense.
4. **Embed** each chunk into a vector.
5. **Store** vectors + text + metadata (source, section, date, access level) in a vector database.

**Phase B — Query (online)**

1. User asks a question.
2. **Embed** the question.
3. **Retrieve** top-k nearest chunks (k is a setting). Optional: hybrid search (keyword + vector),
   metadata filters (only documents this user may see), **reranking** to reorder candidates.
4. **Build the prompt:** system rules + retrieved chunks + question + "answer only from the context; if
   not found, say you don't know."
5. **Generate** the answer, ideally with citations.
6. Guardrails/output checks, logging.

**Failure modes by stage — this is the test design:**

| Stage | What goes wrong | How I'd test |
|---|---|---|
| Ingestion | Parsing drops tables/headers; duplicates; stale or deleted docs still indexed | Ingest known docs, then query facts inside tables; delete a doc and confirm it stops being cited |
| Chunking | Chunk splits a fact across boundaries; chunks too big (noisy) or too small (no context) | Questions whose answers sit at chunk boundaries |
| Embedding | Model doesn't capture domain terms/abbreviations; language mismatch | Synonym and acronym queries |
| Retrieval | Right doc exists but isn't retrieved (low recall); irrelevant chunks outrank right ones (low precision); k too small/large | Known question→source pairs; measure recall@k |
| Ranking | Correct chunk retrieved but buried ("lost in the middle") | Check rank position of the right chunk |
| Prompt | Context pushed past the window and truncated; instructions weak | Long-context queries; context overflow |
| Generation | Model ignores context, uses its own memory, or invents details (hallucination); doesn't say "I don't know" | Unanswerable questions; compare claims with chunks |
| Access control | User sees answers built from documents they shouldn't access | Test with two users with different permissions |
| Freshness | Index not updated after document change | Update a doc, re-query |
| Security | Malicious text in a document tells the model what to do (indirect prompt injection) | Plant an instruction in a test doc |
| Cost/latency | Too many chunks, slow reranker, long prompts | Measure latency and tokens per query |

**Key insight:** when the answer is wrong, first ask *was the right information retrieved?* If no →
retrieval problem. If yes but the answer is still wrong → generation/prompt problem. Separating
the two is what makes RAG testable.

## 5. Real project example (honest bridge)

I haven't tested a production RAG system. How I'd approach one: ingestion and metadata are
data-quality problems I'd check with SQL/API-style verification (what's actually in the store);
permissions are authorisation testing, which I know from API testing; latency under load is a JMeter
exercise; the evaluation set (note 04) is the new part. The structure — verify each stage's input and
output — is the same discipline as testing a pipeline in CIP_LEGACY (high-volume data validation).

## 6. Common interview Q&A

**Q: Explain RAG.** Retrieve relevant private documents at query time and give them to the LLM so it
answers from them, reducing hallucination and avoiding retraining.

**Q: Where can a RAG system fail?** Ingestion/parsing, chunking, embeddings, retrieval, ranking,
prompt/context limits, generation faithfulness, permissions, freshness, injection, cost.

**Q: Wrong answer — how do you debug?** Check retrieved chunks first. If the right chunk is absent,
it's retrieval/ingestion. If present but answer wrong, it's prompt/generation.

**Q: What's chunking and why does it matter?** Splitting documents into retrievable pieces; size and
overlap trade off precision against context.

**Q: What is reranking?** A second step that reorders retrieved candidates by relevance to the query.

**Q: How would you test access control in RAG?** Same query as users with different permissions;
ensure no content from restricted documents appears in the answer or citations.

## 7. Weak areas to watch

- Treating RAG as "just a chatbot" and only testing final answers.
- Ignoring permissions and data freshness.
- Not knowing the retrieval-vs-generation split.
- Claiming a RAG project I haven't done.

## 8. Key takeaway

A bad RAG answer is either "wrong things retrieved" or "right things ignored." Test retrieval and
generation separately, and don't forget permissions, freshness and injection.
