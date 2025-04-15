## 🧠 Что такое Docker?

**Docker** — это инструмент для упаковки приложения с его зависимостями в **контейнеры**, которые изолированы от системы и одинаково работают в любом окружении.

---

## 📦 Основные понятия

| Термин         | Описание |
|----------------|----------|
| **Image**      | Шаблон (образ) контейнера |
| **Container**  | Запущенный инстанс образа |
| **Dockerfile** | Скрипт для сборки image |
| **Volume**     | Постоянное хранилище вне контейнера |
| **Network**    | Связь между контейнерами |
| **Registry**   | Хранилище образов (Docker Hub, GitLab, ECR) |

---

## 🛠️ Основные команды

### 📄 Работа с образами
```bash
docker build -t my-app .       # собрать образ
docker images                  # список образов
docker rmi my-app              # удалить образ
```

### 📦 Работа с контейнерами
```bash
docker run -d -p 80:80 nginx       # запустить контейнер
docker ps                          # список работающих
docker exec -it <id> bash          # войти в контейнер
docker stop <id> && docker rm <id> # остановить и удалить
```

### 🔄 Обновление и логирование
```bash
docker logs -f <id>          # посмотреть логи
docker restart <id>          # перезапустить
docker stats                 # использование ресурсов
```

---

## 📑 Dockerfile: пример

```Dockerfile
FROM golang:1.20-alpine
WORKDIR /app
COPY . .
RUN go build -o main .
CMD ["./main"]
```

> `CMD` запускает процесс внутри контейнера. `ENTRYPOINT` можно использовать, если хочешь всегда запускать определённый бинарь.

---

## 🔧 Docker Compose

Позволяет запускать **мультиконтейнерное окружение**:

### `docker-compose.yml`

```yaml
version: '3.8'
services:
  web:
    image: my-web
    ports:
      - "8080:8080"
    depends_on:
      - db
  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: example
```

Команды:
```bash
docker-compose up -d
docker-compose down
```

---

## 📚 Volume, Network, ENV

### Volume
```bash
docker run -v /host:/container my-app
```

- Используется для сохранения данных вне контейнера
- Часто используют для БД

### Network
```bash
docker network create my-net
docker run --network=my-net ...
```

- Контейнеры в одной сети могут обращаться по имени

### ENV
```Dockerfile
ENV PORT=8080
```
```bash
docker run -e PORT=8080 my-app
```

---

## 🧱 Слои и кэш

- Каждый шаг Dockerfile создаёт **слой** (layer)
- Docker кэширует слои, если они не изменялись

> Старайся сначала добавлять редко изменяемое (например, зависимости), а потом код.

---

## 📦 Виды сетей в Docker

| Тип сети    | Описание |
|-------------|----------|
| **bridge**  | По умолчанию. Контейнеры в одной bridge-сети видят друг друга |
| **host**    | Контейнер использует сеть хоста |
| **none**    | Полностью изолированная |
| **overlay** | Для Docker Swarm |

---

## 🧠 Частые вопросы на собесе

- Чем контейнер отличается от виртуальной машины?
- Как устроен Docker под капотом (namespaces, cgroups)?
- Как работает слоистость образов?
- Как уменьшить размер образа?
- Как использовать multi-stage build?
- Как настраивать Volume для сохранения данных?
- Что будет при падении контейнера?
- Как настроить `healthcheck`?
- Что такое ENTRYPOINT vs CMD?

---

## ⚡ Советы

- Используй **alpine**-образы (маленькие)
- Используй **`.dockerignore`** — как `.gitignore` для Docker
- Оптимизируй Dockerfile: зависимости отдельно, кэш слоёв
- Для прода — только нужные бинарники, никаких dev tools