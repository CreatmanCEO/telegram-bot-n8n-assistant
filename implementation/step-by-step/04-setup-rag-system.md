# Настройка системы векторной памяти (RAG)

В этом руководстве подробно описан процесс настройки и интеграции системы Retrieval-Augmented Generation (RAG) для телеграм-бота на n8n. Система RAG позволяет боту накапливать знания, хранить информацию о пользователе и его предпочтениях, а также использовать эту информацию для предоставления более релевантных и контекстуальных ответов.

## Предварительные требования

1. Уже настроенная основная инфраструктура бота с центральным оркестратором
2. Действующий API-ключ Pinecone
3. Действующий API-ключ OpenAI (для создания эмбеддингов)
4. (Опционально) Настроенный сервер Zep для хранения истории диалогов

## Шаг 1: Настройка хранилища векторов Pinecone

### 1.1. Создание индексов в Pinecone

В Pinecone мы создадим несколько индексов для разных типов данных:

1. **user_data** - для хранения информации о пользователе
2. **knowledge_base** - для хранения общих знаний
3. **domain_knowledge** - для хранения специализированных знаний по конкретным темам

Выполните следующие действия в консоли Pinecone:

1. Войдите в [консоль Pinecone](https://app.pinecone.io/)
2. Нажмите "Create Index"
3. Настройте индекс:
   - **Name**: `user_data`
   - **Dimensions**: `1536` (для OpenAI embeddings) или `768` (для других моделей)
   - **Metric**: `cosine`
   - **Pod Type**: Выберите в зависимости от ваших потребностей (для начала достаточно `starter`)
4. Нажмите "Create Index"
5. Повторите процесс для индексов `knowledge_base` и `domain_knowledge`

### 1.2. Настройка учетных данных Pinecone в n8n

1. В интерфейсе n8n перейдите в "Settings" > "Credentials"
2. Нажмите "Add Credential"
3. Выберите "Pinecone API"
4. Заполните следующие поля:
   - **Credential Name**: `Pinecone Vector DB`
   - **API Key**: Ваш API-ключ Pinecone
   - **Environment**: Ваш Pinecone environment (например, `us-east1-gcp`)
5. Нажмите "Save"

## Шаг 2: Настройка OpenAI Embeddings

### 2.1. Настройка учетных данных OpenAI в n8n

1. В интерфейсе n8n перейдите в "Settings" > "Credentials"
2. Нажмите "Add Credential"
3. Выберите "OpenAI API"
4. Заполните следующие поля:
   - **Credential Name**: `OpenAI Embeddings`
   - **API Key**: Ваш API-ключ OpenAI
5. Нажмите "Save"

### 2.2. Создание ноды OpenAI Embeddings

Создайте новый рабочий поток для индексации данных в Pinecone:

1. Создайте новый workflow с именем "RAG Indexing"
2. Добавьте ноду "OpenAI" из библиотеки n8n
3. Настройте ноду:
   - **Resource**: `Embeddings`
   - **Operation**: `Create`
   - **Input**: `={{ $json.text }}`
   - **Model**: `text-embedding-ada-002`
4. Подключите учетные данные OpenAI, которые вы создали ранее

## Шаг 3: Создание рабочего потока для индексации данных

### 3.1. Базовая структура рабочего потока

Создайте следующую структуру рабочего потока:

```
HTTP Trigger → JSON Parser → OpenAI Embeddings → Pinecone Upsert → HTTP Response
```

### 3.2. Настройка HTTP Trigger

1. Добавьте ноду "HTTP Trigger"
2. Настройте ноду:
   - **Authentication**: `Basic Auth`
   - **HTTP Method**: `POST`
   - **Path**: `/rag/index`
   - **Respond**: `Immediately`

### 3.3. Настройка JSON Parser

1. Добавьте ноду "JSON Parse"
2. Настройте ноду:
   - **Property Name**: `data`
   - **Keep Only Set**: ✓

### 3.4. Настройка Pinecone Upsert

1. Добавьте ноду "Pinecone"
2. Настройте ноду:
   - **Operation**: `Upsert`
   - **Index**: `={{ $json.index || 'knowledge_base' }}`
   - **Vector**: `={{ $json.embedding }}`
   - **ID**: `={{ $json.id || $uuid }}`
   - **Metadata**: 
   ```json
   {
     "text": "={{ $json.text }}",
     "source": "={{ $json.source || 'user_input' }}",
     "category": "={{ $json.category || 'general' }}",
     "timestamp": "={{ $now }}",
     "userId": "={{ $json.userId }}"
   }
   ```

### 3.5. Настройка HTTP Response

1. Добавьте ноду "Respond to Webhook"
2. Настройте ноду:
   - **Response Code**: `200`
   - **Response Body**: `={"success": true, "id": "{{ $json.id }}"}`

## Шаг 4: Создание рабочего потока для поиска информации

### 4.1. Базовая структура рабочего потока

Создайте следующую структуру рабочего потока:

```
HTTP Trigger → JSON Parser → OpenAI Embeddings → Pinecone Query → Format Results → HTTP Response
```

### 4.2. Настройка HTTP Trigger

1. Добавьте ноду "HTTP Trigger"
2. Настройте ноду:
   - **Authentication**: `Basic Auth`
   - **HTTP Method**: `POST`
   - **Path**: `/rag/search`
   - **Respond**: `Immediately`

### 4.3. Настройка Pinecone Query

1. Добавьте ноду "Pinecone"
2. Настройте ноду:
   - **Operation**: `Query`
   - **Index**: `={{ $json.index || 'knowledge_base' }}`
   - **Vector**: `={{ $json.embedding }}`
   - **Top K**: `5`
   - **Filters**: 
   ```json
   {
     "userId": "={{ $json.userId || null }}",
     "category": "={{ $json.category || null }}"
   }
   ```
   - **Include Metadata**: ✓

### 4.4. Настройка Format Results

1. Добавьте ноду "Function"
2. Настройте ноду:
   ```javascript
   // Форматирование результатов запроса
   const results = $input.first().json.matches || [];
   const formattedResults = results.map(match => ({
     text: match.metadata.text,
     score: match.score,
     source: match.metadata.source,
     category: match.metadata.category,
     timestamp: match.metadata.timestamp
   }));
   
   return {
     json: {
       results: formattedResults,
       query: $input.first().json.queryText
     }
   };
   ```

## Шаг 5: Интеграция с основным рабочим потоком

### 5.1. Добавление RAG Tool в Main Orchestrator

1. Откройте рабочий поток "Main" (центральный оркестратор)
2. Добавьте ноду "@n8n/n8n-nodes-langchain.vectorStorePinecone"
3. Настройте ноду:
   ```json
   {
     "parameters": {
       "mode": "retrieve-as-tool",
       "toolName": "Knowledge",
       "toolDescription": "Use this tool when asking questions from our own knowledge base or when you need context about user preferences and previous interactions.",
       "pineconeIndex": "knowledge_base",
       "options": {}
     },
     "name": "Knowledge database",
     "type": "@n8n/n8n-nodes-langchain.vectorStorePinecone",
     "typeVersion": 1.1,
     "position": [1100, 460],
     "credentials": {
       "pineconeApi": {
         "id": "YOUR_CREDENTIALS_ID",
         "name": "Pinecone API account"
       }
     }
   }
   ```

4. Добавьте ноду "@n8n/n8n-nodes-langchain.embeddingsOpenAi"
5. Настройте ноду:
   ```json
   {
     "parameters": {
       "options": {}
     },
     "name": "Embeddings OpenAI",
     "type": "@n8n/n8n-nodes-langchain.embeddingsOpenAi",
     "typeVersion": 1.2,
     "credentials": {
       "openAiApi": {
         "id": "YOUR_CREDENTIALS_ID",
         "name": "OpenAi account"
       }
     }
   }
   ```

6. Подключите "Embeddings OpenAI" к "Knowledge database" через соединение "ai_embedding"

### 5.2. Обновление системного промпта Main Agent

Обновите системный промпт в Main Agent, добавив инструкции по использованию Knowledge tool:

```
## Работа с памятью
- Используй Knowledge tool для получения информации из векторной памяти
- При получении ценной информации от пользователя, сохраняй ее в Knowledge
- Используй информацию из Knowledge для персонализации ответов
- Обращайся к Knowledge для получения контекста о предыдущих взаимодействиях
```

## Шаг 6: Создание механизма сохранения информации

### 6.1. Создание рабочего потока для сохранения информации

Создайте новый рабочий поток "Save Knowledge":

```
Manual Trigger → Set → OpenAI Embeddings → Pinecone Upsert
```

### 6.2. Настройка Set ноды

1. Добавьте ноду "Set"
2. Настройте ноду:
   ```json
   {
     "parameters": {
       "keepOnlySet": true,
       "values": {
         "text": "={{ $json.text }}",
         "userId": "={{ $json.userId }}",
         "category": "={{ $json.category }}",
         "source": "={{ $json.source }}",
         "id": "={{ $uuid }}"
       }
     }
   }
   ```

### 6.3. Создание Tool для сохранения информации

1. Откройте рабочий поток "Main"
2. Добавьте ноду "@n8n/n8n-nodes-langchain.toolWorkflow"
3. Настройте ноду:
   ```json
   {
     "parameters": {
       "name": "saveKnowledge",
       "description": "Используйте этот инструмент для сохранения важной информации в памяти системы. Всегда указывайте text (текст для сохранения), userId (ID пользователя), category (категория) и source (источник информации).",
       "workflowId": {
         "value": "YOUR_WORKFLOW_ID",
         "mode": "list",
         "cachedResultName": "Save Knowledge"
       },
       "workflowInputs": {
         "mappingMode": "defineBelow",
         "value": {
           "text": "={{ $fromAI(\"text\") }}",
           "userId": "={{ $fromAI(\"userId\") }}",
           "category": "={{ $fromAI(\"category\") }}",
           "source": "={{ $fromAI(\"source\") }}"
         }
       }
     }
   }
   ```

4. Обновите системный промпт в Main Agent, добавив инструкции по использованию saveKnowledge tool:

```
## Сохранение информации
Используй saveKnowledge tool для сохранения важной информации:
- text: текст для сохранения
- userId: ID пользователя (используй ID из Telegram)
- category: категория информации (preferences, facts, contacts, etc.)
- source: источник информации (user_input, web_search, conversation, etc.)

Пример: Если пользователь говорит "Я предпочитаю получать новости рано утром", сохрани эту информацию с категорией "preferences".
```

## Шаг 7: Настройка специального агента для управления пользовательским профилем

### 7.1. Создание UserProfileAgent

Создайте новый рабочий поток "User Profile Agent":

```
Workflow Trigger → User Profile Logic → Return Response
```

### 7.2. Настройка User Profile Logic

1. Добавьте ноду "@n8n/n8n-nodes-langchain.agent"
2. Настройте ноду:
   ```json
   {
     "parameters": {
       "promptType": "define",
       "text": "={{ $json.query }}",
       "options": {
         "systemMessage": "# UserProfileAgent\nТы агент, отвечающий за управление информацией о пользователе. Твоя задача - собирать, обновлять и извлекать информацию о пользователе.\n\n## Функции\n1. Создание и обновление профиля пользователя\n2. Извлечение информации о предпочтениях пользователя\n3. Создание информационного портрета пользователя\n\n## Инструменты\n- Используй searchUserProfile для поиска информации о пользователе\n- Используй updateUserProfile для обновления профиля пользователя\n- Используй createUserProfile для создания нового профиля\n\n## Правила\n- Всегда требуй userId для операций с профилем\n- Категоризируй информацию по типам (личные данные, предпочтения, интересы и т.д.)\n- Обеспечивай конфиденциальность данных пользователя"
       }
     }
   }
   ```

### 7.3. Добавление User Profile Tools

1. Добавьте ноду "@n8n/n8n-nodes-langchain.vectorStorePinecone" для поиска
2. Настройте ноду:
   ```json
   {
     "parameters": {
       "mode": "retrieve-as-tool",
       "toolName": "searchUserProfile",
       "toolDescription": "Search for information in the user profile. Requires userId parameter.",
       "pineconeIndex": "user_data",
       "options": {}
     }
   }
   ```

3. Добавьте HTTP Request ноды для обновления и создания профилей, подключенные к вашим эндпоинтам индексации

### 7.4. Интеграция UserProfileAgent с Main Orchestrator

1. Добавьте ноду "@n8n/n8n-nodes-langchain.toolWorkflow" в Main Orchestrator
2. Настройте ноду:
   ```json
   {
     "parameters": {
       "name": "userProfileAgent",
       "description": "Используйте этот инструмент для управления информацией о пользователе.",
       "workflowId": {
         "value": "YOUR_WORKFLOW_ID",
         "mode": "list",
         "cachedResultName": "User Profile Agent"
       }
     }
   }
   ```

## Шаг 8: Тестирование и отладка системы RAG

### 8.1. Тестирование индексации

1. Отправьте тестовый запрос к эндпоинту `/rag/index`:
   ```json
   {
     "text": "Пользователь предпочитает получать новости утром.",
     "userId": "123456789",
     "category": "preferences",
     "source": "test"
   }
   ```

2. Проверьте, что данные были успешно добавлены в Pinecone:
   - Войдите в консоль Pinecone
   - Перейдите к индексу `knowledge_base`
   - Используйте Query Console для проверки

### 8.2. Тестирование поиска

1. Отправьте тестовый запрос к эндпоинту `/rag/search`:
   ```json
   {
     "text": "Когда пользователь предпочитает получать новости?",
     "userId": "123456789"
   }
   ```

2. Проверьте, что результаты содержат релевантную информацию

### 8.3. Тестирование интеграции с основным рабочим потоком

1. Откройте чат с ботом в Telegram
2. Отправьте сообщение, связанное с информацией, которую вы добавили
3. Проверьте, использует ли бот эту информацию в своем ответе

## Шаг 9: Оптимизация и масштабирование

### 9.1. Оптимизация производительности

- Используйте кэширование для часто запрашиваемой информации
- Ограничивайте количество запросов к Pinecone
- Оптимизируйте размер эмбеддингов

### 9.2. Организация данных

- Создайте схему метаданных для категоризации информации
- Разделите данные по индексам в зависимости от типа
- Регулярно обновляйте и очищайте устаревшие данные

### 9.3. Мониторинг и логирование

- Добавьте логирование для всех операций с RAG
- Настройте мониторинг использования Pinecone
- Отслеживайте качество результатов поиска