---
layout: post
title: "RAG Architecture: From Fundamentals to Enterprise Design"
date: 2026-11-20 20:30:00 +0530
categories: [ai, architecture]
description: "A practical introduction to Retrieval-Augmented Generation, its architecture, design choices and enterprise considerations."
reading_time: "10 min read"
---

## 1. What RAG Is

Retrieval-Augmented Generation (RAG) is an architectural pattern that connects a language model to an external knowledge source at answer time. Instead of relying only on information encoded in model parameters, the application retrieves relevant content and supplies it as context for the model to use.

This is particularly useful when information changes frequently, is private to an organisation, or must be traceable to approved source material.

```mermaid
flowchart LR
    Q["User question"] --> R["Retrieve relevant content"]
    K[("External knowledge source")] --> R
    R --> C["Context"]
    Q --> P["Prompt"]
    C --> P
    P --> M["Language model"]
    M --> A["Grounded answer"]
```

Best suited when information:

- Changes frequently
- Is private to an organisation
- Must be traceable to approved source material

---

## 2. Why RAG: Separation of Responsibilities

```mermaid
flowchart TB
    subgraph S1["Knowledge storage and retrieval"]
        A1["Curate source content"] --> A2["Index"] --> A3["Retrieve"]
    end
    subgraph S2["Language generation"]
        B1["Use retrieved evidence"] --> B2["Formulate response"]
    end
    A3 --> B1
```

| Concern | Position in the article |
| --- | --- |
| Fine-tuning | Not usually the right mechanism for keeping frequently changing facts current |
| Correctness | RAG improves grounding but does not guarantee correctness |
| Quality drivers | Retrieval quality, source quality, prompt design, evaluation |

---

## 3. High-Level Architecture: Two Flows

```mermaid
flowchart LR
    subgraph ING["Ingestion flow (asynchronous)"]
        direction TB
        I1["Documents, PDFs, web pages"] --> I2["Parse and extract text"]
        I2 --> I3["Clean and normalise"]
        I3 --> I4["Chunk the content"]
        I4 --> I5["Generate embeddings + metadata"]
    end

    IDX[("Vector / search index")]

    subgraph QRY["Query flow (latency-sensitive)"]
        direction TB
        Q1["User question"] --> Q2["Identity, access and input checks"]
        Q2 --> Q3["Query understanding / embedding"]
        Q3 --> Q4["Retrieve authorised chunks"]
        Q4 --> Q5["Re-rank and assemble context"]
        Q5 --> Q6["Prompt + context + question"]
        Q6 --> Q7["Language model"]
        Q7 --> Q8["Grounded answer + source references"]
        Q8 --> Q9["Output checks, logging, response"]
    end

    I5 --> IDX
    IDX --> Q4
```

| Flow | Trigger | Characteristic |
| --- | --- | --- |
| Ingestion | Content added or updated | Asynchronous |
| Query | Each user request | Latency-sensitive |

---

## 4. Query Flow as a Sequence

```mermaid
sequenceDiagram
    actor U as User
    participant App as Application
    participant Sec as Access checks
    participant Idx as Search index
    participant RR as Re-ranker
    participant LLM as Language model

    U->>App: Question
    App->>Sec: Authenticate and validate input
    Sec-->>App: User identity and access scope
    App->>Idx: Embed query and search with access filters
    Idx-->>App: Candidate chunks
    App->>RR: Candidates + question
    RR-->>App: Top-ranked evidence
    App->>LLM: Prompt with question and context
    LLM-->>App: Answer with source references
    App->>App: Output checks and logging
    App-->>U: Response
```

---

## 5. Core Building Blocks

### 5.1 Ingestion

Extraction should preserve:

- Headings
- Tables
- Document identifiers
- Effective dates

### 5.2 Chunking and Metadata

Chunk size and overlap are design parameters, not universal constants.

```mermaid
flowchart LR
    D["Large document"] --> C1["Chunk 1"]
    D --> C2["Chunk 2"]
    D --> C3["Chunk n"]
    C1 --> M["Metadata kept with each chunk"]
    M --> F1["Source ID"]
    M --> F2["Title"]
    M --> F3["Section"]
    M --> F4["Version"]
    M --> F5["Owner"]
    M --> F6["Effective date"]
```

Recommended starting experiment: compare multiple chunk sizes and evaluate retrieval quality against a representative question set.

### 5.3 Embeddings and Indexing

```mermaid
flowchart TB
    T["Text"] --> E["Embedding model"] --> V["Numerical representation"]
    V --> IDX[("Index")]
    IDX --> O1["Vector similarity search"]
    IDX --> O2["Keyword / lexical search"]
    IDX --> O3["Hybrid retrieval"]
    IDX --> O4["Metadata filters: type, status, date, access scope"]
```

### 5.4 Retrieval and Re-ranking

```mermaid
flowchart LR
    Q["Question"] --> R["Retriever"]
    R --> CAND["Candidate passages"]
    CAND --> RR["Re-ranker scores against the question"]
    RR --> TOP["Most relevant evidence on top"]
    TOP --> CTX["Context passed to model"]
```

More passages are not always better: excess context increases cost, latency and distraction.

### 5.5 Generation

The prompt must:

1. Separate the user question from retrieved evidence
2. Instruct the model to ground its response in supplied context
3. Instruct the model to acknowledge missing evidence
4. Request source references where possible

---

## 6. Design Decisions

| Decision | Questions to evaluate |
| --- | --- |
| Chunking | Fixed size, semantic or structure-aware? |
| Retrieval | Vector, keyword or hybrid? |
| Re-ranking | Is the extra latency justified? |
| Metadata | Which filters are required? |
| Model | Which model meets quality, latency and cost needs? |
| Context | How much evidence should be passed to the model? |
| Citations | How will each answer trace to its source? |
| Refresh | How are changed and deleted documents handled? |

There is no single best configuration. Evaluate with realistic questions and representative documents.

---

## 7. Enterprise Considerations

```mermaid
mindmap
  root((Enterprise RAG))
    Security and access control
      Authenticate before retrieval
      Document-level or chunk-level authorisation
      No leakage across users or business units
      Retrieved text is untrusted data
    Data lifecycle
      Track versions and effective dates
      Re-index when logic changes
      Remove superseded and deleted content
      Retention and data residency
    Resilience and operations
      Handle service outages
      Timeouts, bounded retries, rate limits
      Monitor latency, tokens, failures
      Safe fallback when evidence is missing
    Governance
      Record source IDs and model versions
      Protect prompts and logs
      Human review for high-impact decisions
      Test prompt injection and leakage
```

### Security Enforcement Point

```mermaid
flowchart LR
    U["User"] --> AUTH["Authenticate"]
    AUTH --> RET["Retrieval with authorisation filter"]
    RET --> CH["Only chunks the user may access"]
    CH --> LLM["Language model"]
    LLM --> OUT["Output checks"]
    RET -. "Retrieved text treated as data, not instructions" .- LLM
```

---

## 8. Evaluation

Evaluate retrieval and generation separately.

| Layer | Example measures |
| --- | --- |
| Retrieval | Recall@k, precision@k, ranking quality |
| Grounding | Faithfulness to retrieved evidence |
| Answer quality | Relevance, completeness, correctness |
| Operations | Latency, availability, cost |
| Security | Access-control tests, leakage scenarios |

```mermaid
flowchart LR
    SET["Reviewed evaluation set: questions, expected sources, acceptable answer traits"] --> RUN["Run evaluation"]
    RUN --> SCORE["Score each layer"]
    SCORE --> CHANGE{"Change to chunking, embedding model, retriever, prompt or LLM?"}
    CHANGE -- "Yes" --> RUN
    CHANGE -- "No" --> MON["Continue monitoring"]
```

---

## 9. Key Takeaway

RAG is a system, not a database feature.

```mermaid
flowchart LR
    A["Trusted content"] --> B["Reliable ingestion"] --> C["Access-aware retrieval"] --> D["Grounded generation"] --> E["Evaluation"] --> F["Operational controls"]
```

A vector index alone does not make an application enterprise-ready.

---

*Learning note: This article is an architectural introduction. Specific technology choices should be validated against the organisation's security, regulatory, data and operational requirements.*
