# Настройка Fal.AI для генерации медиа-контента

## Обзор

Fal.AI — это платформа для развертывания и запуска моделей искусственного интеллекта в облаке. Для нашего телеграм-бота мы используем Fal.AI для генерации изображений и видеоконтента. Платформа предоставляет доступ к различным моделям, включая Stable Diffusion, DALL-E и другие.

## Преимущества использования Fal.AI

1. **Широкий выбор моделей** для генерации различных типов контента
2. **Высокая производительность** благодаря оптимизированной инфраструктуре
3. **Простой API** для интеграции с различными приложениями
4. **Гибкая система оплаты** с возможностью оплаты только за фактическое использование

## Шаги по получению API-ключа Fal.AI

### 1. Регистрация на Fal.AI

1. Перейдите на сайт [Fal.AI](https://www.fal.ai/)
2. Нажмите кнопку "Sign Up" в правом верхнем углу
3. Зарегистрируйтесь с помощью:
   - GitHub
   - Google
   - Email + пароль
4. Подтвердите свою учетную запись через email (если выбран метод регистрации через email)

### 2. Создание API-ключа

1. После входа в систему перейдите в раздел "API Keys" в боковом меню
2. Нажмите кнопку "Create API Key"
3. Дайте ключу понятное название (например, "N8N Bot Integration")
4. Скопируйте сгенерированный ключ и сохраните его в безопасном месте. Ключ будет показан только один раз!

### 3. Получение кредитов

1. Перейдите в раздел "Billing" в боковом меню
2. Нажмите кнопку "Add Credits"
3. Выберите подходящий план или пополните баланс
4. Завершите процесс оплаты
5. Проверьте, что кредиты были зачислены на ваш аккаунт

## Интеграция с n8n

### 1. Добавление учетных данных в n8n

1. В интерфейсе n8n перейдите в "Settings" > "Credentials"
2. Нажмите кнопку "Add Credential"
3. В поисковой строке введите "HTTP"
4. Выберите тип учетных данных "HTTP Header Auth"
5. Заполните следующие поля:
   - **Name**: Понятное название (например, "Fal.AI API Access")
   - **Authentication Type**: Header Auth
   - **Name**: Authorization
   - **Value**: Key {ваш_API_ключ}
6. Нажмите "Save" для сохранения учетных данных

### 2. Создание HTTP Request ноды для генерации изображений

#### Использование Stable Diffusion XL

```json
{
  "parameters": {
    "method": "POST",
    "url": "https://api.fal.ai/text-to-image",
    "authentication": "headerAuth",
    "contentType": "json",
    "sendBody": true,
    "bodyParameters": {
      "parameters": [
        {
          "name": "model_name",
          "value": "stabilityai/stable-diffusion-xl-base-1.0"
        },
        {
          "name": "prompt",
          "value": "={{ $json.prompt }}"
        },
        {
          "name": "negative_prompt",
          "value": "={{ $json.negative_prompt || 'ugly, deformed, noisy, blurry, distorted, out of focus' }}"
        },
        {
          "name": "width",
          "value": "={{ $json.width || 1024 }}"
        },
        {
          "name": "height",
          "value": "={{ $json.height || 1024 }}"
        },
        {
          "name": "num_images",
          "value": "={{ $json.num_images || 1 }}"
        },
        {
          "name": "scheduler",
          "value": "={{ $json.scheduler || 'dpmsolver++' }}"
        },
        {
          "name": "num_inference_steps",
          "value": "={{ $json.num_inference_steps || 25 }}"
        }
      ]
    },
    "options": {}
  },
  "name": "Generate Image with Stable Diffusion XL",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [600, 300],
  "credentials": {
    "httpHeaderAuth": {
      "id": "YOUR_CREDENTIALS_ID",
      "name": "Fal.AI API Access"
    }
  }
}
```

#### Использование DALL-E 3

```json
{
  "parameters": {
    "method": "POST",
    "url": "https://api.fal.ai/dalle-3",
    "authentication": "headerAuth",
    "contentType": "json",
    "sendBody": true,
    "bodyParameters": {
      "parameters": [
        {
          "name": "prompt",
          "value": "={{ $json.prompt }}"
        },
        {
          "name": "size",
          "value": "={{ $json.size || '1024x1024' }}"
        },
        {
          "name": "quality",
          "value": "={{ $json.quality || 'standard' }}"
        },
        {
          "name": "style",
          "value": "={{ $json.style || 'vivid' }}"
        }
      ]
    },
    "options": {}
  },
  "name": "Generate Image with DALL-E 3",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [600, 400],
  "credentials": {
    "httpHeaderAuth": {
      "id": "YOUR_CREDENTIALS_ID",
      "name": "Fal.AI API Access"
    }
  }
}
```

### 3. Создание HTTP Request ноды для генерации видео

```json
{
  "parameters": {
    "method": "POST",
    "url": "https://api.fal.ai/video-generation",
    "authentication": "headerAuth",
    "contentType": "json",
    "sendBody": true,
    "bodyParameters": {
      "parameters": [
        {
          "name": "model_name",
          "value": "zeroscope-v2-xl"
        },
        {
          "name": "prompt",
          "value": "={{ $json.prompt }}"
        },
        {
          "name": "negative_prompt",
          "value": "={{ $json.negative_prompt || 'poor quality, low resolution, blurry' }}"
        },
        {
          "name": "num_frames",
          "value": "={{ $json.num_frames || 24 }}"
        },
        {
          "name": "fps",
          "value": "={{ $json.fps || 8 }}"
        }
      ]
    },
    "options": {}
  },
  "name": "Generate Video",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [600, 500],
  "credentials": {
    "httpHeaderAuth": {
      "id": "YOUR_CREDENTIALS_ID",
      "name": "Fal.AI API Access"
    }
  }
}
```

## Создание MediaAgent для работы с Fal.AI

### 1. Создание рабочего потока MediaAgent

Создайте новый рабочий поток с именем "MediaAgent":

```
Workflow Trigger → LLM Agent → Fal.AI HTTP Requests → Post-Processing → Return Result
```

### 2. Настройка LLM Agent

```json
{
  "parameters": {
    "promptType": "define",
    "text": "={{ $json.query }}",
    "options": {
      "systemMessage": "# MediaAgent\n\nТы специализированный агент для создания медиа-контента. Твоя задача - генерировать высококачественные изображения и видео на основе запросов пользователя.\n\n## Доступные инструменты\n\n1. generateImageSDXL - Генерация изображений с помощью Stable Diffusion XL\n2. generateImageDALLE - Генерация изображений с помощью DALL-E 3\n3. generateVideo - Генерация короткого видео\n4. saveMedia - Сохранение медиа-контента\n\n## Правила работы\n\n1. Анализируй запрос пользователя и определяй, какой тип медиа-контента нужно создать\n2. Формируй детальный и точный промпт для генерации\n3. Выбирай подходящий инструмент в зависимости от типа контента и требований\n4. Создавай несколько вариантов для выбора, если это запрошено\n5. Сохраняй созданный контент для дальнейшего использования\n\n## Формирование промптов\n\n### Для изображений:\n- Будь конкретным и детальным в описании\n- Указывай стиль, настроение, освещение, композицию\n- Используй отрицательные промпты для исключения нежелательных элементов\n\n### Для видео:\n- Описывай последовательность действий\n- Указывай движение и изменения в сцене\n- Учитывай ограничения (краткость видео)\n\n## Примеры промптов\n\n### Хороший промпт для изображения:\n\"Высококачественный фотореалистичный портрет молодой женщины-ученого в лаборатории, работающей с голографическими дисплеями, голубое освещение, сосредоточенное выражение лица, детализированное окружение, 8k, кинематографическое освещение\"\n\n### Хороший промпт для видео:\n\"Закат над океаном, солнце медленно погружается в воду, меняется освещение от золотистого к пурпурному, небольшие волны разбиваются о берег, спокойная атмосфера\"\n\n## Работа с пользователем\n\n1. Проси уточнения, если запрос неясен\n2. Предлагай альтернативы при необходимости\n3. Объясняй свои решения\n\nТекущая дата: {{ $now }}"
    }
  },
  "name": "Media Agent",
  "type": "@n8n/n8n-nodes-langchain.agent",
  "typeVersion": 1.6
}
```

### 3. Создание tool nodes для MediaAgent

#### generateImageSDXL Tool

```json
{
  "parameters": {
    "toolDescription": "Generate image using Stable Diffusion XL. Required parameters: prompt (detailed description of the image). Optional parameters: negative_prompt, width, height, num_images.",
    "method": "POST",
    "url": "https://api.fal.ai/text-to-image",
    "authentication": "predefinedCredentialType",
    "nodeCredentialType": "httpHeaderAuth",
    "sendBody": true,
    "specifyBody": "json",
    "jsonBody": "{\n  \"model_name\": \"stabilityai/stable-diffusion-xl-base-1.0\",\n  \"prompt\": \"{prompt}\",\n  \"negative_prompt\": \"{negative_prompt}\",\n  \"width\": {width},\n  \"height\": {height},\n  \"num_images\": {num_images},\n  \"scheduler\": \"dpmsolver++\",\n  \"num_inference_steps\": 25\n}",
    "placeholderDefinitions": {
      "values": [
        {
          "name": "prompt",
          "description": "Detailed description of the image to generate",
          "type": "string"
        },
        {
          "name": "negative_prompt",
          "description": "What to avoid in the image (default: ugly, deformed, noisy, blurry, distorted, out of focus)",
          "type": "string",
          "default": "ugly, deformed, noisy, blurry, distorted, out of focus"
        },
        {
          "name": "width",
          "description": "Image width in pixels (default: 1024)",
          "type": "number",
          "default": 1024
        },
        {
          "name": "height",
          "description": "Image height in pixels (default: 1024)",
          "type": "number",
          "default": 1024
        },
        {
          "name": "num_images",
          "description": "Number of images to generate (default: 1)",
          "type": "number",
          "default": 1
        }
      ]
    }
  },
  "name": "Generate Image SDXL",
  "type": "@n8n/n8n-nodes-langchain.toolHttpRequest",
  "typeVersion": 1.1
}
```

#### saveMedia Tool

```json
{
  "parameters": {
    "name": "saveMedia",
    "description": "Save media content to Google Drive. Required parameters: fileUrl (URL of the media file), fileName (name to save the file as), folder (folder ID to save in).",
    "workflowId": {
      "value": "YOUR_WORKFLOW_ID",
      "mode": "list",
      "cachedResultName": "Save Media To Drive"
    },
    "workflowInputs": {
      "mappingMode": "defineBelow",
      "value": {
        "fileUrl": "={{ $fromAI(\"fileUrl\") }}",
        "fileName": "={{ $fromAI(\"fileName\") }}",
        "folder": "={{ $fromAI(\"folder\", \"Folder ID to save to\") }}"
      }
    }
  },
  "name": "Save Media",
  "type": "@n8n/n8n-nodes-langchain.toolWorkflow",
  "typeVersion": 2
}
```

### 4. Интеграция MediaAgent с основным рабочим потоком

1. Добавьте ноду "@n8n/n8n-nodes-langchain.toolWorkflow" в Main Orchestrator
2. Настройте ноду:
   ```json
   {
     "parameters": {
       "name": "mediaAgent",
       "description": "Используйте этот инструмент для создания медиа-контента (изображений и видео).",
       "workflowId": {
         "value": "YOUR_WORKFLOW_ID",
         "mode": "list",
         "cachedResultName": "MediaAgent"
       },
       "workflowInputs": {
         "mappingMode": "defineBelow",
         "value": {
           "query": "={{ $fromAI(\"query\", \"Запрос на создание медиа-контента\") }}"
         }
       }
     }
   }
   ```

3. Обновите системный промпт в Main Agent, добавив инструкции по использованию mediaAgent:

```
## Создание медиа-контента
Используй mediaAgent для создания изображений и видео:
- Для создания изображения: "Создай изображение [детальное описание]"
- Для создания видео: "Создай видео [детальное описание]"
- Указывай максимально подробные и детальные описания для получения качественного результата
```

## Рекомендации по оптимальному использованию Fal.AI

### 1. Оптимизация промптов для изображений

#### Структура эффективного промпта:
1. **Субъект** - что должно быть на изображении
2. **Стиль** - художественный стиль (фотореализм, поп-арт, аниме и т.д.)
3. **Настроение** - эмоциональная тональность изображения
4. **Освещение** - тип и качество освещения
5. **Композиция** - как элементы расположены в кадре
6. **Качество и детали** - уровень детализации, разрешение
7. **Отрицательный промпт** - чего следует избегать

#### Пример:
```
"Фотореалистичный портрет профессионального программиста, работающего за компьютером с несколькими мониторами, в современном офисе с панорамными окнами, сосредоточенное выражение лица, голубоватое освещение от экранов смешивается с теплым дневным светом из окон, детальное изображение кода на экранах, высокое разрешение, кинематографический стиль"
```

### 2. Оптимизация промптов для видео

#### Структура эффективного промпта:
1. **Сцена** - основное содержание видео
2. **Движение** - как происходит движение в кадре
3. **Изменения** - как меняется сцена со временем
4. **Временная шкала** - последовательность событий
5. **Стиль и настроение** - общая эстетика и эмоциональный тон

#### Пример:
```
"Таймлапс строительства небоскреба, начиная с фундамента и заканчивая готовым зданием, рабочие и техника движутся быстро, меняется время суток от рассвета до заката, постепенно растет высота здания, несколько этажей появляются за секунды, современный архитектурный стиль, энергичная атмосфера"
```

### 3. Сохранение и повторное использование успешных промптов

- Создайте библиотеку успешных промптов
- Категоризируйте промпты по типу контента
- Анализируйте результаты и оптимизируйте промпты

### 4. Мониторинг использования

- Регулярно проверяйте баланс и использование в Fal.AI
- Настройте уведомления о низком балансе
- Анализируйте соотношение качества и стоимости для разных моделей

## Устранение проблем

### 1. Проблема: Низкое качество сгенерированных изображений

**Решение**:
- Улучшите детализацию промпта
- Добавьте уточнения по стилю и качеству (например, "8k, высокое разрешение, фотореалистичный")
- Используйте более подробный отрицательный промпт
- Увеличьте число шагов инференса (num_inference_steps)

### 2. Проблема: Ошибка "Rate limit exceeded"

**Решение**:
- Реализуйте механизм повторных попыток с экспоненциальной задержкой
- Уменьшите частоту запросов
- Рассмотрите возможность перехода на более высокий тарифный план

### 3. Проблема: Не удается сохранить сгенерированные файлы

**Решение**:
- Проверьте разрешения для Google Drive
- Реализуйте альтернативный механизм сохранения (например, в локальное хранилище или Telegram)
- Проверьте формат URL и доступность файла

## Дополнительные ресурсы

- [Официальная документация Fal.AI](https://docs.fal.ai/)
- [Руководство по промптам для Stable Diffusion](https://stability.ai/ai-research/guide-to-stable-diffusion-prompting)
- [Галерея примеров](https://fal.ai/gallery)
- [Примеры интеграций с n8n](https://n8n.io/integrations/)
