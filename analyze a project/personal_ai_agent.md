# Personal AI Agent — Architecture Analysis

## Overview

This project is a classic **single-agent + vector-store RAG** system — the most fundamental architecture pattern in production AI products. A user submits a question, the agent classifies and routes it, retrieves relevant context from an embedding-based vector database, and synthesizes a final answer. Simple in principle; the interesting engineering is in the details.

---

## Architecture Breakdown

### 1. Query Classification Layer

Before touching the knowledge base, the agent classifies every incoming question into one of three categories:

| Class | Routing behavior |
|---|---|
| **Relevant** | Retrieve from vector store, generate answer |
| **Irrelevant** | Reject immediately, no retrieval |
| **Follow-up** | Resolve against prior context, then retrieve |

This classification gate is a deliberate cost-control and quality-control decision. Sending every message straight to the retriever would increase noise and waste tokens on out-of-scope queries.

---

### 2. Context Management — Design and Trade-offs

The follow-up handling is the most architecturally interesting piece. The current design maintains a **list** that accumulates relevant questions and follow-ups. When a follow-up arrives, the agent scans the list **back-to-front** to locate the most recent relevant question and injects it into the prompt.

**Critique of the current design:**

The list traversal is largely redundant. A simpler and more efficient structure would be:

- Store only the last relevant question and its retrieved context in a single variable.
- On a new relevant question, overwrite it.
- On a follow-up, read directly from that variable — no traversal needed.

The underlying motivation for keeping context minimal is sound: the project runs on **GPT-4o mini**, and in testing, even moderately long context caused the model to start ignoring the rule prompt. Keeping the context window lean is a practical necessity given the model's sensitivity to prompt length.

---

### 3. Vector Knowledge Base

The knowledge base is built from roughly ten markdown source documents. These documents are embedded (vectorized) and stored in a vector database; at query time, the agent performs a similarity search against the embeddings to retrieve the most relevant chunks.

**What works:**
- Recall rate in testing was surprisingly high for a loosely structured corpus.
- Markdown is easy to maintain and extend.

**Known weaknesses:**
- **No taxonomy.** Content was added organically without a deliberate categorization scheme. A well-designed embedding corpus should have clear topic boundaries that align with how documents are chunked before embedding — otherwise chunk boundaries can cut across semantically distinct ideas and hurt retrieval precision.
- **No category metadata in the source documents.** Classification information that exists at the application layer is not reflected in the documents, which limits how much the retriever can filter or weight results.
- **Corpus is too small.** Coverage gaps are a hard ceiling on what the agent can answer, regardless of retrieval quality.

**Note for future iteration:** HTML has recently emerged as a viable source format for vector stores — semantic tags and document structure can inform better chunking before embedding. Worth experimenting with in a future version.

---

### 4. Answer Generation Layer

The final step feeds three inputs into the LLM:

1. Chunks retrieved from the vector database via embedding similarity search
2. The original user question
3. A rule prompt (system instructions governing tone, format, and behavior)

The rule prompt itself was validated in testing and is not the problem. The observed failures — rule prompt being ignored or overridden — were traced back to context length pressure, not prompt quality. This is a known GPT-4o mini limitation: as the context window fills up, instruction-following degrades. The context minimization strategy described in §2 is a direct engineering response to this constraint.

---

## What This Project Does Well

- **Near-zero hallucination.** Answers are grounded in retrieved documents, and irrelevant queries are rejected outright — the model is rarely in a position to fabricate.
- **Solid recall rate.** Despite the unstructured source corpus, embedding-based retrieval performs better than the corpus quality would suggest.
- **Clean failure mode.** Out-of-scope queries are rejected explicitly rather than producing a confident but wrong answer.

---

## Summary

As a first AI product, this project demonstrates a solid grasp of the core RAG loop and the real-world constraints of deploying a smaller LLM in a conversational context. The main areas for improvement are source document structure (better taxonomy before embedding), simplifying the context management logic, and eventually a model upgrade — which would relax most of the prompt-length constraints the current design works around.
