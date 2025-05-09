# Установка и настройка телеграм-бота на n8n

Данное руководство содержит подробные инструкции по установке и настройке интеллектуального телеграм-бота на базе n8n.

## Требования к системе

### Минимальные требования
- VPS с как минимум 4 ГБ RAM и 2 CPU
- Операционная система: Ubuntu 20.04+ / Debian 11+ / CentOS 8+
- Node.js 18 LTS или выше
- npm 8 или выше
- Доступ по SSH с правами суперпользователя
- 20+ ГБ свободного дискового пространства

### Рекомендуемые требования
- VPS с 8+ ГБ RAM и 4+ CPU
- Операционная система: Ubuntu 22.04
- Node.js 20 LTS
- npm 10 или выше
- SSD-хранилище с 50+ ГБ свободного места
- Выделенный IP-адрес
- Настроенный домен с SSL-сертификатом

## Подготовка сервера

### 1. Обновление системы

```bash
# Обновление списка пакетов
sudo apt update

# Установка обновлений
sudo apt upgrade -y

# Установка необходимых зависимостей
sudo apt install -y curl wget git build-essential
```

### 2. Установка Node.js

```bash
# Установка NVM (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash

# Применение изменений в текущем сеансе
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"

# Установка Node.js 20 LTS
nvm install 20

# Проверка версии Node.js
node -v

# Проверка версии npm
npm -v
```

### 3. Настройка NGINX (для поддержки HTTPS)

```bash
# Установка NGINX
sudo apt install -y nginx

# Установка Certbot для SSL-сертификатов Let's Encrypt
sudo apt install -y certbot python3-certbot-nginx

# Настройка файрвола
sudo ufw allow 'Nginx Full'
sudo ufw allow ssh
sudo ufw enable
```

Создание конфигурационного файла NGINX:

```bash
sudo nano /etc/nginx/sites-available/n8n
```

Добавьте следующую конфигурацию (замените `your-domain.com` на ваш домен):

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Необходимо для WebSockets
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

Активация конфигурации:

```bash
sudo ln -s /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

Настройка SSL:

```bash
sudo certbot --nginx -d your-domain.com
```

## Установка n8n

### 1. Установка через npm

```bash
# Глобальная установка n8n
npm install -g n8n

# Проверка установки
n8n --version
```

### 2. Установка через Docker (альтернативный метод)

```bash
# Установка Docker
sudo apt install -y docker.io docker-compose

# Запуск Docker
sudo systemctl enable docker
sudo systemctl start docker

# Создание docker-compose.yml
mkdir n8n-bot
cd n8n-bot
nano docker-compose.yml
```

Содержимое docker-compose.yml:

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
      - N8N_HOST=your-domain.com
      - N8N_PORT=5678
      - N8N_ENCRYPTION_KEY=your-random-encryption-key
      - NODE_ENV=production
      - WEBHOOK_URL=https://your-domain.com/
      - N8N_EMAIL_MODE=smtp
      - N8N_SMTP_HOST=smtp.example.com
      - N8N_SMTP_PORT=587
      - N8N_SMTP_USER=your-email@example.com
      - N8N_SMTP_PASS=your-password
      - N8N_SMTP_SENDER=your-email@example.com
    volumes:
      - ~/.n8n:/home/node/.n8n
```

Запуск контейнера:

```bash
sudo docker-compose up -d
```

### 3. Настройка n8n как системной службы (для установки через npm)

Создание файла службы:

```bash
sudo nano /etc/systemd/system/n8n.service
```

Содержимое файла службы:

```ini
[Unit]
Description=n8n
After=network.target

[Service]
Type=simple
User=YOUR_USERNAME
ExecStart=/usr/bin/n8n start
WorkingDirectory=/home/YOUR_USERNAME
Environment=NODE_ENV=production
Environment=N8N_PROTOCOL=https
Environment=N8N_HOST=your-domain.com
Environment=N8N_PORT=5678
Environment=N8N_ENCRYPTION_KEY=your-random-encryption-key
Environment=WEBHOOK_URL=https://your-domain.com/
Environment=N8N_EMAIL_MODE=smtp
Environment=N8N_SMTP_HOST=smtp.example.com
Environment=N8N_SMTP_PORT=587
Environment=N8N_SMTP_USER=your-email@example.com
Environment=N8N_SMTP_PASS=your-password
Environment=N8N_SMTP_SENDER=your-email@example.com
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Активация и запуск службы:

```bash
sudo systemctl daemon-reload
sudo systemctl enable n8n
sudo systemctl start n8n
sudo systemctl status n8n
```

## Настройка базы данных

По умолчанию n8n использует SQLite, но для продакшн-окружения рекомендуется использовать более надежную СУБД, например PostgreSQL:

```bash
# Установка PostgreSQL
sudo apt install -y postgresql postgresql-contrib

# Вход в PostgreSQL
sudo -u postgres psql

# Создание базы данных и пользователя для n8n
CREATE USER n8nuser WITH PASSWORD 'your-secure-password';
CREATE DATABASE n8n OWNER n8nuser;
\q

# Добавление настроек PostgreSQL в конфигурацию n8n
# Для установки через npm добавьте в файл /etc/systemd/system/n8n.service:
Environment=DB_TYPE=postgresdb
Environment=DB_POSTGRESDB_HOST=localhost
Environment=DB_POSTGRESDB_PORT=5432
Environment=DB_POSTGRESDB_DATABASE=n8n
Environment=DB_POSTGRESDB_USER=n8nuser
Environment=DB_POSTGRESDB_PASSWORD=your-secure-password

# Перезапуск службы
sudo systemctl daemon-reload
sudo systemctl restart n8n
```

Для Docker установки добавьте эти переменные окружения в `docker-compose.yml`:

```yaml
- DB_TYPE=postgresdb
- DB_POSTGRESDB_HOST=postgres
- DB_POSTGRESDB_PORT=5432
- DB_POSTGRESDB_DATABASE=n8n
- DB_POSTGRESDB_USER=n8nuser
- DB_POSTGRESDB_PASSWORD=your-secure-password
```

И добавьте сервис PostgreSQL:

```yaml
postgres:
  image: postgres:15
  restart: always
  environment:
    - POSTGRES_USER=n8nuser
    - POSTGRES_PASSWORD=your-secure-password
    - POSTGRES_DB=n8n
  volumes:
    - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

## Настройка сервисов для RAG

### 1. Установка Zep (для хранения истории диалогов)

```bash
# Создание директории для Zep
mkdir -p ~/zep
cd ~/zep

# Создание docker-compose.yml для Zep
nano docker-compose.yml
```

Содержимое docker-compose.yml для Zep:

```yaml
version: '3'

services:
  zep:
    image: ghcr.io/getzep/zep:latest
    environment:
      - ZEP_MEMORY_STORE_POSTGRES_DSN=postgres://zepuser:your-secure-password@postgres:5432/zep?sslmode=disable
      - ZEP_OPENAI_API_KEY=your-openai-api-key
    ports:
      - "8000:8000"
    depends_on:
      - postgres
    networks:
      - zep-network
    restart: always

  postgres:
    image: postgres:15
    environment:
      - POSTGRES_USER=zepuser
      - POSTGRES_PASSWORD=your-secure-password
      - POSTGRES_DB=zep
    volumes:
      - zep-postgres-data:/var/lib/postgresql/data
    networks:
      - zep-network
    restart: always

networks:
  zep-network:

volumes:
  zep-postgres-data:
```

Запуск Zep:

```bash
sudo docker-compose up -d
```

### 2. Настройка Redis для кэширования (опционально)

```bash
# Установка Redis
sudo apt install -y redis-server

# Настройка Redis для прослушивания только локальных подключений
sudo nano /etc/redis/redis.conf
```

Найдите строку `bind 127.0.0.1 ::1` и убедитесь, что она не закомментирована.

```bash
# Перезапуск Redis
sudo systemctl restart redis-server

# Добавление Redis в автозагрузку
sudo systemctl enable redis-server
```

## Первоначальная настройка n8n

### 1. Доступ к веб-интерфейсу

После установки и настройки n8n, вы можете получить доступ к веб-интерфейсу по адресу:

```
https://your-domain.com
```

При первом входе вам будет предложено создать учетную запись администратора.

### 2. Настройка учетных данных

Для работы бота необходимо настроить учетные данные для внешних сервисов:

1. Перейдите в "Settings" > "Credentials"
2. Добавьте необходимые учетные данные, используя подробные инструкции из соответствующих разделов документации:
   - [Настройка Telegram Bot API](../api-keys/telegram-setup.md)
   - [Настройка OpenRouter](../api-keys/openrouter-setup.md)
   - [Настройка Fal.AI](../api-keys/falai-setup.md)
   - [Настройка Google Services](../api-keys/google-services-setup.md)
   - [Настройка Pinecone](../api-keys/pinecone-setup.md)

### 3. Импорт рабочих потоков

Для импорта рабочих потоков используйте следующие шаги:

1. Перейдите в "Workflows" в веб-интерфейсе n8n
2. Нажмите кнопку "Import from File"
3. Выберите файлы JSON с рабочими потоками из директории `/examples/workflows/` данного репозитория
4. После импорта проверьте настройки рабочих потоков и подключите соответствующие учетные данные

## Настройка автоматических резервных копий

Для создания регулярных резервных копий настроек и рабочих потоков n8n можно использовать cron:

```bash
# Открыть редактор cron
crontab -e

# Добавить задачу для ежедневного резервного копирования (для установки через npm)
0 2 * * * tar -czf /backup/n8n-backup-$(date +\%Y\%m\%d).tar.gz ~/.n8n && find /backup -name "n8n-backup-*.tar.gz" -mtime +30 -delete
```

Для Docker-установки:

```bash
# Создание скрипта резервного копирования
sudo nano /usr/local/bin/backup-n8n.sh
```

Содержимое скрипта:

```bash
#!/bin/bash
BACKUP_DIR="/backup"
TIMESTAMP=$(date +%Y%m%d)

# Создание директории для резервных копий
mkdir -p $BACKUP_DIR

# Резервное копирование данных n8n
sudo docker-compose exec -T postgres pg_dump -U n8nuser n8n > $BACKUP_DIR/n8n-db-$TIMESTAMP.sql

# Резервное копирование томов Docker
tar -czf $BACKUP_DIR/n8n-volumes-$TIMESTAMP.tar.gz ~/.n8n

# Удаление резервных копий старше 30 дней
find $BACKUP_DIR -name "n8n-db-*.sql" -mtime +30 -delete
find $BACKUP_DIR -name "n8n-volumes-*.tar.gz" -mtime +30 -delete
```

Назначение прав на исполнение и добавление в cron:

```bash
sudo chmod +x /usr/local/bin/backup-n8n.sh
sudo crontab -e

# Добавить задачу
0 2 * * * /usr/local/bin/backup-n8n.sh
```

## Мониторинг и обслуживание

### 1. Настройка мониторинга с использованием Prometheus и Grafana (опционально)

```bash
# Создание директории для мониторинга
mkdir -p ~/monitoring
cd ~/monitoring

# Создание docker-compose.yml
nano docker-compose.yml
```

Содержимое docker-compose.yml для мониторинга:

```yaml
version: '3'

services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus-data:/prometheus
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    restart: always

  grafana:
    image: grafana/grafana
    volumes:
      - grafana-data:/var/lib/grafana
    ports:
      - "3000:3000"
    restart: always

volumes:
  prometheus-data:
  grafana-data:
```

Создание конфигурации Prometheus:

```bash
mkdir -p ~/monitoring/prometheus
nano ~/monitoring/prometheus/prometheus.yml
```

Содержимое prometheus.yml:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'n8n'
    static_configs:
      - targets: ['your-domain.com:5678']
```

Запуск мониторинга:

```bash
cd ~/monitoring
sudo docker-compose up -d
```

### 2. Настройка автоматических обновлений

Для n8n, установленного через npm:

```bash
# Создание скрипта обновления
sudo nano /usr/local/bin/update-n8n.sh
```

Содержимое скрипта:

```bash
#!/bin/bash
# Обновление n8n
npm update -g n8n

# Перезапуск службы
sudo systemctl restart n8n
```

Назначение прав на исполнение и добавление в cron:

```bash
sudo chmod +x /usr/local/bin/update-n8n.sh
sudo crontab -e

# Добавить задачу для еженедельного обновления
0 4 * * 0 /usr/local/bin/update-n8n.sh
```

Для Docker-установки:

```bash
# Создание скрипта обновления
sudo nano /usr/local/bin/update-n8n-docker.sh
```

Содержимое скрипта:

```bash
#!/bin/bash
cd ~/n8n-bot
sudo docker-compose pull
sudo docker-compose up -d
```

Назначение прав на исполнение и добавление в cron:

```bash
sudo chmod +x /usr/local/bin/update-n8n-docker.sh
sudo crontab -e

# Добавить задачу для еженедельного обновления
0 4 * * 0 /usr/local/bin/update-n8n-docker.sh
```

## Защита и безопасность

### 1. Настройка файрвола

```bash
# Разрешение только необходимых портов
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

### 2. Настройка fail2ban для защиты от брутфорса

```bash
# Установка fail2ban
sudo apt install -y fail2ban

# Создание локальной конфигурации
sudo nano /etc/fail2ban/jail.local
```

Содержимое jail.local:

```ini
[DEFAULT]
bantime = 86400
findtime = 3600
maxretry = 5

[sshd]
enabled = true
```

Запуск fail2ban:

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### 3. Ограничение доступа к веб-интерфейсу n8n

Для ограничения доступа к веб-интерфейсу n8n можно настроить базовую аутентификацию в NGINX:

```bash
# Установка apache2-utils для создания файла паролей
sudo apt install -y apache2-utils

# Создание файла паролей
sudo htpasswd -c /etc/nginx/.htpasswd admin
```

Обновление конфигурации NGINX:

```bash
sudo nano /etc/nginx/sites-available/n8n
```

Добавьте следующие строки в блок `location /`:

```nginx
auth_basic "Restricted Area";
auth_basic_user_file /etc/nginx/.htpasswd;
```

Проверка и перезапуск NGINX:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

## Устранение проблем

### 1. Проверка логов

Для установки через npm:

```bash
# Просмотр логов системной службы n8n
sudo journalctl -u n8n -f
```

Для Docker-установки:

```bash
# Просмотр логов контейнера n8n
sudo docker-compose logs -f n8n
```

### 2. Проверка статуса служб

```bash
# Проверка статуса n8n (для системной службы)
sudo systemctl status n8n

# Проверка статуса NGINX
sudo systemctl status nginx

# Проверка статуса Docker (для Docker-установки)
sudo systemctl status docker
```

### 3. Распространенные проблемы и их решения

#### Проблема: n8n не запускается

**Решение**:
- Проверьте логи службы: `sudo journalctl -u n8n -f`
- Убедитесь, что все переменные окружения настроены правильно
- Проверьте права доступа к файлам и директориям

#### Проблема: Вебхуки не работают

**Решение**:
- Убедитесь, что настроен правильный URL вебхука в переменной `WEBHOOK_URL`
- Проверьте, что ваш домен доступен из интернета
- Проверьте настройки SSL и NGINX

#### Проблема: Telegram бот не получает сообщения

**Решение**:
- Проверьте, что вебхук Telegram настроен правильно
- Убедитесь, что API-ключ бота верный
- Проверьте логи n8n на наличие ошибок при взаимодействии с Telegram API

## Первоначальная настройка бота

После успешной установки и настройки всей инфраструктуры, необходимо выполнить следующие шаги для настройки бота:

1. **Создание бота в Telegram**:
   - Следуйте инструкциям в [руководстве по настройке Telegram Bot API](../api-keys/telegram-setup.md)
   - Сохраните полученный токен API

2. **Импорт и настройка рабочих потоков**:
   - Импортируйте основной рабочий поток (Main Orchestrator)
   - Импортируйте и настройте все вспомогательные рабочие потоки (Email Agent, Calendar Agent и т.д.)
   - Убедитесь, что все зависимости между рабочими потоками настроены правильно

3. **Проверка работоспособности**:
   - Отправьте тестовое сообщение боту в Telegram
   - Проверьте логи на наличие ошибок
   - Убедитесь, что бот отвечает и правильно обрабатывает команды

4. **Настройка системы RAG**:
   - Следуйте инструкциям в [руководстве по настройке RAG системы](../implementation/step-by-step/04-setup-rag-system.md)
   - Проверьте, что векторная база данных работает корректно

5. **Настройка резервного копирования**:
   - Настройте автоматическое резервное копирование настроек и рабочих потоков
   - Проверьте, что резервные копии создаются корректно

## Заключение

Установка и настройка интеллектуального телеграм-бота на базе n8n требует некоторых технических знаний, но после завершения настройки вы получите мощный инструмент, способный автоматизировать множество задач.

Для получения более подробной информации о настройке конкретных компонентов системы, обратитесь к соответствующим разделам документации:

- [Архитектура системы](../../ARCHITECTURE.md)
- [Настройка специализированных агентов](../workflows/)
- [Устранение проблем](../../troubleshooting/)

Если у вас возникли вопросы или проблемы при установке, обратитесь к разделу [Устранение проблем](../../troubleshooting/common-issues.md) или создайте новый issue в репозитории проекта.