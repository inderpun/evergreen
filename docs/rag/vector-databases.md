---
last_verified: 2026-06-11
gardener_sources:
  - https://github.com/pgvector/pgvector/releases
  - https://docs.pinecone.io/release-notes
  - https://qdrant.tech/blog/
---

# Vector Databases

## What & why

An embedding model turns text (or images, audio…) into a vector — a list of ~300 to
~3000 numbers — positioned so that *similar meanings land near each other*. A vector
database stores millions of these vectors and answers one question fast: **"which
stored items are nearest to this query vector?"** That single capability (similarity
search) is the retrieval engine behind RAG, semantic search, recommendation, and
deduplication.

Why a dedicated database at all? Because exact nearest-neighbor search over millions
of high-dimensional vectors means comparing against *every* vector — O(N) per query.
Vector DBs exist to make this **approximate but fast** (ANN: approximate nearest
neighbor), trading a sliver of recall for orders-of-magnitude speedups.

## How it actually works

Three load-bearing ideas:

1. **ANN indexes.** The dominant one is **HNSW** (Hierarchical Navigable Small
   World): a layered graph where search hops from a sparse top layer down to dense
   lower layers, like an express-train system. Alternatives: **IVF** (cluster the
   space, search only nearby clusters — what FAISS popularized) and **DiskANN**
   (graph index optimized for SSD, for corpora bigger than RAM).
2. **Distance metric.** Cosine similarity ≈ dot product on normalized vectors;
   match the metric your embedding model was trained for (almost always cosine).
3. **Filtered search.** Real queries are "nearest neighbors *where* tenant_id=X and
   date>Y". Combining metadata filters with ANN efficiently is where engines
   genuinely differ — naive pre-filtering can destroy the index's assumptions.

## Choosing one (the practical table)

| You are… | Reach for | Why |
|---|---|---|
| already on Postgres | **pgvector** | no new infra; HNSW since 0.5; fine to ~10M vectors |
| prototyping locally | **Chroma / LanceDB / FAISS** | embedded, zero ops |
| wanting managed + serverless | **Pinecone** | the managed pioneer; pay-per-use |
| self-hosting at scale | **Qdrant / Milvus / Weaviate** | open-source, filtering + hybrid search |
| already on OpenSearch/Elastic | their k-NN / vector features | one less system |

The honest 2026 take: **the vector DB is rarely your bottleneck or your moat.**
Retrieval *quality* lives in chunking, embeddings, hybrid (dense+keyword) search, and
reranking — see [Retrieval & reranking](retrieval-reranking.md). Default to the store
you already operate; reach for a specialist engine when scale or filtering demands it.

## Interview questions

After this page you should be able to answer:

1. Why is exact nearest-neighbor search impractical at scale, and what does "approximate" buy you?
2. Sketch how HNSW works. Why is it described as "navigable small world"?
3. When would pgvector be the *wrong* choice?
4. Your RAG system retrieves irrelevant chunks. Is the vector DB the first thing you'd swap? What is?
5. How do metadata filters interact badly with ANN indexes?

---

*Last verified 2026-06-11 · maintained by the Gardener*
