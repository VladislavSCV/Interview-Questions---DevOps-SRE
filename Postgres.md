## 📌 **1. Что такое PostgreSQL**

PostgreSQL — это мощная **объектно-реляционная СУБД**, с открытым исходным кодом.

**Плюсы:**
- Расширяемость (модули, типы, функции)
- Поддержка ACID
- Мощная работа с JSON, CTE, оконными функциями
- Репликация, партиционирование, триггеры

---

## 🧱 **2. Архитектура**

- **Process-based**: отдельный процесс на подключение
- **shared_buffers**: кэш
- **WAL (Write-Ahead Log)**: журнал транзакций
- **background writer, checkpointer** — фоновые процессы

---

## 🧠 **3. Основы SQL (напоминалка)**

```sql
SELECT * FROM users WHERE age > 20 ORDER BY created_at DESC LIMIT 10;
INSERT INTO users(name, age) VALUES('Alice', 25);
UPDATE users SET age = 26 WHERE name = 'Alice';
DELETE FROM users WHERE name = 'Alice';
```

---

## 📦 **4. Типы данных**

- `INTEGER`, `SERIAL`, `BIGINT`
- `TEXT`, `VARCHAR(n)`
- `BOOLEAN`
- `TIMESTAMP`, `DATE`
- `JSON`, `JSONB`
- `ARRAY`, `UUID`, `INET`

---

## 📚 **5. Индексы**

- `CREATE INDEX idx_name ON users(name);`
- Виды:
  - B-Tree (по умолчанию)
  - GIN (для JSONB / full-text)
  - GiST (гео)
  - HASH (редко используется)

---

## 🛡️ **6. Транзакции и изоляция**

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;
-- или ROLLBACK;
```

**Уровни изоляции:**
- READ COMMITTED (по умолчанию)
- REPEATABLE READ
- SERIALIZABLE

---

## 🔄 **7. Репликация**

- **Streaming replication** — мастер → реплика читает WAL
- **Синхронная**: мастер ждёт реплику
- **Асинхронная**: просто отправляет WAL
- Используется **pg_basebackup** или `repmgr`

---

## 🆘 **8. Failover / HA**

- Инструменты:
  - `Patroni` (кластер на основе etcd/consul)
  - `pg_auto_failover`
  - `repmgr`
  - `HAProxy` — определяет, кто мастер
- При failover:
  - Старая реплика → становится мастером
  - Другие реплики переподключаются

---

## ⚙️ **9. Механизмы блокировок**

- **Row-level lock** (`SELECT ... FOR UPDATE`)
- **Table-level lock**
- **Deadlocks** — циклическое ожидание блокировок
- `pg_locks` — показывает текущие блокировки

---

## ⚡ **10. Performance & Analyze**

- `EXPLAIN (ANALYZE)` — план выполнения запроса
- `VACUUM`, `ANALYZE`, `REINDEX`
- `autovacuum` — автоочистка мусора
- Используй индексы осознанно — они не всегда ускоряют

---

## 🔎 **11. JSON и JSONB**

```sql
SELECT data->>'name' FROM users WHERE data->>'role' = 'admin';
```

- `JSON` — как есть
- `JSONB` — бинарный, быстрее для фильтрации и индексов
- Индекс: `CREATE INDEX ON users USING gin(data);`

---

## 🧮 **12. Оконные функции и CTE**

**Оконные:**

```sql
SELECT name, age, RANK() OVER (ORDER BY age DESC) FROM users;
```

**CTE:**

```sql
WITH recent AS (
  SELECT * FROM orders WHERE created_at > now() - interval '7 days'
)
SELECT * FROM recent WHERE total > 100;
```

---

## 📊 **13. Партиционирование**

```sql
CREATE TABLE events (
  id serial, created_at date
) PARTITION BY RANGE (created_at);
```

- RANGE / LIST / HASH
- Улучшает масштабируемость на больших таблицах

---

## 🧪 **14. Расширения**

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION pg_stat_statements;
```

Популярные:
- `uuid-ossp`
- `pg_stat_statements` — анализ производительности
- `pg_trgm` — поиск по похожести
- `PostGIS` — гео-данные

---

## 🔐 **15. Безопасность**

- Роли и привилегии: `GRANT`, `REVOKE`, `CREATE ROLE`
- `pg_hba.conf` — правила доступа (host, user, метод)
- `ALTER USER` с паролем
- SSL-подключения

---

## 🛠️ **16. Полезные команды**

```bash
psql -U user -d dbname
\dt       -- список таблиц
\d users  -- схема таблицы
\i file.sql  -- выполнить файл
\q        -- выход
```

---

## 📈 **17. Monitoring / Metrics**

- `pg_stat_activity` — активные сессии
- `pg_stat_user_tables`, `pg_stat_database`
- Метрики через:
  - `pg_stat_statements`
  - `pgBadger`
  - `Postgres Exporter` + Prometheus + Grafana

---

## 🧠 **18. Типичные вопросы на собесе**

- Как работает репликация? Sync vs Async?
- Как решать проблему split-brain?
- Что делать при deadlock?
- Как работает MVCC?
- Чем JSON отличается от JSONB?
- Как устроен план выполнения запроса?
- Как сделать HA?