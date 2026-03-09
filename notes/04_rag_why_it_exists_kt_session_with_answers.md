# Why RAG Exists --- Grounding, Accuracy, and Reducing Hallucinations



------------------------------------------------------------------------

# Session Objective

Understand:

-   Why **Retrieval Augmented Generation (RAG)** became the dominant
    architecture for LLM systems
-   The **three core problems RAG solves**
-   Why alternatives like **fine-tuning or prompt stuffing alone are
    insufficient**
-   Why RAG is considered the **production-friendly middle ground**

------------------------------------------------------------------------

# The Core Problem with LLMs

Large Language Models generate text using **probabilities learned during
training**.

They do **not have built-in access to your company's private data**.

This leads to two major problems:

1.  The model may **not know the answer**
2.  The model may **confidently generate incorrect answers**

This behavior is known as **hallucination**.

Example:

    User: What is our company's Kafka partition scaling policy?

Since the model was trained mostly on public internet data, it may:

-   invent a policy
-   give a generic Kafka answer
-   provide outdated information

------------------------------------------------------------------------

# What RAG Does

RAG solves this by **retrieving real documents before the model
answers**.

Instead of relying only on training data, the model receives **external
context**.

Pipeline:

    User Query
    ↓
    Embedding generation
    ↓
    Vector search
    ↓
    Retrieve relevant documents
    ↓
    LLM generates answer using retrieved context

This process **grounds the model in real data**.

------------------------------------------------------------------------

# The Three Problems RAG Solves

## 1. Grounding

Grounding means **connecting model responses to real documents**.

Without grounding:

    User query → LLM → generated answer

With RAG:

    User query
    ↓
    Retrieve documents
    ↓
    LLM answers using retrieved documents

Example:

Query:

    How do we debug ECS deployment failures?

Retrieved documents:

    ECS troubleshooting guide
    deployment rollback runbook

The model now answers using **real internal documentation**.

------------------------------------------------------------------------

## 2. Accuracy

LLMs generate answers based on probability.

This means they may produce **plausible but incorrect responses**.

RAG improves accuracy by forcing the model to **reference retrieved
context**.

Example:

Without RAG:

    LLM guesses Kafka scaling approach

With RAG:

    LLM reads company Kafka scaling document
    → generates answer from it

Accuracy improves significantly.

------------------------------------------------------------------------

## 3. Freshness

LLMs are trained on **static datasets**.

Training data may be **months or years old**.

Example:

A model trained in 2023 may not know:

-   your latest architecture
-   updated internal APIs
-   current company policies

RAG solves this by retrieving **live documents**.

Example:

    New runbook added yesterday
    → indexed in vector DB
    → available for retrieval

No model retraining required.

------------------------------------------------------------------------

# Why Not Just Fine-Tune?

Fine-tuning trains a model on your data.

However it has several drawbacks.

### Problem 1 --- Static Knowledge

Fine-tuned models **cannot update easily**.

If documentation changes:

    fine-tune again

This is expensive and slow.

------------------------------------------------------------------------

### Problem 2 --- Large Data Requirements

Fine-tuning requires:

-   large datasets
-   careful dataset curation
-   training pipelines

Many companies don't have this infrastructure.

------------------------------------------------------------------------

### Problem 3 --- Hard to Trace Sources

Fine-tuning stores knowledge **inside model weights**.

This makes it difficult to trace answers back to documents.

Example:

    Which document did this answer come from?

This is difficult to answer with fine-tuned models.

------------------------------------------------------------------------

# Why Not Just Prompt Stuffing?

Another approach is to **paste documents directly into the prompt**.

Example:

    User question
    +
    Entire documentation
    +
    LLM

This is called **prompt stuffing**.

------------------------------------------------------------------------

## Problems with Prompt Stuffing

### Context Limits

LLMs have context limits.

Example:

    128k tokens

Large documentation sets cannot fit.

------------------------------------------------------------------------

### Cost

Large prompts dramatically increase token usage.

Example:

    50k tokens per request × thousands of users

Costs increase quickly.

------------------------------------------------------------------------

### Noise

Too much context makes reasoning worse.

The model struggles to identify **relevant information**.

------------------------------------------------------------------------

# Comparison of Approaches

  Approach          Pros                Cons
  ----------------- ------------------- -------------------------
  No grounding      simple              hallucinations
  Prompt stuffing   easy setup          context limits
  Fine-tuning       deeper knowledge    difficult updates
  RAG               dynamic retrieval   requires infrastructure

------------------------------------------------------------------------

# Why RAG Is the Practical Middle Ground

RAG combines:

-   **LLM reasoning**
-   **external knowledge retrieval**

This provides a balance of:

-   accuracy
-   flexibility
-   maintainability

Architecture:

    User query
    ↓
    Embedding
    ↓
    Vector search
    ↓
    Retrieve documents
    ↓
    LLM generation

------------------------------------------------------------------------

# Example Production Use Cases

### Internal Knowledge Assistants

Used to search:

-   engineering documentation
-   runbooks
-   incident reports
-   design docs

### Customer Support Bots

Retrieve:

-   support articles
-   FAQs
-   product documentation

### Developer Tools

Examples:

-   code search
-   API documentation assistants
-   internal StackOverflow-style tools

------------------------------------------------------------------------

# Example: Engineering RAG Assistant

Ingestion pipeline:

    Engineering docs
    ↓
    Chunking
    ↓
    Embedding generation
    ↓
    Vector DB storage

Query pipeline:

    Engineer query
    ↓
    Embedding
    ↓
    Vector search
    ↓
    Retrieve relevant docs
    ↓
    LLM generates grounded answer

------------------------------------------------------------------------

# Engineering Discussion Questions

### Question 1

When would **fine-tuning be better than RAG**?

### Question 2

Why might **prompt stuffing degrade LLM performance**?

### Question 3

What happens if vector search retrieves **irrelevant documents**? How
could you fix it?

------------------------------------------------------------------------

# Answers to Discussion Questions

### Answer 1 --- When Fine-Tuning Is Better

Fine-tuning is useful when:

-   The task requires **behavior learning rather than knowledge
    retrieval**
-   The system must follow **specific response formats**
-   The domain has **consistent patterns rather than changing
    documents**

Examples:

-   classification tasks
-   sentiment analysis
-   structured response generation

Fine-tuning works best when the knowledge **does not change
frequently**.

------------------------------------------------------------------------

### Answer 2 --- Why Prompt Stuffing Degrades Performance

Prompt stuffing reduces performance because:

1.  **Context limits** --- the model cannot process extremely large
    documents.
2.  **Noise** --- irrelevant information makes it harder for the model
    to find useful context.
3.  **Cost** --- large prompts dramatically increase token usage and
    inference cost.
4.  **Attention dilution** --- the model's attention spreads across too
    much information.

This makes reasoning less accurate.

------------------------------------------------------------------------

### Answer 3 --- If Vector Search Retrieves Irrelevant Documents

This problem is called **poor retrieval quality**.

Solutions include:

-   improving **chunking strategy**
-   using **better embedding models**
-   applying **hybrid search (BM25 + vector search)**
-   adding a **reranking model**
-   adjusting **similarity thresholds**
-   filtering by metadata

These techniques improve retrieval relevance.

------------------------------------------------------------------------

# Key Takeaways

1.  LLMs alone cannot access **private or up-to-date knowledge**.
2.  RAG solves three core problems:
    -   grounding
    -   accuracy
    -   freshness
3.  Fine-tuning stores knowledge in **model weights**, making updates
    difficult.
4.  Prompt stuffing fails due to **context limits and noise**.
5.  RAG provides the **most practical architecture for production AI
    systems**.
