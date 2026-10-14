
# Level 2: Containers

## Container list

| Container | Technology | Responsibility |
|-----------|------------|----------------|
| Web Application | React, TypeScript | User interface for search and meeting upload |
| API Gateway | Python, FastAPI | Orchestrates all requests, hybrid search, RAG context building |
| Ingestion Worker | Python | Diarization, transcription, chunking, embedding |
| PostgreSQL | PostgreSQL 15 | Structured data: raw transcripts, metadata, speaker records |
| Qdrant | Qdrant | Vector store: chunk embeddings, summary vectors, voice signatures |
| Object Storage | S3 / MinIO | Audio file binaries |
| Transcription Service | Pyannote + Whisper (wrapped as a service) | Diarization and speech-to-text |
| Embedding Service | Custom model (e.g., sentence-transformers) | Text and voice embedding generation |
| LLM Service | Local LLM (e.g., llama.cpp) | Final answer generation with RAG context |

## Diagram

```mermaid
flowchart LR
    user["👤 User (Browser)"]
    web["🌐 Web Application (React)"]
    api["⚙️ API Gateway (FastAPI)"]
    worker["⚙️ Ingestion Worker"]
    postgres[("🗄️ PostgreSQL")]
    qdrant[("🔍 Qdrant")]
    s3[("📦 Object Storage (MinIO)")]
    transcribe["🎤 Transcription Service"]
    embed["🧠 Embedding Service"]
    llm["🤖 LLM Service"]

    user -->|"Uses"| web
    web -->|"REST"| api
    api -->|"Search + filter"| qdrant
    api -->|"Read metadata"| postgres
    api -->|"Prompt"| llm
    api -->|"Embed query"| embed
    api -->|"Enqueue job"| worker
    worker -->|"Process audio"| transcribe
    worker -->|"Embed chunks/summaries"| embed
    worker -->|"Upsert vectors"| qdrant
    worker -->|"Write transcripts"| postgres
    worker -->|"Fetch/store audio"| s3

    classDef person fill:#08427B,color:#fff,stroke:#052E56
    classDef container fill:#1168BD,color:#fff,stroke:#0B4880
    classDef db fill:#1F5C99,color:#fff,stroke:#0B4880
    classDef ext fill:#999,color:#fff,stroke:#666

    class user person
    class web,api,worker,transcribe,embed,llm container
    class postgres,qdrant,s3 db
```
Redis is implied for task queuing between API and Worker but not shown as a main container for simplicity.