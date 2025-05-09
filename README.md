# Интеллектуальный телеграм-бот на базе n8n

Этот репозиторий содержит полную документацию по разработке, настройке и улучшению интеллектуального ассистента на базе n8n, который интегрируется с Telegram и использует возможности современных языковых моделей.

## Возможности системы

- **Обработка сообщений**: Текстовых, голосовых и медиа-сообщений через Telegram
- **Интеллектуальный анализ**: Распознавание задач пользователя, их приоритизация и планирование выполнения
- **Поиск информации**: Поиск и анализ информации в интернете по запросу пользователя
- **Векторная память (RAG)**: Накопление знаний о пользователе и интересующих его темах
- **Работа с сервисами Google**: Управление электронной почтой, календарем и документами
- **Создание контента**: Написание различных текстов для СМИ и соцсетей
- **Генерация медиа**: Создание изображений и видео через интеграцию с Fal.AI
- **Публикация в соцсетях**: Автоматизация публикаций в Telegram, Twitter(X), Facebook и других платформах
- **Исследование тем**: Глубокое изучение запрошенных тем с созданием базы знаний
- **Персонализация**: Создание подробного информационного портрета пользователя

## Структура документации

- [**Архитектура системы**](ARCHITECTURE.md) - Общее описание архитектуры и компонентов
- [**План развития**](ROADMAP.md) - Дорожная карта по улучшению системы

### Текущее состояние системы
- [Обзор текущей системы](docs/current-state/system-overview.md)
- [Описание существующих компонентов](docs/current-state/components-description.md)
- [Известные проблемы](docs/current-state/known-issues.md)
- [Текущие ограничения](docs/current-state/limitations.md)

### Запланированное состояние системы
- [Обзор запланированной системы](docs/planned-state/system-overview.md)
- [Описание новых компонентов](docs/planned-state/new-components.md)
- [Планируемые улучшения](docs/planned-state/improvements.md)
- [План интеграции новых компонентов](docs/planned-state/integration-plan.md)

### Инструкции по настройке
- [Необходимые предварительные условия](docs/setup/prerequisites.md)
- [Инструкции по установке](docs/setup/installation.md)
- [Настройка системы](docs/setup/configuration.md)
- [Развертывание на сервере](docs/setup/deployment.md)

### Получение ключей API
- [Настройка Telegram Bot API](docs/api-keys/telegram-setup.md)
- [Получение ключей OpenRouter](docs/api-keys/openrouter-setup.md)
- [Настройка Fal.AI](docs/api-keys/falai-setup.md)
- [Настройка Google Services](docs/api-keys/google-services-setup.md)
- [Настройка Pinecone](docs/api-keys/pinecone-setup.md)
- [Настройка API социальных сетей](docs/api-keys/social-media-setup.md)

### Рабочие потоки
- [Главный оркестратор](docs/workflows/main-orchestrator.md)
- [Агент поиска информации](docs/workflows/search-agent.md)
- [Агент работы с почтой](docs/workflows/email-agent.md)
- [Агент работы с календарем](docs/workflows/calendar-agent.md)
- [Агент создания контента](docs/workflows/content-agent.md)
- [Агент создания медиа-контента](docs/workflows/media-agent.md)
- [Агент публикации в соцсетях](docs/workflows/social-agent.md)
- [Система векторной памяти](docs/workflows/rag-memory-system.md)

## Реализация и отладка
- [Пошаговые инструкции по внедрению](implementation/step-by-step/)
- [Руководство по компонентам](implementation/components/)
- [Продвинутые техники](implementation/advanced/)
- [Устранение неполадок](troubleshooting/)

## Примеры
- [Примеры рабочих потоков](examples/workflows/)
- [Примеры конфигураций](examples/configurations/)

## Требования
- VPS с установленным n8n
- Доступ к API Telegram
- Доступ к OpenRouter для использования различных LLM-моделей
- Доступ к Fal.AI для генерации медиа-контента
- Доступ к Pinecone для векторной памяти
- Доступ к API социальных сетей (по необходимости)

## Начало работы

Для начала работы с проектом, ознакомьтесь с [инструкциями по установке](docs/setup/installation.md) и [настройке системы](docs/setup/configuration.md).