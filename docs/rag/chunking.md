---
last_verified: 2026-06-12
gardener_sources:
  - https://docs.llamaindex.ai/en/stable/module_guides/loading/node_parsers/
  - https://python.langchain.com/docs/concepts/text_splitters/
  - https://github.com/chroma-core/chroma
---

# Chunking Strategies

## What & why

A RAG pipeline retrieves pieces of documents, not whole documents. **Chunking** is the
process of splitting source material into those pieces before indexing. The chunks
become the unit of retrieval: when a query comes in, the system returns the
top-k most similar chunks, not pages or files.

Chunking feels like plumbing — a detail you configure once and forget. In practice,
**chunking is where RAG quality lives.** A retrieval system with a good embedding
model and a bad chunking strategy will consistently return irrelevant or incomplete
context. Many RAG systems that "don't work" are suffering from a chunking problem, not
an embedding or model problem.

The core tension: large chunks preserve context but are less precise — a retrieved
chunk might contain the relevant sentence plus a lot of noise. Small chunks are
precise but may split the information across chunks, so the retrieved piece is
incomplete without its neighbors.

## How it actually works

### Fixed-size chunking

Split every N characters (or tokens), with an optional overlap. The simplest
approach; easy to reason about costs and counts.

**Strengths:** Predictable chunk sizes, easy to implement, no document structure
assumptions.

**Weaknesses:** Splits on arbitrary boundaries. A sentence may be cut in half. A
paragraph that spans the boundary of two chunks requires both to be retrieved. For
prose, the results can be incoherent.

Use when: document structure is unknown or inconsistent, you need a baseline to
benchmark against, or the content is short enough that mid-sentence splits are rare.

### Recursive character splitting

Split on a priority-ordered list of separators — typically `\n\n`, then `\n`, then
`. `, then ` `. Recurse into each piece until it is under the target size. This
preserves paragraph and sentence boundaries where they exist, falling back to
character-level splits only when necessary.

This is the most common default in RAG frameworks and a good starting point for
general prose.

### Semantic chunking

Instead of splitting on structure or size, embed candidate boundaries and split where
the embedding similarity *drops* between adjacent sentences — i.e., where the topic
shifts. The result: chunks that correspond to coherent topics rather than arbitrary
lengths.

**Strengths:** Chunks often contain exactly one idea; retrieval is more precise.
**Weaknesses:** Requires running an embedding model at indexing time, making it more
expensive and complex. Chunk sizes vary, which makes cost estimation harder.

Use when: documents have dense topic changes (e.g., reference documentation, reports
with distinct sections), and you have already confirmed that simpler strategies are
the bottleneck.

### Structure-aware (document-type) chunking

Use the document's native structure as split points: Markdown headers (`##`), HTML
tags, PDF bounding boxes, code function boundaries.

| Document type | Natural split point |
|---|---|
| Markdown / docs | Headers (`#`, `##`) |
| HTML | `<section>`, `<article>`, `<p>` |
| Code | Function/class definitions |
| PDF with structure | Page, section detected via layout |
| Tables | Keep each table as one chunk |

This is often the highest-quality approach for structured content, because the splits
are meaningful by definition. It requires knowing the document type in advance and
handling each format explicitly.

## Key parameters and their trade-offs

### Chunk size

| Smaller chunks | Larger chunks |
|---|---|
| More precise retrieval | More context per chunk |
| More chunks — higher storage + index cost | Fewer chunks — lower index cost |
| May miss context that spans a boundary | May return irrelevant material alongside the target |
| Better for narrow factual queries | Better for questions requiring synthesis across a passage |

A practical starting range is 256–1024 tokens. Benchmark on your actual query types —
the right size is task-dependent.

### Overlap

Overlap repeats a portion of each chunk at the start of the next. It ensures that a
sentence split across a boundary is fully present in at least one chunk.

Typical overlap: 10–20% of chunk size. More than 20% inflates your index with near-
duplicate content and can confuse reranking. Zero overlap is rarely a good idea for
prose.

### Metadata

Chunks do not have to stand alone. Every chunk should carry metadata: source document
ID, page or section number, document title, creation date. At retrieval time, metadata
enables filtering ("only chunks from documents created after this date") and makes
generated citations accurate. Metadata is cheap to attach at indexing time and very
expensive to add retroactively.

## The "chunking is where RAG quality lives" argument

Embedding models and vector databases are largely interchangeable — the differences
between reasonable choices are small. Chunking is not. A retrieval system returns
*chunks*. If the chunks are the wrong size, contain mixed topics, or cut across natural
semantic units, no amount of embedding quality or reranking can recover the information
that was never in any single chunk to begin with.

Before you swap your embedding model or rebuild your vector index, check:

1. Are the retrieved chunks actually relevant when you read them as a human?
2. Is the answer to the query *present* in at least one of the top-k chunks?
3. If the answer spans multiple chunks, are those chunks typically co-retrieved?

If (2) is false, the problem is chunking (or the source document doesn't contain the
answer). If (1) and (3) are false with (2) true, the problem is retrieval or reranking.

## Interview questions

After this page you should be able to answer:

1. Why does chunking matter more than people expect? What breaks when you get it wrong?
2. Compare fixed-size and recursive chunking. Under what circumstances does each perform better?
3. A team is building RAG over a large code repository. What chunking strategy would you recommend, and why?
4. You are evaluating a RAG system and find that retrieved chunks are individually relevant but the LLM's answer is still wrong. Where do you look next?
5. What is overlap in chunking, and what goes wrong if you set it too high or too low?

---

*Last verified 2026-06-12 · maintained by the Gardener*
