# TRAIS — TRanscription, AI assisted Search

[Русский](README_ru.md)
## What is it?
TRAIS is a fully local RAG pipeline for working with a meeting audio archive: transcription, speaker identification, summarization, and semantic search across the entire history.

## Why is it needed?
Large companies accumulate thousands of hours of meetings. The information inside them gets lost. TRAIS turns unstructured audio into a question‑accessible knowledge base without sending any data outside.

## Architectural principles
- **Locality** — no data leaves the company perimeter.
- **Storage separation** — structured data is kept apart from vector data (see [ADR-001](docs/eng/architecture/decisions/adr-001-split-storage.md)).
- **Hybrid search** — combines semantics and exact filters in a single query ([ADR-001](docs/eng/architecture/decisions/adr-004-hybrid-search.md)).

## Key technologies
Pyannote, Whisper, Qdrant, PostgreSQL, Text/Audio Embedder, local LLM.

## Documentation map
1. [Architecture overview](docs/eng/architecture/overview.md)
2. C4 diagrams: [Context](docs/eng/architecture/c4/level1-context.md), [Containers](docs/eng/architecture/c4/level2-containers.md), [Components](docs/eng/architecture/c4/level3-components.md)
3. [Architecture Decision Records](docs/eng/architecture/decisions/)

