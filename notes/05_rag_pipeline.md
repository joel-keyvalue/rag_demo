# RAG Pipelines

## Why RAG Exists

LLMs have a **knowledge cutoff**. They don't know about your company's
internal docs, recent data, or private codebases.

Three problems RAG solves:

**Grounding** — connect the model to real, verifiable sources\
**Accuracy** — reduce hallucinations by giving the model actual data\
**Freshness** — provide up-to-date information the model was never
trained on

## The Alternative Without RAG

Without RAG, you either:

Fine-tune the model (expensive, slow to update)\
Stuff everything into the prompt (hits context limits, costs scale)\
Accept that the model guesses (hallucinations)

RAG is the practical middle ground for most production applications.

---

# The Full RAG Pipeline

The pipeline has two phases: **pre-processing** (before any user
asks anything) and **query-time** (when a user asks a question).

## Pre-Processing Phase (before user queries)

Documents → Chunking → Embedding → Vector Store

1. **Document ingestion** — load source material (PDFs, docs, code,
   wikis, databases)
2. **Chunking** — split documents into smaller pieces
3. **Embedding** — convert each chunk into a vector
4. **Indexing** — store vectors in a vector database with metadata

## Query-Time Phase (when a user asks a question)

User Query → Embedding → Vector Search → Retrieved Chunks →
Prompt Assembly → LLM → Response

1. **Query embedding** — convert user question to a vector
2. **Retrieval** — find the most similar chunks from the vector store
3. **Augmentation** — inject retrieved chunks into the prompt
4. **Generation** — LLM generates a response grounded in the retrieved
   context

---

# Document Ingestion

## What Gets Ingested

Internal documentation\
Knowledge base articles\
API specifications\
Code repositories\
Meeting notes, tickets, wikis\
PDF reports, spreadsheets

## Practical Considerations

Documents change. The ingestion pipeline needs to handle **updates**
and **deletions**, not just initial loading.

Most teams run ingestion as a **batch job** (nightly or on-change),
not in real time.

---

# Chunking Strategies

This is **where most RAG pipelines break**.

Chunking determines how documents are split before embedding. Bad
chunking leads to bad retrieval, which leads to bad answers.

## 1. Fixed-Size Chunking

Split text every N characters or N tokens.

Example:

Document: 5000 tokens\
Chunk size: 500 tokens\
Result: 10 chunks

**Problem**: Chunks may split mid-sentence or mid-paragraph. Context
is lost at boundaries.

**Mitigation**: Add **overlap**. Each chunk includes the last 50-100
tokens of the previous chunk.

```
Chunk 1: tokens 0-500
Chunk 2: tokens 450-950  (50 token overlap)
Chunk 3: tokens 900-1400
```

## 2. Semantic Chunking

Split based on **meaning boundaries** rather than character count.

Techniques:

Split on paragraph breaks\
Split on section headers\
Use an LLM or embedding model to detect topic shifts

**Advantage**: Chunks are more coherent.\
**Disadvantage**: Chunk sizes vary. Some may be very small or very
large.

## 3. Hierarchical Chunking

Maintain **parent-child relationships** between chunks.

Example:

Parent: Full section "Authentication Architecture"\
Children: Individual paragraphs within that section

At retrieval time, if a child chunk matches, the system can also pull
in the parent for broader context.

**Advantage**: Provides both precision and context.\
**Disadvantage**: More complex to implement and store.

## Which Strategy To Use

**Start with fixed-size chunking with overlap**. It works for most
cases and is simple to implement.

Move to semantic or hierarchical chunking when retrieval quality is
the bottleneck.

---

# Chunk Size Matters

Too small (50-100 tokens):

Chunks lack context\
Retrieval returns fragments that don't make sense alone

Too large (2000+ tokens):

Chunks contain mixed topics\
Retrieval is less precise\
More tokens used per retrieval

**Sweet spot for most applications: 200-800 tokens with 10-20%
overlap.**

---

# Embedding the Chunks

Each chunk is passed through an embedding model to produce a vector.

```
Chunk text → Embedding Model → Vector [0.12, -0.45, ...]
```

The vector and the original chunk text are stored together.

Typical storage structure:

```
{
    "id": "doc-123-chunk-4",
    "text": "Authentication uses JWT tokens with...",
    "embedding": [0.12, -0.45, ...],
    "metadata": {
        "source": "auth-docs.md",
        "section": "Authentication",
        "page": 4
    }
}
```

**Metadata** is critical. It enables filtering at query time (e.g.,
"only search documentation from the auth service").

---

# Retrieval

## Basic Retrieval

User query is embedded, and the top-K most similar chunks are
returned.

Typical K values: 3-10 chunks.

## Retrieval With Filtering

Combine vector similarity with metadata filters.

Example:

Vector search: top 20 most similar chunks\
Filter: only chunks from "backend-docs"\
Result: top 5 after filtering

## Reranking

Initial vector search returns candidates. A **reranker model** scores
them more carefully.

Pipeline:

Vector search → Top 20 candidates → Reranker → Top 5

Rerankers (like Cohere Rerank or cross-encoder models) are slower but
more accurate than pure vector similarity.

This is a common production optimization.

---

# Augmentation — Building the Prompt

Retrieved chunks are inserted into the prompt alongside the user
query.

Example prompt structure:

```
System: You are a helpful assistant. Answer based on the provided
context. If the context doesn't contain the answer, say so.

Context:
[Chunk 1 text]
[Chunk 2 text]
[Chunk 3 text]

User: How does our authentication system handle token refresh?
```

## Important Design Decisions

**Tell the model to use the context**. Without explicit instructions,
the model may ignore retrieved chunks and answer from training data.

**Tell the model what to do when context is insufficient**. Otherwise
it will hallucinate an answer.

**Order matters**. Place the most relevant chunks first, or closest to
the user query (recency bias in attention).
