---
layout: post
title: "RAG Architecture: From Fundamentals to Enterprise Design"
date: 2026-11-20 20:30:00 +0530
categories: [ai, architecture]
description: "A practical introduction to Retrieval-Augmented Generation, its architecture, design choices and enterprise considerations."
reading_time: "10 min read"
---

Retrieval-Augmented Generation (RAG) is an architectural pattern that connects a language model to an external knowledge source at answer time. Instead of relying only on information encoded in model parameters, the application retrieves relevant content and supplies it as context for the model to use.

This is particularly useful when information changes frequently, is private to an organisation, or must be traceable to approved source material.

## 1. Why RAG?

A foundation model does not automatically know the latest version of an organisation's internal policies, product terms or operating procedures. Fine-tuning is not usually the right mechanism for keeping frequently changing factual knowledge current.

RAG separates two responsibilities:

- **Knowledge storage and retrieval:** curate, index and retrieve relevant source content.
- **Language generation:** use the retrieved evidence to formulate a useful response.

RAG can improve grounding, but it does not guarantee correctness. Retrieval quality, source quality, prompt design and evaluation all matter.

## 2. High-level architecture

A typical RAG solution has two flows: an ingestion flow that prepares the knowledge base, and a query flow that retrieves evidence for each user question.

```text
                 KNOWLEDGE INGESTION
                 -------------------
 Documents / PDFs / Web pages
             |
             v
      Parse and extract text
             |
             v
      Clean and normalise
             |
             v
       Chunk the content
             |
             v
   Generate embeddings + metadata
             |
             v
       Vector / search index


                   QUERY FLOW
                   ----------
 User question
      |
      v
 Identity, access and input checks
      |
      v
 Query understanding / embedding
      |
      v
 Retrieve relevant authorised chunks
      |
      v
 Re-rank and assemble context
      |
      v
 Prompt + context + user question
      |
      v
          Language model
      |
      v
 Grounded answer + source references
      |
      v
 Output checks, logging and response
```

The ingestion pipeline is usually asynchronous and can run whenever content is added or updated. The query pipeline is latency-sensitive and runs for each request.

## 3. Core building blocks

### Document ingestion

Sources may include PDFs, HTML pages, knowledge articles, product documents or structured records. Extraction should preserve useful structure such as headings, tables, document identifiers and effective dates.

### Chunking

Large documents are divided into smaller passages so retrieval can return focused evidence. Chunk size and overlap are design parameters, not universal constants.

A useful starting experiment is to compare multiple chunk sizes and evaluate retrieval quality against a representative question set. Keep metadata such as source ID, title, section, version, owner and effective date with each chunk.

### Embeddings and indexing

An embedding model converts text into a numerical representation. Similarity search uses these representations to find semantically related passages.

Depending on the use case, the index may support:
- Vector similarity search.
- Keyword or lexical search.
- Hybrid retrieval combining both.
- Metadata filters for document type, status, date or access scope.

### Retrieval and re-ranking

The retriever fetches candidate passages. A re-ranker can then score those candidates against the original question and move the most relevant evidence to the top.

Retrieving more passages is not always better. Too much context can increase cost, latency and distraction.

### Generation

The prompt should clearly distinguish the user's question from retrieved evidence. It should instruct the model to ground its response in the supplied context, acknowledge missing evidence and provide source references where possible.

## 4. RAG design decisions

| Decision | Questions to evaluate |
|---|---|
| Chunking | Fixed size, semantic or structure-aware? |
| Retrieval | Vector, keyword or hybrid? |
| Re-ranking | Is the extra latency justified? |
| Metadata | Which filters are required? |
| Model | Which model meets quality, latency and cost needs? |
| Context | How much evidence should be passed to the model? |
| Citations | How will each answer trace to its source? |
| Refresh | How are changed and deleted documents handled? |

There is no single best configuration. Evaluate with realistic questions and representative documents rather than choosing parameters based only on intuition.

## 5. Enterprise considerations

A production-oriented design needs more than a vector database and an LLM.

**Security and access control**
- Authenticate the user before retrieval.
- Enforce document-level or chunk-level authorisation at retrieval time.
- Prevent sensitive content from leaking across users or business units.
- Treat retrieved text as untrusted data, not as instructions to override system policy.

**Data lifecycle**
- Track document versions and effective dates.
- Support re-indexing when parsing, chunking or embedding logic changes.
- Remove or invalidate deleted and superseded content.
- Define retention and data residency requirements.

**Resilience and operations**
- Handle unavailable embedding, search and model services.
- Apply timeouts, bounded retries and rate limits.
- Monitor latency, token consumption, retrieval failures and model errors.
- Provide a safe fallback when evidence is unavailable.

**Governance**
- Record source identifiers and model/configuration versions.
- Protect prompts and logs from unnecessary sensitive data.
- Define human review for high-impact decisions.
- Test for prompt injection, data leakage and unsupported answers.

## 6. How should RAG be evaluated?

Evaluate retrieval and generation separately.

| Layer | Example measures |
|---|---|
| Retrieval | Recall@k, precision@k, ranking quality |
| Grounding | Faithfulness to retrieved evidence |
| Answer quality | Relevance, completeness and correctness |
| Operations | Latency, availability and cost |
| Security | Access-control tests and leakage scenarios |

Build a small, reviewed evaluation set containing realistic questions, expected source documents and acceptable answer characteristics. Re-run it when changing the chunking strategy, embedding model, retriever, prompt or language model.

## 7. RAG is a system, not a database feature

The most important architectural lesson is that RAG quality depends on the complete information lifecycle: trusted content, reliable ingestion, access-aware retrieval, grounded generation, evaluation and operational controls.

A vector index alone does not make an application enterprise-ready.

## What I will explore next

In the next knowledge drop, I will take this foundation further and examine how an agentic application can use retrieval as one capability among several tools, while retaining clear permissions, auditability and human oversight.

---

*Learning note: This article is an architectural introduction. Specific technology choices should be validated against the organisation's security, regulatory, data and operational requirements.*
