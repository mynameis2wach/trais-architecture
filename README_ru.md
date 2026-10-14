# TRAIS — TRanscription, AI assisted Search

[English](README.md)

## Что это?
TRAIS — полностью локальный RAG‑пайплайн для работы с аудиоархивом встреч: транскрипция, идентификация спикеров, суммаризация и семантический поиск по всей истории.

## Зачем это нужно?
Крупные компании накапливают тысячи часов встреч. Информация в них теряется. TRAIS превращает неструктурированное аудио в базу знаний, доступную для вопросов, не отправляя данные вовне.

## Архитектурные принципы
- **Локальность** — никакие данные не покидают периметр компании.
- **Разделение хранилищ** — структурированные данные отделены от векторных ([ADR-001](docs/ru/architecture/decisions/adr-001-split-storage.md)).
- **Гибридный поиск** — объединяет семантику и точные фильтры в одном запросе ([ADR-004](docs/ru/architecture/decisions/adr-004-hybrid-search.md)).

## Ключевые технологии
Pyannote, Whisper, Qdrant, PostgreSQL, эмбеддер текста/аудио, локальная LLM.

## Карта документации
1. [Обзор архитектуры](docs/ru/architecture/overview.md)
2. C4‑диаграммы: [Контекст](docs/ru/architecture/c4/level1-context.md), [Контейнеры](docs/ru/architecture/c4/level2-containers.md), [Компоненты](docs/ru/architecture/c4/level3-components.md)
3. [Решения по архитектуре (ADR)](docs/ru/architecture/decisions/)