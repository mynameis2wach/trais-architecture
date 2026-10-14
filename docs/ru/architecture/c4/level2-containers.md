# Уровень 2: Контейнеры

## Список контейнеров

| Контейнер | Технология | Ответственность |
|-----------|------------|-----------------|
| Веб-приложение | React, TypeScript | Пользовательский интерфейс для поиска и загрузки встреч |
| Backend API | Python, FastAPI | Оркестрация всех запросов, гибридный поиск, построение RAG‑контекста |
| Обработчик загрузки | Python | Диаризация, транскрипция, разбивка на чанки, эмбеддинг |
| PostgreSQL | PostgreSQL 15 | Структурированные данные: сырые транскрипты, метаданные, записи о спикерах |
| Qdrant | Qdrant | Векторное хранилище: эмбеддинги чанков, векторы саммари, голосовые подписи |
| Объектное хранилище | S3 / MinIO | Бинарные файлы аудио |
| Сервис транскрипции | Pyannote + Whisper (обёрнутые как сервис) | Диаризация и распознавание речи |
| Сервис эмбеддинга | Собственная модель (например, sentence-transformers) | Генерация текстовых и голосовых эмбеддингов |
| LLM‑сервис | Локальная LLM (например, llama.cpp) | Генерация финального ответа на основе RAG‑контекста |

## Диаграмма

```mermaid
flowchart LR
    user["👤 Пользователь (браузер)"]
    web["🌐 Веб-приложение (React)"]
    api["⚙️ Backend API (FastAPI)"]
    worker["⚙️ Обработчик загрузки"]
    postgres[("🗄️ PostgreSQL")]
    qdrant[("🔍 Qdrant")]
    s3[("📦 Объектное хранилище (MinIO)")]
    transcribe["🎤 Сервис транскрипции"]
    embed["🧠 Сервис эмбеддинга"]
    llm["🤖 LLM‑сервис"]

    user -->|"Использует"| web
    web -->|"REST"| api
    api -->|"Поиск + фильтр"| qdrant
    api -->|"Чтение метаданных"| postgres
    api -->|"Промпт"| llm
    api -->|"Эмбеддинг запроса"| embed
    api -->|"Поставить задачу"| worker
    worker -->|"Обработка аудио"| transcribe
    worker -->|"Эмбеддинг чанков/саммари"| embed
    worker -->|"Добавление векторов"| qdrant
    worker -->|"Запись транскриптов"| postgres
    worker -->|"Загрузка/выгрузка аудио"| s3

    classDef person fill:#08427B,color:#fff,stroke:#052E56
    classDef container fill:#1168BD,color:#fff,stroke:#0B4880
    classDef db fill:#1F5C99,color:#fff,stroke:#0B4880
    classDef ext fill:#999,color:#fff,stroke:#666

    class user person
    class web,api,worker,transcribe,embed,llm container
    class postgres,qdrant,s3 db
```
Redis подразумевается для очереди задач между API и Обработчиком, но для простоты не показан как отдельный контейнер.
