# Similarity Metrics

Three common ways to compare embeddings:

1. **Cosine similarity** — most widely used
2. **Dot product** — fast, common in vector databases
3. **Euclidean distance** — less common for text embeddings

---

# 1. Cosine Similarity

## Concept

Measures the **angle** between two vectors, ignoring their magnitude.

Returns a value between -1 and 1.

1.0 → identical meaning\
0.0 → unrelated\
-1.0 → opposite meaning

## Example

```
similarity("restart the server", "reboot the machine") → 0.94
similarity("restart the server", "best pizza in Colombo") → 0.11
```

## Implementation

```python
import numpy as np

def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))
```

---

# 2. Dot Product

## Concept

Multiplies corresponding values and sums them.

If vectors are **normalized** (length = 1), dot product gives the
**same result as cosine similarity**.

Most embedding models return normalized vectors. This is why many
vector databases default to dot product — it's faster to compute
(skips the division step).

## When It Differs From Cosine

When vectors are **not** normalized, dot product is affected by
magnitude. A longer vector scores higher regardless of meaning.

For normalized embeddings (most common case), cosine and dot product
are interchangeable.

---

# 3. Euclidean Distance

## Concept

Measures the **straight-line distance** between two points in vector
space.

Smaller distance → more similar.

## Why It's Less Common

Euclidean distance is sensitive to vector magnitude. Two vectors with
the same direction but different lengths will have a large euclidean
distance even though they represent similar meaning.

Cosine similarity handles this better for text embeddings.

## When It's Used

Some vector databases offer it as an option. Useful when magnitude
carries meaningful information (e.g., frequency or importance
weighting), which is uncommon for standard text embeddings.

---

# Which Metric To Use

**Default choice: cosine similarity.** Works reliably for text
embeddings.

If your embedding model returns normalized vectors (most do), **dot
product** gives identical results and is faster.

**Euclidean distance** is rarely the best choice for text, but worth
knowing it exists.

---

# Vector Search

## Concept

Vector search finds the **most similar vectors** in a collection.

Instead of matching keywords in a database, you:

1. Convert the query to an embedding
2. Compare it against all stored embeddings
3. Return the closest matches

## Pipeline

User Query → Embedding Model → Query Vector → Compare against stored
vectors → Return top-K results

## Example

Stored documents (already embedded):

Doc 1: "Setting up Kafka consumers in NestJS"\
Doc 2: "PostgreSQL indexing strategies"\
Doc 3: "Debugging message queue lag"

User query: "My Kafka consumer is slow"

Query embedding is closest to Doc 1 and Doc 3.\
Those are returned as search results.

---

# Why Not Just Use LIKE or Full-Text Search?

Keyword search:

"deploy failure" matches "deploy failure" but not "broken release
pipeline"

Full-text search:

Better than LIKE. Handles stemming and ranking. But still
fundamentally word-based.

Semantic search:

Matches by **meaning**. "deploy failure" finds "broken release
pipeline" because the vectors are close.

In production, many systems **combine both**:

Keyword search for exact matches\
Semantic search for meaning-based matches\
This combination is called **hybrid search**.

---
