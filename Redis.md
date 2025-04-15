## 🔥 **1. Что такое Redis?**

Redis (Remote Dictionary Server) — это **in-memory NoSQL key-value хранилище** с возможностью работы с различными структурами данных.

- Очень быстрое (данные в памяти)
- Поддерживает **персистентность**
- Часто используется как **кэш**, **брокер сообщений**, **база временных данных**, **лок**, **счётчик**

---

## ⚙️ **2. Основные типы данных**

| Тип данных     | Описание | Пример |
|----------------|----------|--------|
| **String**     | Простой текст или бинарные данные | `SET key "value"` |
| **List**       | Упорядоченный список значений | `LPUSH`, `RPUSH`, `LRANGE` |
| **Set**        | Неупорядоченное множество уникальных элементов | `SADD`, `SMEMBERS` |
| **Sorted Set** | Множество с приоритетами (score) | `ZADD`, `ZRANGE` |
| **Hash**       | Ассоциативный массив (ключ -> значение) | `HSET`, `HGET` |
| **Bitmap**     | Побитовые операции (флаги, посещения) | `SETBIT`, `GETBIT` |
| **HyperLogLog**| Приближённый подсчёт уникальных значений | `PFADD`, `PFCOUNT` |
| **Stream**     | Поток сообщений | `XADD`, `XREAD`, `XGROUP` |

---

## 💾 **3. Персистентность (Persistence)**

| Тип           | Описание |
|---------------|----------|
| **RDB (Snapshot)** | Создаёт снапшоты всей БД в файл (быстро, но не real-time) |
| **AOF (Append Only File)** | Записывает каждую операцию (надёжнее, медленнее) |
| **Hybrid** | Можно комбинировать оба метода |

**Команды**:  
- `SAVE`, `BGSAVE` — ручной дамп  
- `BGREWRITEAOF` — переписывает AOF-файл компактно  

---

## ⚡ **4. Expire / TTL / Кэш**

Redis отлично подходит для **кэширования**:

```bash
SET key value EX 60  # Установить TTL 60 секунд
GET key
TTL key              # Проверить сколько осталось
```

Также:  
- `PERSIST key` — убрать TTL  
- `EXPIRE`, `PEXPIRE` — установить TTL

---

## 🧠 **5. Pub/Sub в Redis**

Для обмена сообщениями между сервисами (не хранится):

```bash
SUBSCRIBE mychannel
PUBLISH mychannel "hello"
```

Подходит для простых сценариев, **не гарантирует доставку** если клиент отключился.

---

## 🪄 **6. Redis Streams**

Современный механизм очередей / логов:

```bash
XADD mystream * name Alice age 21
XREAD COUNT 2 STREAMS mystream 0
```

- Можно использовать как **Kafka-подобный поток** (с группами потребителей)
- Группы: `XGROUP`, `XREADGROUP`

---

## 🧪 **7. Транзакции и Lua**

- **MULTI/EXEC** — простая транзакция
- **WATCH** — optimistic lock (отмена при изменении)
- **Lua-скрипты** — атомарные, эффективные

```bash
MULTI
INCR counter
SET key value
EXEC
```

```bash
EVAL "return redis.call('GET', KEYS[1])" 1 mykey
```

---

## 🔁 **8. Репликация и отказоустойчивость**

- `replicaof` / `SLAVEOF` — простая репликация
- **Redis Sentinel** — авто failover, мониторинг
- **Redis Cluster** — масштабирование по нодам (hash slot’ы)

---

## 🏗️ **9. Redis Sentinel (HA)**

Позволяет:
- Определить, что мастер умер
- Назначить нового мастера
- Перенастроить реплики

**Команды:**  
- `sentinel monitor`
- `sentinel get-master-addr-by-name`

---

## ☁️ **10. Redis Cluster**

- Поддерживает **шардирование** по slot'ам (16384)
- Автоматически распределяет ключи
- Использует **Gossip Protocol** для связи между нодами

Команда: `redis-cli --cluster create ...`

---

## 🧩 **11. Use cases**

- **Кэш** (например, между API и БД)
- **Счётчики** (`INCR`, `DECR`)
- **Флаги** (`SETBIT`, `GETBIT`)
- **Очереди** (`LPUSH`, `RPOP`)
- **Rate Limiting** (временные ключи)
- **Session store**
- **Lock-системы** (через `SET key val NX PX 10000`)

---

## 🔐 **12. Безопасность**

- `requirepass` в конфиге
- `rename-command` для защиты `FLUSHALL`, `DEBUG`, и др.
- ACLs (начиная с Redis 6): `user`, `permissions`

---

## 🧰 **13. Полезные команды**

```bash
MONITOR           # Смотри все команды в реальном времени
KEYS *            # Плохо! Используй SCAN
SCAN              # Итерирование по ключам
INFO              # Общая информация
CLIENT LIST       # Кто подключён
FLUSHDB / FLUSHALL# Очистка
```

---

## 🧠 **14. Частые ошибки на собесах**

- Использование `KEYS` вместо `SCAN`
- Не настроен `maxmemory-policy` → падение при нехватке RAM
- Нет TTL → кэш разрастается до бесконечности
- Ожидание, что pub/sub гарантирует доставку
- Использование Redis Streams без понимания `XACK`/`XGROUP`

---

## ✅ **15. Практика и лучшие практики**

- **Используй TTL** — всегда
- **Мониторь Redis** — latency, memory, hit/miss
- Используй **Redis exporter для Prometheus**
- Сохраняй **Atomicity** с Lua, WATCH/MULTI
- Не храни большие payload’ы — Redis не для этого
- Знай свои команды — часто спрашивают `ZADD`, `HSET`, `SETEX`, `XADD`