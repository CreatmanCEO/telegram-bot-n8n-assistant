# Установка и настройка n8n для телеграм-бота

В этом руководстве подробно описаны все шаги по установке и настройке n8n на VPS для развертывания телеграм-бота.

## Предварительные требования

### Минимальные системные требования

- **ОС**: Ubuntu 20.04 LTS или новее
- **CPU**: 2+ ядра
- **RAM**: Минимум 4 ГБ (рекомендуется 8 ГБ для продуктивной работы)
- **Дисковое пространство**: Минимум 20 ГБ SSD
- **Сеть**: Стабильное интернет-соединение, доступное из внешнего мира
- **Домен**: Рекомендуется для настройки HTTPS (не обязательно, но крайне желательно)

### Необходимое программное обеспечение

- Node.js (версия 16 или новее)
- npm
- Docker и Docker Compose (опционально, но рекомендуется)
- Nginx (для проксирования запросов)
- Certbot (для получения и настройки SSL-сертификатов)

## Установка n8n

### Метод 1: Установка через npm (рекомендуется для разработки)

```bash
# Установка n8n глобально через npm
npm install n8n -g

# Запуск n8n
n8n
```

### Метод 2: Установка через Docker (рекомендуется для продакшена)

1. Создайте директорию для данных n8n:

```bash
mkdir -p /opt/n8n/data
```

2. Создайте файл `docker-compose.yml`:

```bash
mkdir -p /opt/n8n
cd /opt/n8n
nano docker-compose.yml
```

3. Добавьте следующий контент в файл `docker-compose.yml`:

```yaml
version: '3'

services:
  n8n:
    image: n8nio/n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_PROTOCOL=https
      - N8N_HOST=n8n.yourdomain.com  # Замените на ваш домен
      - N8N_PORT=5678
      - N8N_ENCRYPTION_KEY=your-secret-encryption-key  # Замените на ваш ключ шифрования
      - N8N_DISABLE_PRODUCTION_MAIN_PROCESS=false
      - NODE_ENV=production
      - WEBHOOK_URL=https://n8n.yourdomain.com  # Замените на ваш домен
      # Добавьте дополнительные переменные окружения здесь
    volumes:
      - /opt/n8n/data:/home/node/.n8n
    networks:
      - n8n_network

networks:
  n8n_network:
    driver: bridge
```

4. Запустите n8n с помощью Docker Compose:

```bash
cd /opt/n8n
docker-compose up -d
```

### Метод 3: Установка через скрипт для продакшн-окружения

1. Создайте bash-скрипт с автоматической настройкой n8n:

```bash
nano install-n8n.sh
```

2. Добавьте следующее содержимое в скрипт:

```bash
#!/bin/bash

# Обновление системы
apt update && apt upgrade -y

# Установка необходимых зависимостей
apt install -y curl build-essential git

# Установка Node.js и npm
curl -fsSL https://deb.nodesource.com/setup_16.x | bash -
apt install -y nodejs

# Проверка версий
node -v
npm -v

# Установка n8n
npm install n8n -g

# Создание пользователя для n8n
useradd -m -d /opt/n8n -s /bin/bash n8n

# Создание директории для данных
mkdir -p /opt/n8n/data
chown -R n8n:n8n /opt/n8n

# Создание systemd сервиса
cat > /etc/systemd/system/n8n.service << 'EOL'
[Unit]
Description=n8n
After=network.target

[Service]
Type=simple
User=n8n
WorkingDirectory=/opt/n8n
ExecStart=/usr/bin/n8n start
Restart=always
RestartSec=10
Environment=N8N_PROTOCOL=https
Environment=N8N_HOST=n8n.yourdomain.com
Environment=N8N_PORT=5678
Environment=N8N_ENCRYPTION_KEY=your-secret-encryption-key
Environment=N8N_DISABLE_PRODUCTION_MAIN_PROCESS=false
Environment=NODE_ENV=production
Environment=WEBHOOK_URL=https://n8n.yourdomain.com

[Install]
WantedBy=multi-user.target
EOL

# Обновление и запуск сервиса
systemctl daemon-reload
systemctl enable n8n
systemctl start n8n

echo "n8n установлен и запущен как системный сервис"
```

3. Сделайте скрипт исполняемым и запустите его:

```bash
chmod +x install-n8n.sh
sudo ./install-n8n.sh
```

## Настройка Nginx для проксирования запросов

1. Установите Nginx:

```bash
sudo apt update
sudo apt install nginx
```

2. Создайте конфигурационный файл для n8n:

```bash
sudo nano /etc/nginx/sites-available/n8n
```

3. Добавьте следующий конфигурационный код:

```nginx
server {
    listen 80;
    server_name n8n.yourdomain.com;  # Замените на ваш домен

    location / {
        proxy_pass http://localhost:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket поддержка
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Долгие соединения
        proxy_read_timeout 3600;
        proxy_send_timeout 3600;
    }
}
```

4. Создайте символическую ссылку и проверьте конфигурацию:

```bash
sudo ln -s /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
sudo nginx -t
```

5. Перезапустите Nginx:

```bash
sudo systemctl restart nginx
```

## Настройка SSL с Let's Encrypt

1. Установите Certbot:

```bash
sudo apt install certbot python3-certbot-nginx
```

2. Получите SSL-сертификат:

```bash
sudo certbot --nginx -d n8n.yourdomain.com
```

3. Следуйте инструкциям на экране и выберите перенаправление всего HTTP-трафика на HTTPS.

4. Certbot автоматически обновит конфигурацию Nginx для использования SSL.

## Настройка n8n после установки

### Первый вход и настройка безопасности

1. Откройте n8n в браузере по адресу `https://n8n.yourdomain.com`

2. Создайте учетную запись администратора:
   - Email
   - Пароль
   - Имя

3. Настройка дополнительных параметров безопасности:
   - Перейдите в "Settings" > "Security"
   - Настройте ограничения доступа по IP (опционально)
   - Настройте дополнительные параметры безопасности по необходимости

### Настройка переменных окружения

Для настройки переменных окружения, которые будут доступны в рабочих потоках, создайте или отредактируйте файл `.env` в директории данных n8n:

```bash
# Для установки через npm
nano ~/.n8n/.env

# Для установки через Docker
nano /opt/n8n/data/.env
```

Добавьте необходимые переменные окружения:

```
# API ключи для различных сервисов
OPENAI_API_KEY=your_openai_api_key
OPENROUTER_API_KEY=your_openrouter_api_key
PINECONE_API_KEY=your_pinecone_api_key
FALAI_API_KEY=your_falai_api_key

# Настройки Telegram
TELEGRAM_BOT_TOKEN=your_telegram_bot_token
TELEGRAM_CHAT_ID=your_telegram_chat_id

# Настройки для других сервисов
...
```

После добавления или изменения переменных окружения, перезапустите n8n:

```bash
# Для npm
n8n restart

# Для Docker
cd /opt/n8n
docker-compose restart

# Для systemd
sudo systemctl restart n8n
```

## Настройка хранилища для рабочих потоков

По умолчанию n8n хранит рабочие потоки и учетные данные в SQLite базе данных. Для продакшн-окружения рекомендуется использовать более надежное хранилище, например PostgreSQL или MySQL.

### Настройка PostgreSQL

1. Установите PostgreSQL:

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

2. Создайте пользователя и базу данных для n8n:

```bash
sudo -u postgres psql
```

В консоли PostgreSQL выполните:

```sql
CREATE USER n8n WITH PASSWORD 'your_password';
CREATE DATABASE n8n OWNER n8n;
GRANT ALL PRIVILEGES ON DATABASE n8n TO n8n;
\q
```

3. Обновите конфигурацию n8n для использования PostgreSQL:

Для установки через npm добавьте следующие переменные окружения в `.env`:

```
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=localhost
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=your_password
```

Для установки через Docker обновите `docker-compose.yml`:

```yaml
services:
  n8n:
    # ... остальная конфигурация ...
    environment:
      # ... остальные переменные окружения ...
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=your_password
    depends_on:
      - postgres

  postgres:
    image: postgres:13
    restart: always
    environment:
      - POSTGRES_USER=n8n
      - POSTGRES_PASSWORD=your_password
      - POSTGRES_DB=n8n
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - n8n_network

volumes:
  postgres_data:
```

4. Перезапустите n8n для применения изменений.

## Установка дополнительных узлов n8n

n8n поддерживает расширение функциональности с помощью дополнительных узлов (nodes). Для нашего телеграм-бота потребуются дополнительные узлы для интеграции с LangChain и другими сервисами.

### Установка узлов через npm

```bash
# Для npm
cd ~/.n8n
npm install @n8n/n8n-nodes-langchain

# Для Docker
docker exec -it n8n-container bash
cd /home/node/.n8n
npm install @n8n/n8n-nodes-langchain
```

### Установка через n8n CLI

```bash
# Для npm
n8n install @n8n/n8n-nodes-langchain

# Для Docker
docker exec -it n8n-container n8n install @n8n/n8n-nodes-langchain
```

## Настройка автоматического резервного копирования

Для защиты ваших рабочих потоков и данных от потери рекомендуется настроить автоматическое резервное копирование.

### Создание скрипта резервного копирования

1. Создайте скрипт для резервного копирования:

```bash
nano /opt/n8n/backup.sh
```

2. Добавьте следующее содержимое:

```bash
#!/bin/bash

# Настройки
BACKUP_DIR="/opt/n8n/backups"
DATE=$(date +"%Y%m%d_%H%M%S")
N8N_DATA_DIR="/opt/n8n/data"

# Создание директории для бэкапов, если она не существует
mkdir -p $BACKUP_DIR

# Остановка n8n для создания консистентного бэкапа (опционально)
# systemctl stop n8n

# Создание архива с данными n8n
tar -czf $BACKUP_DIR/n8n_backup_$DATE.tar.gz -C $N8N_DATA_DIR .

# Запуск n8n, если он был остановлен
# systemctl start n8n

# Очистка старых бэкапов (оставляем только последние 7)
find $BACKUP_DIR -type f -name "n8n_backup_*.tar.gz" -mtime +7 -delete

echo "Резервное копирование n8n завершено: $BACKUP_DIR/n8n_backup_$DATE.tar.gz"
```

3. Сделайте скрипт исполняемым:

```bash
chmod +x /opt/n8n/backup.sh
```

### Настройка автоматического выполнения через cron

1. Откройте crontab:

```bash
crontab -e
```

2. Добавьте следующую строку для выполнения резервного копирования ежедневно в 2:00:

```
0 2 * * * /opt/n8n/backup.sh >> /var/log/n8n_backup.log 2>&1
```

## Мониторинг и обновление

### Настройка мониторинга

Используйте стандартные инструменты мониторинга, такие как:

1. **Проверка статуса сервиса:**

```bash
systemctl status n8n
```

2. **Просмотр логов:**

```bash
# Для systemd
journalctl -u n8n -f

# Для Docker
docker logs -f n8n
```

### Обновление n8n

#### Обновление через npm:

```bash
# Обновление глобальной установки n8n
npm update -g n8n

# Перезапуск сервиса
systemctl restart n8n
```

#### Обновление через Docker:

```bash
# Перейдите в директорию с docker-compose.yml
cd /opt/n8n

# Остановите текущий контейнер
docker-compose down

# Получите последний образ
docker-compose pull

# Запустите обновленный контейнер
docker-compose up -d
```

## Оптимизация производительности

### Настройка Node.js для лучшей производительности

Создайте файл `.n8nrc.js` в корневой директории n8n:

```bash
nano /opt/n8n/.n8nrc.js
```

Добавьте следующий код:

```javascript
module.exports = {
  node: {
    maxExecutionTimeout: 300000, // 5 минут максимальное время выполнения
    memoryAllocation: 4096, // Выделение памяти (в МБ)
    concurrency: 10 // Максимальное количество одновременных выполнений
  }
};
```

### Настройка лимитов для n8n в systemd

Отредактируйте файл сервиса n8n:

```bash
sudo nano /etc/systemd/system/n8n.service
```

Добавьте следующие строки в секцию `[Service]`:

```
MemoryLimit=8G
CPUQuota=80%
```

Обновите сервис:

```bash
sudo systemctl daemon-reload
sudo systemctl restart n8n
```

## Устранение распространенных проблем

### Проблема: n8n не запускается

**Возможные решения:**
1. Проверьте логи для выявления ошибок:
   ```bash
   journalctl -u n8n -n 100
   ```

2. Проверьте наличие всех необходимых переменных окружения:
   ```bash
   cat /etc/systemd/system/n8n.service | grep Environment
   ```

3. Проверьте права доступа:
   ```bash
   ls -la /opt/n8n
   sudo chown -R n8n:n8n /opt/n8n
   ```

### Проблема: Webhook не работает

**Возможные решения:**
1. Проверьте настройки Nginx:
   ```bash
   sudo nginx -t
   ```

2. Проверьте, что порт открыт:
   ```bash
   sudo ufw status
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
   ```

3. Проверьте настройки переменных окружения для webhook URL:
   ```bash
   grep -r "WEBHOOK_URL" /opt/n8n
   ```

### Проблема: Высокое потребление ресурсов

**Возможные решения:**
1. Проверьте использование ресурсов:
   ```bash
   top
   htop
   ```

2. Настройте ограничения ресурсов для Docker:
   ```yaml
   services:
     n8n:
       # ... остальная конфигурация ...
       deploy:
         resources:
           limits:
             cpus: '2'
             memory: 4g
   ```

3. Оптимизируйте рабочие потоки, избегая бесконечных циклов и неэффективных операций.

## Заключение

После выполнения всех описанных выше шагов ваша инсталляция n8n будет готова для развертывания телеграм-бота. Для настройки конкретных рабочих потоков и интеграций обратитесь к соответствующим разделам документации.

Важные моменты для проверки перед началом работы:
1. n8n успешно запущен и доступен через HTTPS
2. Все необходимые API-ключи настроены как переменные окружения
3. Резервное копирование настроено и работает корректно
4. Мониторинг настроен для отслеживания состояния системы

После установки и настройки n8n, следующим шагом будет [настройка Telegram Bot API](../api-keys/telegram-setup.md) и [создание основного рабочего потока](../workflows/main-orchestrator.md) для вашего бота.