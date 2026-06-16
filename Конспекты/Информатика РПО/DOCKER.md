


#  Docker: Полное руководство и шпаргалка

Docker — это платформа для упаковки приложения и всех его зависимостей в стандартизированный модуль (контейнер), который гарантированно работает одинаково в любой среде.


## Архитектура (Mermaid)

Как это работает
Ниже представлена базовая схема жизненного цикла Docker-объектов.

```mermaid
graph TD
    A[Dockerfile<br/>Инструкции] -->|docker build| B(Image<br/>Образ)
    B -->|docker run| C(Container<br/>Контейнер]
    C -->|docker commit| B
    B <-->|push / pull| D[(Registry<br/>Реестр, напр. Docker Hub)]
    E[Volume<br/>Том] -.->|монтирование| C
    F[Network<br/>Сеть] -.->|связь| C
```



##  Шпаргалка по командам

###  Образы (Images)
| Команда                       | Описание                                                       |
| :---------------------------- | :------------------------------------------------------------- |
| `docker build -t myapp:1.0 .` | Собрать образ с тегом из текущей директории (`.`)              |
| `docker images`               | Показать все локальные образы                                  |
| `docker rmi <image_id>`       | Удалить образ (сначала остановите использующие его контейнеры) |
| `docker pull ubuntu:22.04`    | Скачать образ из реестра                                       |
| `docker inspect <image>`      | Показать детальную JSON-информацию об образе                   |

###  Контейнеры (Containers)
| Команда | Описание |
| :--- | :--- |
| `docker run -d -p 8080:80 --name web nginx` | Запустить в фоне (`-d`), пробросить порт 8080 хоста на 80 контейнера, дать имя |
| `docker ps` | Показать только **запущенные** контейнеры |
| `docker ps -a` | Показать **все** контейнеры (включая остановленные) |
| `docker stop <id>` / `docker start <id>` | Остановить / запустить контейнер |
| `docker rm <id>` | Удалить контейнер (добавьте `-f` для принудительного удаления) |
| `docker exec -it <id> /bin/bash` | Войти внутрь работающего контейнера (используйте `/bin/sh` для Alpine) |
| `docker logs -f --tail 100 <id>` | Показать последние 100 строк логов и следить за новыми (`-f`) |

###  Тома и Сети (Volumes & Networks)
| Команда | Описание |
| :--- | :--- |
| `docker volume ls` | Показать все тома |
| `docker volume create my_data` | Создать именованный том |
| `docker network ls` | Показать сети |
| `docker network create my_net` | Создать пользовательскую сеть (контейнеры в ней видят друг друга по имени) |

###  Очистка (ОСТОРОЖНО)
```bash
docker system prune          # Удалить остановленные контейнеры, пустые сети и кэш
docker system prune -a       # + удалить все образы, не связанные с запущенными контейнерами
docker system prune --volumes# + удалить все неиспользуемые тома (данные будут потеряны!)
```

---

##  Шаблоны конфигураций

### 1. Оптимальный `Dockerfile` (на примере Node.js/Python)
```dockerfile
# 1. Используем конкретную версию и легкий базовый образ (alpine/slim)
FROM node:18-alpine

# 2. Создаем непривилегированного пользователя для безопасности
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# 3. Задаем рабочую директорию
WORKDIR /app

# 4. Копируем ТОЛЬКО файлы зависимостей (используем кэш слоев Docker)
COPY package*.json ./

# 5. Устанавливаем зависимости и очищаем кэш в одном слое
RUN npm ci --only=production && rm -rf /root/.npm

# 6. Копируем исходный код
COPY --chown=appuser:appgroup . .

# 7. Переключаемся на непривилегированного пользователя
USER appuser

# 8. Указываем порт (только для документации)
EXPOSE 3000

# 9. Команда запуска (предпочтительно в формате exec, а не shell)
CMD ["node", "server.js"]
Не забудь создать файл `.dockerignore`
Добавь туда: `node_modules`, `.git`, `.env`, `*.log`, `.DS_Store`.

### 2. Продакшен-готовый `docker-compose.yml`
```yaml
version: '3.8'

services:
  # Основное приложение
  app:
    build: 
      context: .
      dockerfile: Dockerfile
    container_name: my_web_app
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://dbuser:dbpass@database:5432/appdb
    depends_on:
      database:
        condition: service_healthy # Ждет, пока БД не станет доступной
    networks:
      - app_network
    volumes:
      - app_logs:/app/logs # Именованный том для логов

  # База данных
  database:
    image: postgres:15-alpine
    container_name: my_postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: dbuser
      POSTGRES_PASSWORD: dbpass
      POSTGRES_DB: appdb
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U dbuser -d appdb"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - postgres_data:/var/lib/postgresql/data # Сохранение данных на хосте
    networks:
      - app_network

# Именованные тома (управляются Docker, хранятся в /var/lib/docker/volumes)
volumes:
  postgres_data:
  app_logs:

# Пользовательская сеть для изоляции
networks:
  app_network:
    driver: bridge
```



##  Best Practices 
Чек-лист перед деплоем
Конкретные теги**: Никогда не используй `latest` в продакшене. Пиши `python:3.11.4-slim`.
Многоступенчатая сборка (Multi-stage): Для Go, Rust, Java используй отдельный этап `build`, чтобы в финальный образ попал только скомпилированный бинарник, а не весь компилятор.
Один процесс на контейнер: Не запускай SSH-сервер, cron и приложение в одном контейнере. Используй Supervisor или, что лучше, раздели на несколько контейнеров в Docker Compose.
Переменные окружения: Никогда не хардкодь пароли в Dockerfile. Передавай их через `.env` файл или секреты.

