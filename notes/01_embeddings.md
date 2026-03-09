# Embeddings & Semantic Search

## Why This Matters

Traditional search is **keyword-based**. It matches exact words.

LLM-powered applications need **meaning-based** search. Two sentences
can mean the same thing with completely different words.

Example:

Query: "How do I fix a broken deployment?"\
Relevant document: "Troubleshooting failed CI/CD pipelines"

Keyword search fails here. Semantic search finds it.

---

# What Are Embeddings?

## Concept

An embedding is a **list of numbers** (a vector) that represents the
**meaning** of a piece of text.

Example:

"Kubernetes pod crashed" → [0.12, -0.45, 0.78, ..., 0.33]

The model reads the text and compresses its meaning into a
high-dimensional vector (typically 768 to 3072 dimensions).

## Key Property

Texts with **similar meaning** produce vectors that are **close
together** in vector space.

Example:

"restart the server" → [0.82, 0.15, ...]\
"reboot the machine" → [0.80, 0.17, ...]\
"best pizza in Colombo" → [-0.45, 0.92, ...]

The first two vectors are close. The third is far away.

---

# How Embeddings Are Generated

An **embedding model** converts text into vectors.

This is **not** the same model used for chat or generation.

Common embedding models:

OpenAI text-embedding-3-small (1536 dimensions)\
OpenAI text-embedding-3-large (3072 dimensions)\
Cohere embed-v3\
Open-source: sentence-transformers, nomic-embed

## Pipeline

Text → Embedding Model → Vector (list of floats)

Example using OpenAI:

```python
from openai import OpenAI

client = OpenAI()

response = client.embeddings.create(
    model="text-embedding-3-small",
    input="Kubernetes pod crash loop"
)

vector = response.data[0].embedding
# [0.012, -0.034, 0.056, ...]  (1536 numbers)
```

---
