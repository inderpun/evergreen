# RAG

**Retrieval-Augmented Generation — grounding LLM outputs in your actual data.**

RAG is the pattern that makes LLMs useful for domain-specific work: instead of asking
a model to remember facts it may not have seen (or that postdate its training), you
retrieve the relevant information at inference time and include it in the prompt. The
model becomes a reasoning engine over *your* data rather than a leaky memory.

## What this section covers

| Page | Core idea |
|---|---|
| [Vector databases](vector-databases.md) | The stores that make similarity search fast at scale |
| [Chunking strategies](chunking.md) | How to split documents so retrieval actually finds the right piece |
| [Retrieval & reranking](retrieval-reranking.md) | Dense search, sparse search, hybrid approaches, and the reranking layer |
| [RAG evaluation](rag-evals.md) | How to measure whether your RAG pipeline is actually working |

## Suggested reading order

Vector databases → Chunking → Retrieval & reranking → RAG evaluation. The first two
are "how to build"; the last two are "how to know if it's working." The chunking page
is often the highest-leverage place to improve a struggling RAG system.

## After this section you can...

- Design a RAG pipeline from scratch and explain every component
- Diagnose whether a RAG failure is a retrieval problem or a generation problem
- Choose an appropriate chunking strategy for a given document type
- Know when a vector database is the wrong choice
- Run a structured eval on a RAG system and produce a number you can track over time

---

*Section maintained by [Inderpuneet Singh](https://github.com/inderpun) and the Gardener agent.*
