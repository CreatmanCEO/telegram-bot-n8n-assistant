# Настройка Google Services для интеграции с n8n

## Обзор

Google Services предоставляет широкий спектр API для интеграции с различными сервисами Google, включая Gmail, Google Calendar, Google Drive и другие. В нашем проекте интеграция с Google Services используется для:

1. **Gmail** — чтение, отправка и управление электронной почтой
2. **Google Calendar** — создание, редактирование и управление событиями календаря
3. **Google Drive** — хранение и управление файлами и документами

Для доступа к этим сервисам необходимо настроить проект в Google Cloud Platform и получить соответствующие API-ключи и токены доступа.

## Необходимые интеграции

Для полноценной работы телеграм-бота необходимо настроить следующие Google API:

- Gmail API
- Google Calendar API
- Google Drive API
- Google Sheets API (опционально, для хранения структурированных данных)
- Google Docs API (опционально, для работы с документами)

## Шаги по настройке Google Cloud Platform

### 1. Создание проекта в Google Cloud Platform

1. Перейдите на [Google Cloud Console](https://console.cloud.google.com/)
2. Нажмите на выпадающее меню проектов в верхней части экрана и выберите "New Project"
3. Введите название проекта (например, "n8n-telegram-assistant")
4. Нажмите "Create"
5. Дождитесь создания проекта и переключитесь на него

### 2. Активация необходимых API

1. В боковом меню выберите "APIs & Services" > "Library"
2. Найдите и активируйте следующие API:
   - Gmail API
   - Google Calendar API
   - Google Drive API
   - Google Sheets API (если необходимо)
   - Google Docs API (если необходимо)
3. Для каждого API:
   - Нажмите на API в списке результатов поиска
   - Нажмите кнопку "Enable"
   - Дождитесь активации API

### 3. Настройка экрана согласия OAuth

1. В боковом меню выберите "APIs & Services" > "OAuth consent screen"
2. Выберите тип пользователей:
   - "External" для публичного использования (доступно всем пользователям Google)
   - "Internal" для использования только внутри вашей организации (рекомендуется для тестирования)
3. Нажмите "Create"
4. Заполните обязательные поля:
   - App name: название вашего приложения
   - User support email: ваш email для поддержки
   - Developer contact information: ваш контактный email
5. Нажмите "Save and Continue"
6. В разделе "Scopes":
   - Нажмите "Add or Remove Scopes"
   - Добавьте следующие scopes:
     - `https://www.googleapis.com/auth/gmail.modify` (для Gmail)
     - `https://www.googleapis.com/auth/calendar` (для Calendar)
     - `https://www.googleapis.com/auth/drive` (для Drive)
     - `https://www.googleapis.com/auth/spreadsheets` (для Sheets, если необходимо)
     - `https://www.googleapis.com/auth/documents` (для Docs, если необходимо)
7. Нажмите "Save and Continue"
8. В разделе "Test users":
   - Нажмите "Add Users"
   - Добавьте email аккаунтов, которые будут использоваться для тестирования
   - Нажмите "Save and Continue"
9. Проверьте введенную информацию и нажмите "Back to Dashboard"

### 4. Создание учетных данных OAuth

1. В боковом меню выберите "APIs & Services" > "Credentials"
2. Нажмите "Create Credentials" > "OAuth client ID"
3. Выберите тип приложения: "Web application"
4. Введите название (например, "n8n OAuth Client")
5. В разделе "Authorized redirect URIs":
   - Нажмите "Add URI"
   - Добавьте URL перенаправления из n8n (обычно это `https://your-n8n-domain/rest/oauth2-credential/callback` или `http://localhost:5678/rest/oauth2-credential/callback` для локальной разработки)
6. Нажмите "Create"
7. Сохраните появившиеся Client ID и Client Secret в безопасном месте

### 5. (Опционально) Создание API-ключа для доступа к API без OAuth

1. В боковом меню выберите "APIs & Services" > "Credentials"
2. Нажмите "Create Credentials" > "API key"
3. Сохраните сгенерированный API-ключ в безопасном месте
4. Нажмите "Restrict Key" для ограничения доступа к API:
   - В разделе "API restrictions" выберите "Restrict key"
   - Выберите API, к которым будет иметь доступ этот ключ
   - Нажмите "Save"

## Настройка n8n для работы с Google Services

### 1. Настройка OAuth для Gmail в n8n

1. В интерфейсе n8n перейдите в "Settings" > "Credentials"
2. Нажмите "Add Credential"
3. В поисковой строке введите "Gmail"
4. Выберите тип учетных данных "Gmail OAuth2 API"
5. Заполните следующие поля:
   - **Credential Name**: Понятное название (например, "Gmail Account")
   - **Client ID**: Вставьте Client ID из Google Cloud Platform
   - **Client Secret**: Вставьте Client Secret из Google Cloud Platform
   - **Scope**: Выберите необходимые разрешения (минимум `https://www.googleapis.com/auth/gmail.modify`)
6. Нажмите "Connect"
7. Авторизуйтесь в своем аккаунте Google и предоставьте запрашиваемые разрешения
8. После успешной авторизации нажмите "Save" для сохранения учетных данных

### 2. Настройка OAuth для Google Calendar в n8n

1. В интерфейсе n8n перейдите в "Settings" > "Credentials"
2. Нажмите "Add Credential"
3. В поисковой строке введите "Google Calendar"
4. Выберите тип учетных данных "Google Calendar OAuth2 API"
5. Заполните следующие поля:
   - **Credential Name**: Понятное название (например, "Google Calendar Account")
   - **Client ID**: Вставьте Client ID из Google Cloud Platform
   - **Client Secret**: Вставьте Client Secret из Google Cloud Platform
   - **Scope**: Выберите необходимые разрешения (минимум `https://www.googleapis.com/auth/calendar`)
6. Нажмите "Connect"
7. Авторизуйтесь в своем аккаунте Google и предоставьте запрашиваемые разрешения
8. После успешной авторизации нажмите "Save" для сохранения учетных данных

### 3. Настройка OAuth для Google Drive в n8n

1. В интерфейсе n8n перейдите в "Settings" > "Credentials"
2. Нажмите "Add Credential"
3. В поисковой строке введите "Google Drive"
4. Выберите тип учетных данных "Google Drive OAuth2 API"
5. Заполните следующие поля:
   - **Credential Name**: Понятное название (например, "Google Drive Account")
   - **Client ID**: Вставьте Client ID из Google Cloud Platform
   - **Client Secret**: Вставьте Client Secret из Google Cloud Platform
   - **Scope**: Выберите необходимые разрешения (минимум `https://www.googleapis.com/auth/drive`)
6. Нажмите "Connect"
7. Авторизуйтесь в своем аккаунте Google и предоставьте запрашиваемые разрешения
8. После успешной авторизации нажмите "Save" для сохранения учетных данных

## Интеграция с агентами n8n

### 1. EmailAgent - Интеграция с Gmail

Для работы EmailAgent необходимо настроить следующие ноды в рабочем потоке:

#### 1.1. Gmail Tool для отправки писем

```json
{
  "parameters": {
    "sendTo": "={{ $fromAI(\"emailAddress\") }}",
    "subject": "={{ $fromAI(\"subject\") }}",
    "message": "={{ $fromAI(\"emailBody\") }}",
    "options": {
      "appendAttribution": false
    }
  },
  "type": "n8n-nodes-base.gmailTool",
  "typeVersion": 2.1,
  "position": [600, 300],
  "name": "Send Email",
  "credentials": {
    "gmailOAuth2": {
      "id": "YOUR_CREDENTIALS_ID",
      "name": "Gmail account"
    }
  }
}
```

#### 1.2. Gmail Tool для получения писем

```json
{
  "parameters": {
    "operation": "getAll",
    "limit": "={{ $fromAI(\"limit\",\"how many emails the user wants\") }}",
    "simple": false,
    "filters": {
      "sender": "={{ $fromAI(\"sender\",\"who the emails are from\") }}"
    },
    "options": {}
  },
  "type": "n8n-nodes-base.gmailTool",
  "typeVersion": 2.1,
  "position": [600, 400],
  "name": "Get Emails",
  "credentials": {
    "gmailOAuth2": {
      "id": "YOUR_CREDENTIALS_ID",
      "name": "Gmail account"
    }
  }
}
```

#### 1.3. Gmail Tool для создания черновиков

```json
{
  "parameters": {
    "resource": "draft",
    "subject": "={{ $fromAI(\"subject\") }}",
    "emailType": "html",
    "message": "={{ $fromAI(\"emailBody\") }}",
    "options": {
      "sendTo": "={{ $fromAI(\"emailAddress\") }}"
    }
  },
  "type": "n8n-nodes-base.gmailTool",
  "typeVersion": 2.1,
  "position": [600, 500],
  "name": "Create Draft",
  "credentials": {
    "gmailOAuth2": {
      "id": "YOUR_CREDENTIALS_ID",
      "name": "Gmail account"
    }
  }
}
```

### 2. CalendarAgent - Интеграция с Google Calendar

Для работы CalendarAgent необходимо настроить следующие ноды в рабочем потоке:

#### 2.1. Google Calendar Tool для создания событий

```json
{
  "parameters": {
    "calendar": {
      "value": "primary",
      "mode": "list"
    },
    "start": "={{ $fromAI(\"eventStart\") }}",
    "end": "={{ $fromAI(\"eventEnd\") }}",
    "additionalFields": {
      "attendees": [],
      "summary": "={{ $fromAI(\"eventTitle\") }}"
    }
  },
  "type": "n8n-nodes-base.googleCalendarTool",
  "typeVersion": 1.3,
  "position": [600, 300],
  "name": "Create Event",
  "credentials": {
    "googleCalendarOAuth2Api": {
      "id": "YOUR_CREDENTIALS_ID",
      "name": "Google Calendar account"
    }
  }
}
```

#### 2.2. Google Calendar Tool для получения событий

```json
{
  "parameters": {
    "operation": "getAll",
    "calendar": {
      "value": "primary",
      "mode": "list"
    },
    "timeMin": "={{ $fromAI(\"dayBefore\",\"the day before the date the user requested\") }}",
    "timeMax": "={{ $fromAI(\"dayAfter\",\"the day after the date the user requested\") }}",
    "options": {}
  },
  "type": "n8n-nodes-base.googleCalendarTool",
  "typeVersion": 1.3,
  "position": [600, 400],
  "name": "Get Events",
  "credentials": {
    "googleCalendarOAuth2Api": {
      "id": "YOUR_CREDENTIALS_ID",
      "name": "Google Calendar account"
    }
  }
}
```

#### 2.3. Google Calendar Tool для удаления событий

```json
{
  "parameters": {
    "operation": "delete",
    "calendar": {
      "value": "primary",
      "mode": "list"
    },
    "eventId": "={{ $fromAI(\"eventID\") }}",
    "options": {}
  },
  "type": "n8n-nodes-base.googleCalendarTool",
  "typeVersion": 1.3,
  "position": [600, 500],
  "name": "Delete Event",
  "credentials": {
    "googleCalendarOAuth2Api": {
      "id": "YOUR_CREDENTIALS_ID",
      "name": "Google Calendar account"
    }
  }
}
```

### 3. MediaStorageAgent - Интеграция с Google Drive

Для сохранения и управления медиа-файлами в Google Drive необходимо настроить следующие ноды:

#### 3.1. Google Drive для загрузки файлов

```json
{
  "parameters": {
    "operation": "upload",
    "name": "={{ $json.fileName }}",
    "driveId": {
      "value": "My Drive",
      "mode": "list"
    },
    "folderId": {
      "value": "YOUR_FOLDER_ID",
      "mode": "id"
    },
    "binary": {
      "data": "={{ $json.binaryPropertyName }}"
    },
    "options": {}
  },
  "type": "n8n-nodes-base.googleDrive",
  "typeVersion": 3,
  "position": [600, 300],
  "name": "Upload File to Drive",
  "credentials": {
    "googleDriveOAuth2Api": {
      "id": "YOUR_CREDENTIALS_ID",
      "name": "Google Drive account"
    }
  }
}
```

#### 3.2. Google Drive для скачивания файлов

```json
{
  "parameters": {
    "operation": "download",
    "fileId": {
      "value": "={{ $json.fileId }}",
      "mode": "id"
    },
    "options": {}
  },
  "type": "n8n-nodes-base.googleDrive",
  "typeVersion": 3,
  "position": [600, 400],
  "name": "Download File from Drive",
  "credentials": {
    "googleDriveOAuth2Api": {
      "id": "YOUR_CREDENTIALS_ID",
      "name": "Google Drive account"
    }
  }
}
```

#### 3.3. Google Drive для поиска файлов

```json
{
  "parameters": {
    "operation": "search",
    "queryParameters": {
      "parameters": [
        {
          "name": "q",
          "value": "={{ \"name contains '\"+$json.searchTerm+\"'\" }}"
        }
      ]
    },
    "options": {}
  },
  "type": "n8n-nodes-base.googleDrive",
  "typeVersion": 3,
  "position": [600, 500],
  "name": "Search Files in Drive",
  "credentials": {
    "googleDriveOAuth2Api": {
      "id": "YOUR_CREDENTIALS_ID",
      "name": "Google Drive account"
    }
  }
}
```

## Оптимизация интеграции с Google Services

### 1. Управление квотами и ограничениями API

Google API имеют ограничения на количество запросов в день/секунду. Для оптимизации использования:

1. **Мониторинг квот**
   - Регулярно проверяйте использование квот в [Google Cloud Console](https://console.cloud.google.com/apis/dashboard)
   - Настройте оповещения о приближении к лимитам

2. **Кэширование данных**
   - Кэшируйте часто запрашиваемые данные
   - Используйте локальное хранилище для уменьшения количества запросов к API

3. **Пакетные запросы**
   - Объединяйте несколько операций в один запрос, когда это возможно
   - Используйте batch API для Gmail и Drive

### 2. Обработка токенов OAuth

1. **Автоматическое обновление токенов**
   - n8n автоматически обновляет токены OAuth, но стоит проверять их статус
   - Реализуйте механизм обработки ошибок в случае проблем с обновлением токенов

2. **Безопасное хранение токенов**
   - Убедитесь, что n8n использует безопасное хранилище для токенов
   - Ограничьте доступ к настройкам n8n

### 3. Обработка ошибок при работе с API

1. **Типичные ошибки и их решения**
   - `401 Unauthorized`: проблемы с токеном доступа, необходимо переаутентифицироваться
   - `403 Forbidden`: недостаточно прав, проверьте scopes OAuth
   - `429 Too Many Requests`: превышены квоты, реализуйте механизм повторных попыток с экспоненциальной задержкой

2. **Реализация механизма повторных попыток**
   ```javascript
   async function retryApiCall(apiCallFunction, maxRetries = 3, initialDelay = 1000) {
     let retries = 0;
     
     while (retries < maxRetries) {
       try {
         return await apiCallFunction();
       } catch (error) {
         if (error.status === 429 || retries === maxRetries - 1) {
           throw error;
         }
         
         const delay = initialDelay * Math.pow(2, retries);
         await new Promise(resolve => setTimeout(resolve, delay));
         
         retries++;
       }
     }
   }
   ```

## Безопасность и приватность

### 1. Ограничение доступа к API

1. **Ограничение области видимости (scopes) OAuth**
   - Запрашивайте только необходимые разрешения
   - Используйте наиболее ограниченные scopes, которые позволяют выполнить требуемые операции

2. **Ограничение доступа API-ключей**
   - Ограничьте API-ключи конкретными API
   - Настройте ограничения по IP-адресам

### 2. Обработка конфиденциальных данных

1. **Шифрование данных в покое**
   - Убедитесь, что n8n использует шифрование для хранения учетных данных
   - Рассмотрите возможность дополнительного шифрования конфиденциальных данных

2. **Минимизация данных**
   - Загружайте и храните только необходимые данные
   - Регулярно очищайте кэши и временные файлы

3. **Логирование и аудит**
   - Настройте логирование для отслеживания доступа к конфиденциальным данным
   - Регулярно проверяйте логи на предмет подозрительной активности

## Рекомендации для рабочих процессов с Google Services

### 1. EmailAgent

1. **Оптимизация обработки писем**
   - Используйте фильтры при получении писем для уменьшения объема данных
   - Реализуйте пагинацию для обработки большого количества писем

2. **Форматирование писем**
   - Используйте HTML-форматирование для создания красивых писем
   - Предоставьте шаблоны для типичных сценариев использования

### 2. CalendarAgent

1. **Работа с повторяющимися событиями**
   - Используйте правила повторения для создания регулярных событий
   - Обрабатывайте исключения для повторяющихся событий

2. **Работа с разными часовыми поясами**
   - Учитывайте часовой пояс пользователя
   - Указывайте часовой пояс явно при создании событий

### 3. MediaStorageAgent

1. **Организация файлов**
   - Создайте структуру папок для разных типов контента
   - Используйте метаданные для категоризации файлов

2. **Оптимизация загрузки/скачивания**
   - Реализуйте возобновляемую загрузку для больших файлов
   - Используйте потоковую передачу для медиа-файлов

## Примеры системных промптов для агентов

### 1. Системный промпт для EmailAgent

```
# EmailAgent

Ты специализированный агент для работы с электронной почтой. Твоя задача - помогать пользователю управлять его почтой через Gmail.

## Возможности

1. Отправка писем
2. Чтение писем
3. Создание черновиков
4. Ответы на письма
5. Работа с метками

## Доступные инструменты

1. Send Email - отправка писем
2. Get Emails - получение писем
3. Create Draft - создание черновиков
4. Email Reply - ответ на письмо
5. Label Emails - работа с метками

## Правила работы

1. Всегда запрашивай подтверждение перед отправкой письма
2. При получении писем используй фильтрацию для уменьшения объема данных
3. Форматируй письма для улучшения читаемости
4. Предлагай готовые шаблоны для типичных сценариев
```

### 2. Системный промпт для CalendarAgent

```
# CalendarAgent

Ты специализированный агент для работы с календарем. Твоя задача - помогать пользователю управлять его расписанием через Google Calendar.

## Возможности

1. Создание событий
2. Получение списка событий
3. Обновление событий
4. Удаление событий

## Доступные инструменты

1. Create Event - создание события
2. Get Events - получение списка событий
3. Update Event - обновление события
4. Delete Event - удаление события

## Правила работы

1. Всегда уточняй дату и время события
2. Предлагай оптимальное время для событий на основе текущего расписания
3. Добавляй напоминания для важных событий
4. Учитывай часовой пояс пользователя
```

## Дополнительные ресурсы

- [Документация Gmail API](https://developers.google.com/gmail/api/guides)
- [Документация Google Calendar API](https://developers.google.com/calendar/api/guides/overview)
- [Документация Google Drive API](https://developers.google.com/drive/api/guides/about-sdk)
- [Руководство по OAuth 2.0 для Google API](https://developers.google.com/identity/protocols/oauth2)
- [n8n документация по Google интеграциям](https://docs.n8n.io/integrations/builtin/credentials/google/)
- [Лучшие практики для работы с Google API](https://developers.google.com/apis-explorer/#best-practices)