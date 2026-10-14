# Level 3: Components

## API Gateway – internal components

The API Gateway is built around a central `MainManager` that orchestrates all operations.  
Specialised managers handle communication with external services, while `AuthMiddleware` enforces security.

| Component | Responsibility |
|-----------|----------------|
| `MainManager` | Accepts `/ask` and `/upload` requests, builds RAG context, coordinates search and ingestion |
| `QdrantRequestManager` | Encapsulates gRPC calls to Qdrant |
| `LLMManager` | Communicates with the LLM service |
| `AuthMiddleware` | (future) Authentication and rate limiting |

```mermaid
flowchart LR
    main["MainManager (orchestrator)"]
    qdrant["QdrantRequestManager"]
    llm["LLMManager"]
    auth["AuthMiddleware"]

    main -->|"Uses"| qdrant
    main -->|"Uses"| llm
    main -->|"Checked by"| auth

    classDef component fill:#1168BD,color:#fff,stroke:#0B4880
    class main,qdrant,llm,auth component
```
## Sequence diagrams for core flows

These diagrams illustrate how the components collaborate.

### Upload Voice Signature

```mermaid
sequenceDiagram
    participant Client
    participant TRAIS
    participant AudioProcessor
    participant Embedder
    participant Qdrant

    Client->>TRAIS: POST /upload_signature
    TRAIS->>AudioProcessor: normalize / trim
    AudioProcessor-->>TRAIS: clean audio
    TRAIS->>Embedder: embedding(audio)
    Embedder-->>TRAIS: vector
    TRAIS->>TRAIS: validate quality
    TRAIS->>Qdrant: upsert(speaker_id, vector, metadata)
    TRAIS-->>Client: ok
```
### Ask LLM
```mermaid
sequenceDiagram
    participant Client
    participant TRAIS
    participant Embedder
    participant Qdrant
    participant PostgreSQL
    participant LLM

    Client->>TRAIS: POST /ask
    TRAIS->>Embedder: embedding(query)
    Embedder-->>TRAIS: vector
    TRAIS->>Qdrant: search(vector)
    Qdrant-->>TRAIS: candidates
    alt candidates found
        TRAIS->>PostgreSQL: metadata(ids)
        PostgreSQL-->>TRAIS: data
        TRAIS->>TRAIS: build RAG context
        TRAIS->>LLM: prompt(context + query)
        LLM-->>TRAIS: answer
    else empty
        TRAIS->>LLM: prompt(query only)
        LLM-->>TRAIS: fallback answer
    end
    TRAIS-->>Client: answer
```
### Upload Meeting
```mermaid
sequenceDiagram
    participant Client
    participant TRAIS
    participant Pyannote
    participant Whisper
    participant Embedder
    participant Qdrant
    participant PostgreSQL
    participant LLM

    Client->>TRAIS: POST /upload
    TRAIS->>Pyannote: diarization(audio)
    Pyannote-->>TRAIS: segments
    loop for each segment
        TRAIS->>Whisper: transcribe(segment)
        Whisper-->>TRAIS: text
        TRAIS->>Embedder: voice embedding(segment)
        Embedder-->>TRAIS: vector
        TRAIS->>Qdrant: identify speaker(vector)
        Qdrant-->>TRAIS: speaker_id
        TRAIS->>Embedder: text embedding(text)
        Embedder-->>TRAIS: text_vector
        TRAIS->>Qdrant: upsert chunk(text_vector, speaker_id, meeting_id)
    end
    TRAIS->>TRAIS: aggregate transcript
    TRAIS->>LLM: prompt(full transcript)
    LLM-->>TRAIS: structured output (summary, title)
    TRAIS->>Embedder: embeddings(summary + title)
    Embedder-->>TRAIS: vectors
    TRAIS->>Qdrant: upsert meeting(vectors)
    TRAIS->>PostgreSQL: store metadata
    TRAIS-->>Client: done
```