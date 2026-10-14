# Level 1: System Context

TRAIS as a whole (the system) interacts with one type of user and optionally with external infrastructure.

**Actors:**
- **User** – a person who uploads meeting audio, submits voice signatures, and asks questions in natural language.
- **External AI Services** (optional, if not bundled) – Pyannote, Whisper, Embedder, LLM. In a fully local deployment these are internal containers, but they may be treated as external if provided by a separate platform.

```mermaid
flowchart LR
    user["👤 User"]
    trais["TRAIS — RAG pipeline"]

    user -->|"Uses (HTTPS)"| trais

    classDef person fill:#08427B,color:#fff,stroke:#052E56
    classDef system fill:#1168BD,color:#fff,stroke:#0B4880
    class user person
    class trais system
```
In the fully local setup, all AI components run within the system boundary, so only the User is external.