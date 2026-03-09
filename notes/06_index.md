# Indexes and Retrieval Methods — Group Study Notes

**Follows from:** [05 RAG Pipeline](05_rag_pipeline) — after chunking and embedding, we need to decide **how to store and search** (lexical vs vector, and which vector index).

---

# Where This Fits in the Pipeline

From the RAG pipeline:

- **Pre-processing:** Documents → Chunking → **Embedding** → **Index / Store**
- **Query-time:** Query → **Embedding** (optional for lexical) → **Retrieval** → Chunks → Prompt → LLM

This session is about **retrieval and indexing**:

1. **Lexical (keyword) retrieval** — TF-IDF, BM25 — no embeddings, match on terms.
2. **Vector retrieval** — similarity over embeddings; choice of **index type**: flat (exact) vs HNSW (approximate).

---

# Two Families of Retrieval

|                             | **Lexical / Sparse**                | **Vector / Dense**                        |
| --------------------------- | ----------------------------------- | ----------------------------------------- |
| **Representation**          | Bag of words / term weights         | Embedding vector                          |
| **Matching**                | Keyword overlap, TF-IDF, BM25       | Similarity (cosine, dot product)          |
| **Synonyms / paraphrasing** | Weak (needs exact or stemmed terms) | Strong (semantic similarity)              |
| **Typo / spelling**         | Brittle                             | More robust (if embedding normalizes)     |
| **Typical use**             | Classic search, hybrid RAG          | Default in “embedding + vector store” RAG |

In RAG, **hybrid search** = run both (e.g. BM25 + vector), then merge or rerank results.

---

# TF-IDF (Term Frequency – Inverse Document Frequency)

## Idea

- **TF (Term Frequency):** How often a term appears in a **document** → more occurrences → higher weight for that document.
- **IDF (Inverse Document Frequency):** How **rare** the term is across all documents → rare terms are more discriminative, so they get higher weight.

So: weight = “how much this term appears here” × “how distinctive this term is in the corpus”.

## Intuition

- Word like “the” appears everywhere → low IDF → low contribution.
- Word like “Kafka” in a doc about Kafka → high TF there, and high IDF if few docs mention it → strong signal.

## Formula (conceptually)

- **TF(t, d):** term frequency of term *t* in document *d*
- **IDF(t)** = log( N / df(t) ), where *N* = number of docs, df(*t*) = number of docs containing *t*
- **TF-IDF(t, d)** = TF(t, d) × IDF(t)

Documents are represented as **sparse vectors** (one dimension per term). Query is also a vector; similarity = e.g. cosine between query vector and document vectors.

## Pros and cons (for RAG)

- **Pros:** No embedding model, fast, interpretable (you see which terms matched).
- **Cons:** No semantic understanding; synonyms and paraphrases don’t match unless you add synonyms or use stemming.

---

# BM25 (Best Matching 25)

## Idea

BM25 is a **refinement of TF-IDF** used in most modern text search engines (Elasticsearch, OpenSearch, Lucene, etc.).

Main improvements over plain TF-IDF:

1. **Saturation of term frequency** — TF doesn’t grow without bound; repeated terms have diminishing gains (avoids one long doc dominating just by repeating a word).
2. **Document length normalization** — long documents don’t get an unfair advantage; scoring accounts for document length.
3. **Tunable parameters** — e.g. *k*₁, *b* for TF and length normalization.

## Intuition

- “Relevant” means the term is important in the doc, not that the doc is just long or repetitive.
- BM25 formalizes that with a formula that caps the effect of raw term count and normalizes by document length.

## Role in RAG

- **Sparse retrieval:** BM25 is the standard lexical retriever (often used in **hybrid** with a vector retriever).
- **When it helps:** Exact names, IDs, jargon, phrases that appear verbatim in the corpus.
- **When it’s weak:** Conceptual questions, paraphrasing, multi-lingual or “meaning” queries — here dense (vector) retrieval usually does better.

---

# Flat Index (Vector)

## What it is

- Store **every** vector (e.g. one per chunk).
- At query time: embed the query, then compare the query vector to **all** stored vectors (e.g. cosine or dot product), and return the top-K.

No approximate or hierarchical structure — **brute-force exact search**.

## Pros and cons

| **Pros**                             | **Cons**                                  |
| ------------------------------------ | ----------------------------------------- |
| Exact — no approximation error       | Slow for large N (O(N) per query)         |
| Simple to implement and reason about | Memory: must load all vectors (or stream) |
| No index-building hyperparameters    | Not scalable to millions of vectors       |

## When to use

- **Small corpora** (e.g. &lt; 10k–100k chunks).
- Prototyping, debugging, or when you need **guaranteed exact** top-K.
- When latency isn’t critical.

---

# HNSW Index (Hierarchical Navigable Small World)

## What it is

- **Approximate Nearest Neighbor (ANN)** index for vectors.
- Graph-based: vectors are nodes; edges connect “nearby” vectors. The graph is **hierarchical** (multiple layers): bottom layer = all nodes, upper layers = fewer nodes, used for fast “skip” navigation.
- Search: start at top layer, move to neighbor that gets you closer to the query vector, drop to lower layer, repeat until bottom → return best neighbors.

## Intuition

- Like a “skip list” over vectors: you don’t compare to everyone, you follow links through the graph to get close to the query quickly, then refine.

## Pros and cons

| **Pros**                                                    | **Cons**                                 |
| ----------------------------------------------------------- | ---------------------------------------- |
| Fast query time (sublinear in N in practice)                | Approximate — might miss some true top-K |
| Scales to millions/billions of vectors                      | Index build time and memory              |
| Good recall with proper tuning (ef_construction, ef_search) | Parameters affect speed vs accuracy      |

## When to use

- **Large vector stores** (hundreds of thousands of vectors and up).
- Production RAG where latency and scale matter.
- When “good enough” top-K recall is acceptable (often 95%+ with tuned HNSW).

## Parameters (for discussion)

- **ef_construction** — higher → better index quality, slower build.
- **ef_search** (or M) — higher → better recall at query time, slower queries.
- **m** (max edges per node) — affects connectivity and memory.

---

# How They Relate — Summary

- **TF-IDF / BM25:** **Lexical** retrieval (sparse, keyword-style). Use for exact terms, names, IDs; often in **hybrid** with vector retrieval in RAG.
- **Flat index:** **Exact** vector search over all chunks. Use for small scale and when you need exactness.
- **HNSW:** **Approximate** vector search. Use for large-scale, low-latency RAG.

In a typical RAG stack after [05 RAG Pipeline](05_rag_pipeline):

- Chunk → embed → store in a **vector index** (flat for small, HNSW for large).
- Optionally add **BM25** (or TF-IDF) and do **hybrid retrieval** (BM25 + vector), then merge or rerank before passing chunks to the LLM.
