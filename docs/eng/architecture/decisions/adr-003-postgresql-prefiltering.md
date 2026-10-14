# ADR-003: PostgreSQL pre‑filtering for hybrid search

## Status
Rejected

## Context
When executing hybrid queries (semantic search + structured filters), an alternative to performing everything inside Qdrant was considered: first run the structured filter in PostgreSQL to obtain a subset of candidate meeting/chunk IDs, then pass those IDs to Qdrant for the vector similarity step.

The rationale was that PostgreSQL may handle large volumes of structured data more efficiently than Qdrant’s payload index, potentially reducing the number of vectors that Qdrant has to scan and therefore improving overall response time.

## Decision
Implement a two-step search flow:
1. **PostgreSQL** filters meetings/chunks by date, speaker, and other structured metadata, returning the allowed IDs.
2. **Qdrant** performs the vector similarity search, scoped to those IDs.

## Consequences
- **Positive**:  
  PostgreSQL is highly optimised for complex relational filtering on massive tables, which could, in theory, narrow down the search space faster than Qdrant’s payload filters for certain query patterns.
- **Negative** (reason for rejection):  
  The approach introduces at least one extra network round-trip (API → PostgreSQL → API → Qdrant).  
  It shifts complexity to the application layer, requiring the API to stitch together two separate queries.  
  In practice, Qdrant’s payload-based filtering runs the entire hybrid query in a single gRPC call, and the latency penalty of the additional round‑trip outweighs any potential gain from PostgreSQL pre‑filtering.  
  The optimisation would only become noticeable under extremely high data volumes, but even then, the added latency and complexity make it a poor trade‑off for the expected scale.

## Related decisions
Rejected in favour of [**ADR-004**](adr-004-hybrid-search.md) — hybrid search executed entirely inside Qdrant with payload filters.