# ADR-004: Hybrid search with payload‑based filtering

## Status
Accepted

## Context
Users need to search meeting history by combining semantic understanding with exact filters on dates, speakers, duration, and other metadata. The challenge is to execute this efficiently without moving large amounts of data between services.

## Decision
Perform **hybrid search directly inside Qdrant**: a single request carries both the query vector and the structured filters (payload conditions). Qdrant returns a ranked list of relevant chunk or meeting IDs in one step.

The API Gateway constructs this unified query and never performs separate filtering in PostgreSQL (see [ADR-001](adr-001-split-storage.md) and [ADR-003](adr-003-postgresql-prefiltering.md)).

## Consequences
- **Positive**:
  - Maximum performance – filtering and ranking occur in the same engine, avoiding data transfer overhead.
  - Simple API code – no stitching of results from different databases.
- **Negative**:
  - The filtering logic depends on Qdrant’s payload capabilities; extremely complex SQL‑style filters might require post‑processing.
  - Metadata required for filtering must be duplicated in Qdrant payloads, increasing storage consumption slightly.