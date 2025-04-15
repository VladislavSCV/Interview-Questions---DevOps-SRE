## 📦 Что такое ELK?

**ELK** — это стек инструментов:

- **Elasticsearch** — полнотекстовый поисковик и движок аналитики
- **Logstash** — сбор, обработка и передача логов
- **Kibana** — визуализация данных из Elasticsearch

Иногда добавляют **Beats** (Filebeat, Metricbeat и др.) — для лёгкой отправки данных → называют **Elastic Stack**

---

## 🔎 **Elasticsearch**

### Что это?
- Распределённый поисковый движок
- Основан на **Lucene**
- Работает с **JSON-документами**
- Очень быстро ищет и аггрегирует

### Ключевые понятия:
- **Index** — как таблица в БД (коллекция документов)
- **Document** — JSON-объект
- **Shard** — часть индекса (для масштабирования)
- **Replica** — копия шардов

### Основные запросы:

```bash
GET /my-index/_search
POST /my-index/_doc
PUT /my-index
DELETE /my-index
```

### Mapping и анализаторы:
- **Mapping** = схема индекса
- **Analyzer** = как разбивать текст (по токенам)

---

## 🛠 **Logstash**

### Что это?
- Инструмент для **ETL** логов (Extract-Transform-Load)
- Забирает данные → обрабатывает → отсылает дальше (обычно в Elasticsearch)

### Конфигурация:
```conf
input {
  file {
    path => "/var/log/nginx/access.log"
  }
}

filter {
  grok {
    match => { "message" => "%{COMMONAPACHELOG}" }
  }
  date {
    match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "nginx-logs-%{+YYYY.MM.dd}"
  }
}
```

### Плагины:
- `input` (beats, file, syslog, jdbc, kafka…)
- `filter` (grok, mutate, date, geoip…)
- `output` (elasticsearch, file, stdout, kafka…)

---

## 📊 **Kibana**

### Что это?
- Интерфейс для поиска и визуализации данных из Elasticsearch
- Работает через REST API

### Возможности:
- Dashboard
- Discover (поиск логов)
- Visualize (графики, пайчарты, таймлайны)
- Dev Tools (консоль для запросов)
- Alerting (оповещения)

---

## 📬 **Beats (дополнительно)**

- **Filebeat** — отправка логов
- **Metricbeat** — метрики системы
- **Packetbeat** — сетевой трафик
- **Heartbeat** — проверка доступности
- Лёгкие, написаны на Go

Пример Filebeat:

```yaml
filebeat.inputs:
  - type: log
    paths:
      - /var/log/nginx/*.log

output.logstash:
  hosts: ["localhost:5044"]
```

---

## ⚙️ Архитектура типовая

```
[Filebeat] --> [Logstash] --> [Elasticsearch] --> [Kibana]
```

Можно **без Logstash** (Filebeat → Elasticsearch) для простых сценариев.

---

## 🔥 Частые вопросы на собесе

- Чем отличается Logstash от Beats?
- Что такое Shard и Replica?
- Как масштабируется Elasticsearch?
- Как настраивать парсинг логов в Logstash?
- Как искать в Kibana?
- Что такое Ingest pipeline?
- Как работает Full-Text Search?
- Что делает grok?
- Как работает индексирование?
- Как настраивать alert'ы?

---

## 💡 Полезное

- Kibana Dev Tools = Swagger для Elasticsearch
- Grok Debugger: https://grokdebug.herokuapp.com/
- Поиск: `field:value`, `field:[10 TO 100]`, `field.keyword: "exact match"`
- Aggregations: `terms`, `range`, `date_histogram`
- В проде часто используют Kafka → Logstash для буферизации