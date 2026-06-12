---
last_verified: 2026-06-12
gardener_sources:
  - https://huggingface.co/blog/mteb
  - https://www.sbert.net/docs/pretrained_models.html
  - https://github.com/embeddings-benchmark/mteb
---

# Embeddings

## What & why

An **embedding** is a dense vector — a list of floating-point numbers — that
represents the meaning of a piece of text. The key property: *similar meanings land
close together in the vector space*. "dog" and "puppy" are near each other; "dog"
and "treaty" are far apart.

This turns a fuzzy semantic question ("is this chunk relevant to that query?") into a
geometric one ("how close are these two vectors?"), which computers can answer cheaply
at scale. That is the foundation of semantic search, RAG retrieval, deduplication,
and clustering of unstructured text.

## How it actually works

### Training intuition

Embedding models are typically trained with **contrastive learning**. The setup:

1. Collect pairs of texts that *should* be close (a question and its answer, two
   paraphrases, a title and its abstract).
2. Also collect pairs that *should* be far (random negatives, hard negatives chosen
   because they look superficially similar but differ in meaning).
3. Train the model with a loss that pulls positive pairs together and pushes
   negative pairs apart in the vector space.

The result is a model where position in the vector space encodes semantic
relationships it has been trained to distinguish. Models trained on general corpora
learn general-purpose similarity; models trained on domain-specific pairs (e.g.,
code search, medical literature) learn domain-specific similarity.

### Architecture

Most embedding models are encoder-only transformers (BERT-family). The entire
sequence passes through the encoder; the output is pooled — often by averaging the
token representations (mean pooling) or by taking the `[CLS]` token — to produce a
single fixed-size vector.

This is distinct from a **generative LLM** (decoder-only). A generative LLM *could*
produce an embedding, but its architecture is optimized for next-token prediction,
not for producing geometrically meaningful similarity scores. Use a dedicated
embedding model for retrieval tasks; don't embed with your generative model unless
you have measured that it works better.

### Dimensionality

Embedding vectors range from ~384 to ~3072 dimensions in common models. Higher
dimensionality lets the model express more nuanced distinctions, at the cost of
storage and compute. For most RAG use cases, mid-range dimensions (768–1536) are
sufficient. The right choice is empirical: run your task's evaluation, not a
benchmark that doesn't match your distribution.

## Similarity metrics

| Metric | Definition | When to use |
|---|---|---|
| **Cosine similarity** | Dot product of unit-normalized vectors | Most embedding models are trained for this; use it by default |
| **Dot product** | Raw inner product | Equivalent to cosine on normalized vectors; some models return un-normalized embeddings |
| **L2 (Euclidean) distance** | Geometric distance in vector space | Less common for text; sometimes used in image embeddings |

**Match the metric to the model.** A model trained with cosine similarity as its
objective will not behave as intended if you query it with L2 distance. This is
stated in model cards; read them.

### Bi-encoders vs. cross-encoders

| | Bi-encoder | Cross-encoder |
|---|---|---|
| How it works | Encodes query and document independently; compare vectors | Encodes query + document *together*; outputs a relevance score |
| Speed | Fast — pre-encode all documents, O(1) lookup | Slow — must run inference on every (query, document) pair |
| Quality | Good first-pass retrieval | Higher accuracy for reranking |
| Typical role | Retrieval (generate candidates) | Reranking (score top-k candidates) |

The standard RAG retrieval pattern uses a bi-encoder for first-pass recall, then a
cross-encoder to rerank the top candidates before passing them to the LLM.

## Interview questions

After this page you should be able to answer:

1. What does "similar meanings land near each other" actually mean geometrically? What is the model learning during training?
2. Why should you use a dedicated embedding model rather than your generative LLM for retrieval?
3. You have a RAG system that is slow to return results. The bottleneck appears to be the retrieval step. Embeddings are pre-computed. What else might be slowing things down?
4. A team trained a general-purpose embedding model on Wikipedia. You are building a medical document search system. What do you expect will go wrong, and what is the right fix?
5. Explain the difference between a bi-encoder and a cross-encoder. When would you use each?

---

*Last verified 2026-06-12 · maintained by the Gardener*
