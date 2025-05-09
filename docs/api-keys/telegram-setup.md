# Настройка Telegram Bot API

## Обзор

Telegram Bot API позволяет создавать ботов для платформы Telegram. Бот — это специальный аккаунт, которым можно управлять программно и который не требует дополнительного номера телефона. Боты могут отвечать на сообщения, отправлять различные типы контента и интегрироваться с другими сервисами.

В нашем проекте Telegram используется как основной интерфейс взаимодействия с пользователем. Telegram Bot API обеспечивает прием сообщений от пользователя (текстовых, голосовых, медиа) и отправку ответов.

## Шаги по созданию Telegram бота и получению API-ключа

### 1. Создание нового бота с помощью BotFather

1. Откройте Telegram и найдите [@BotFather](https://t.me/BotFather)
2. Отправьте команду `/newbot`
3. Следуйте инструкциям:
   - Введите имя бота (то, которое будет отображаться пользователям)
   - Введите имя пользователя для бота (должно заканчиваться на 'bot')
4. После успешного создания BotFather выдаст вам токен API для вашего бота. Это строка, которая выглядит примерно так: `123456789:ABCDefGhIJKlmNoPQRsTUVwxyZ`
5. **ВАЖНО: Сохраните этот токен в безопасном месте!** Он предоставляет полный доступ к вашему боту

### 2. Настройка дополнительных параметров бота

#### 2.1. Установка описания бота
1. Отправьте команду `/setdescription` BotFather
2. Выберите своего бота из списка
3. Введите описание бота (до 512 символов)

#### 2.2. Установка картинки профиля
1. Отправьте команду `/setuserpic` BotFather
2. Выберите своего бота из списка
3. Отправьте изображение, которое хотите использовать в качестве аватара

#### 2.3. Настройка команд бота (опционально)
1. Отправьте команду `/setcommands` BotFather
2. Выберите своего бота из списка
3. Отправьте список команд в формате:
   ```
   start - Начать разговор с ботом
   help - Получить справку по работе с ботом
   settings - Настройки бота
   ```

### 3. Настройка приватности (рекомендуется)

По умолчанию ваш бот получает все сообщения, отправленные в группы. Это может быть нежелательно с точки зрения конфиденциальности и производительности.

1. Отправьте команду `/setprivacy` BotFather
2. Выберите своего бота из списка
3. Выберите `Enable` для включения режима приватности (бот будет получать только сообщения, начинающиеся с '/' или упоминающие его)

## Интеграция Telegram Bot API с n8n

### 1. Добавление учетных данных в n8n

1. В интерфейсе n8n перейдите в "Settings" > "Credentials"
2. Нажмите кнопку "Add Credential"
3. В поисковой строке введите "Telegram"
4. Выберите тип учетных данных "Telegram API"
5. Заполните следующие поля:
   - **Name**: Понятное название (например, "My Assistant Bot")
   - **Access Token**: Вставьте токен, полученный от BotFather
6. Нажмите "Save" для сохранения учетных данных

### 2. Создание Telegram Trigger ноды для приема сообщений

```json
{
  "parameters": {
    "updates": ["message"],
    "additionalFields": {}
  },
  "name": "Telegram Trigger",
  "type": "n8n-nodes-base.telegramTrigger",
  "typeVersion": 1.1,
  "position": [0, 0],
  "webhookId": "YOUR_WEBHOOK_ID",
  "credentials": {
    "telegramApi": {
      "id": "YOUR_CREDENTIALS_ID",
      "name": "Your Telegram Bot"
    }
  }
}
```

### 3. Создание Telegram ноды для отправки сообщений

```json
{
  "parameters": {
    "chatId": "={{ $('Telegram Trigger').item.json.message.chat.id }}",
    "text": "={{ $json.output }}",
    "additionalFields": {
      "appendAttribution": false
    }
  },
  "name": "Send Telegram Message",
  "type": "n8n-nodes-base.telegram",
  "typeVersion": 1.2,
  "position": [800, 220],
  "credentials": {
    "telegramApi": {
      "id": "YOUR_CREDENTIALS_ID",
      "name": "Your Telegram Bot"
    }
  }
}
```

### 4. Настройка ноды для загрузки голосовых сообщений

```json
{
  "parameters": {
    "resource": "file",
    "fileId": "={{ $json.message.voice.file_id }}"
  },
  "name": "Download Voice Message",
  "type": "n8n-nodes-base.telegram",
  "typeVersion": 1.2,
  "position": [200, 100],
  "credentials": {
    "telegramApi": {
      "id": "YOUR_CREDENTIALS_ID",
      "name": "Your Telegram Bot"
    }
  }
}
```

## Настройка вебхуков для Telegram

Для работы Telegram бота через n8n необходимо настроить вебхуки. Есть два основных подхода:

### Подход 1: Использование n8n Webhook URL (для публичных серверов)

Если ваш сервер n8n имеет публичный IP-адрес или домен:

1. Получите URL вебхука от n8n:
   - В интерфейсе n8n откройте рабочий поток с Telegram Trigger
   - Активируйте рабочий поток
   - Скопируйте URL вебхука из Telegram Trigger ноды

2. Настройте вебхук в Telegram API:
   ```
   https://api.telegram.org/bot<ваш_токен>/setWebhook?url=<ваш_webhook_url>
   ```

### Подход 2: Использование n8n на локальном сервере через туннель

Если ваш сервер n8n находится за NAT или не имеет публичного IP-адреса:

1. Установите и настройте [ngrok](https://ngrok.com/):
   ```bash
   ngrok http <порт_вашего_n8n>
   ```

2. Скопируйте HTTPS URL, предоставленный ngrok (например, `https://abcd1234.ngrok.io`)

3. Настройте вебхук в Telegram API:
   ```
   https://api.telegram.org/bot<ваш_токен>/setWebhook?url=<ваш_ngrok_url>/webhook/<ваш_webhook_id>
   ```

### Подход 3: Использование n8n на VPS

Для нашего проекта, где n8n установлен на VPS, рекомендуется:

1. Настроить HTTPS на вашем сервере (используя Let's Encrypt или другой сервис SSL/TLS)
2. Настроить обратный прокси (nginx, Apache) для перенаправления запросов к n8n
3. Использовать подход 1 с публичным URL вашего сервера

## Расширенные функции Telegram Bot API

### 1. Отправка медиа-контента

#### Отправка изображений

```json
{
  "parameters": {
    "operation": "sendPhoto",
    "chatId": "={{ $('Telegram Trigger').item.json.message.chat.id }}",
    "binaryData": true,
    "binaryPropertyName": "image",
    "additionalFields": {
      "caption": "Сгенерированное изображение"
    }
  },
  "name": "Send Image",
  "type": "n8n-nodes-base.telegram",
  "typeVersion": 1.2
}
```

#### Отправка документов

```json
{
  "parameters": {
    "operation": "sendDocument",
    "chatId": "={{ $('Telegram Trigger').item.json.message.chat.id }}",
    "binaryData": true,
    "binaryPropertyName": "document",
    "additionalFields": {
      "caption": "Ваш документ"
    }
  },
  "name": "Send Document",
  "type": "n8n-nodes-base.telegram",
  "typeVersion": 1.2
}
```

### 2. Использование клавиатуры для интерактивности

#### Inline клавиатура

```json
{
  "parameters": {
    "chatId": "={{ $('Telegram Trigger').item.json.message.chat.id }}",
    "text": "Пожалуйста, выберите действие:",
    "additionalFields": {
      "reply_markup": {
        "inline_keyboard": [
          [
            {
              "text": "Подтвердить",
              "callback_data": "confirm"
            },
            {
              "text": "Отменить",
              "callback_data": "cancel"
            }
          ]
        ]
      }
    }
  },
  "name": "Send With Inline Keyboard",
  "type": "n8n-nodes-base.telegram",
  "typeVersion": 1.2
}
```

#### Обработка callback запросов

Для обработки нажатий на кнопки inline клавиатуры, настройте Telegram Trigger для приема callback_query:

```json
{
  "parameters": {
    "updates": ["callback_query"],
    "additionalFields": {}
  },
  "name": "Callback Trigger",
  "type": "n8n-nodes-base.telegramTrigger",
  "typeVersion": 1.1
}
```

### 3. Отправка сообщений с форматированием

Telegram поддерживает форматирование сообщений с использованием Markdown или HTML:

```json
{
  "parameters": {
    "chatId": "={{ $('Telegram Trigger').item.json.message.chat.id }}",
    "text": "**Жирный текст**, *курсив* и `моноширинный шрифт`",
    "additionalFields": {
      "parse_mode": "Markdown"
    }
  },
  "name": "Send Formatted Message",
  "type": "n8n-nodes-base.telegram",
  "typeVersion": 1.2
}
```

## Рекомендации по безопасности

### 1. Защита токена API

- Никогда не публикуйте токен API вашего бота в открытых репозиториях
- Используйте переменные окружения или секреты n8n для хранения токена
- Регулярно проверяйте активные сессии бота через BotFather

### 2. Ограничение доступа

- Рассмотрите возможность реализации аутентификации пользователей
- Храните список разрешенных пользователей (например, в базе данных или конфигурационном файле)
- Проверяйте ID пользователя/чата перед выполнением конфиденциальных операций

### 3. Мониторинг активности

- Реализуйте логирование всех взаимодействий с ботом
- Настройте уведомления о подозрительной активности
- Периодически просматривайте логи на предмет потенциальных проблем безопасности

## Устранение проблем

### 1. Бот не получает сообщения

**Возможные причины и решения**:
- **Неправильно настроен вебхук**
  - Проверьте URL вебхука с помощью `https://api.telegram.org/bot<ваш_токен>/getWebhookInfo`
  - Убедитесь, что URL доступен из интернета
- **Проблемы с SSL**
  - Telegram требует HTTPS для вебхуков
  - Проверьте валидность вашего SSL-сертификата
- **Бот не добавлен в чат**
  - Убедитесь, что пользователь начал диалог с ботом

### 2. Ошибки при отправке сообщений

**Возможные причины и решения**:
- **Неверный формат параметров**
  - Проверьте соответствие параметров документации Telegram Bot API
- **Ограничения на контент**
  - Убедитесь, что размер и формат отправляемых файлов соответствуют ограничениям Telegram
- **Пользователь заблокировал бота**
  - Обрабатывайте ошибки 403 Forbidden при отправке сообщений

### 3. Проблемы с производительностью

**Возможные решения**:
- Оптимизируйте обработку входящих сообщений
- Используйте асинхронную обработку для длительных операций
- Реализуйте механизм троттлинга для предотвращения перегрузки сервера

## Дополнительные ресурсы

- [Официальная документация Telegram Bot API](https://core.telegram.org/bots/api)
- [Введение в Telegram боты](https://core.telegram.org/bots)
- [n8n документация по Telegram интеграции](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.telegram/)
- [Примеры Telegram ботов на GitHub](https://github.com/topics/telegram-bot)