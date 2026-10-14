# ADR-002: Choice of Qdrant over PostgreSQL + pgvector

## Status
Accepted

## Context
The system generates several types of vectors per meeting:
- multiple chunk-level embeddings (768+ dimensions each),
- one or more summary vectors (up to 1024 dimensions).

With a typical load of dozens of meetings per day, the total number of stored vectors quickly reaches hundreds of thousands and then millions.  
pgvector is an extension of PostgreSQL not designed for such volumes of high‑dimensional vectors; it shows significant performance degradation as the dataset grows.

## Decision
Use a dedicated vector database — **Qdrant** — which is built natively for storing and searching millions of vectors with payload‑based filtering.

## Consequences
- **Positive**:
  - High performance and predictable query latency at scale.
  - Native support for multiple vectors per point fits our data model naturally (chunk embeddings + summary vectors colocated with their metadata).
  - Strong integration with the hybrid search approach (see [ADR-004](adr-004-hybrid-search.md)).
- **Negative**:
  - An additional database system to maintain and monitor (see also [ADR-001](adr-001-split-storage.md) for the overall storage split).
  - One more technology in the stack, increasing operational complexity.