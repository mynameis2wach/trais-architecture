# ADR-001: Separation of structured and vector storage

## Status
Accepted

## Context
The system works with two distinct categories of data:
- **Structured** — audio metadata, raw transcripts, speaker records, timestamps.
- **Vector** — chunk embeddings (typically 768+ dimensions), summary embeddings of various lengths.

Search queries must combine filtering by structured attributes (e.g., date range, speaker IDs) with vector similarity comparison. The central question is whether to store all data in a single database or to separate them.

A related concern is where to perform the structured filtering step during search: within the vector database itself or by pre‑filtering in a relational store (see also [ADR-003](adr-003-postgresql-prefiltering.md)).

## Decision
Use **two separate storage systems**:
- **PostgreSQL** — solely for structured data (transcripts, metadata, speaker records).
- **Qdrant** — for all vector data and for **payload‑based filtering**.

At search time, the system **never contacts PostgreSQL**. Both vector similarity search and structured filtering are executed inside Qdrant in a single gRPC call (see [ADR-004](adr-004-hybrid-search.md)). The alternative of pre‑filtering in PostgreSQL ([ADR-003](adr-003-postgresql-prefiltering.md)) was rejected due to unacceptable latency from extra round‑trips and application‑side complexity.

## Consequences
- **Positive**:
  - Structured and vector workloads have fundamentally different access patterns and scaling requirements; separation eliminates mutual contention.
  - The search path is closed within Qdrant — no additional service hops for filtering — keeping latency predictable and low.
  - This design proved to be cleaner than the rejected two‑step approach ([ADR-003](adr-003-postgresql-prefiltering.md)), where PostgreSQL pre‑filtering added round‑trips without a meaningful performance advantage at our scale.
- **Negative**:
  - Operating two database systems increases infrastructure complexity (monitoring, backups, resource allocation).
  - In some follow‑up scenarios (e.g., fetching full transcripts after search) an extra trip to PostgreSQL is still required; however, this is a read‑after‑ranking step that does not affect the search experience.