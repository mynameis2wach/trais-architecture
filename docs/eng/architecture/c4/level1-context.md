# Level 1: System Context

TRAIS as a whole (the system) interacts with one type of user and optionally with external infrastructure.

**Actors:**
- **User** – a person who uploads meeting audio, submits voice signatures, and asks questions in natural language.
- **AI Services** – Pyannote, Whisper, Embedder, LLM. In a fully local deployment these are internal containers, but they may be treated as external if provided by a separate platform.


```mermaid
flowchart LR
    user["👤 User"]
    trais["TRAIS — RAG pipeline"]
    ai["External/Local AI services<br/>: Pyannote, Whisper, Embedder, LLM"]

    user -->|"HTTPS: audio, voice, questions"| trais
    trais -.->|"ASR, diarization, embeddings"| ai

    classDef person fill:#08427B,color:#fff,stroke:#052E56
    classDef system fill:#1168BD,color:#fff,stroke:#0B4880
    classDef external fill:#777,color:#fff,stroke:#555,stroke-dasharray: 5 5

    class user person
    class trais system
    class ai external
```
In the fully local setup, all AI components run within the system boundary, so only the User is external.
