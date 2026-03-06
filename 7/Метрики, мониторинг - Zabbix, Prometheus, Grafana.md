## 1. Чем отличается лог от метрики?

### Простая аналогия

text

```
Представь больницу:

МЕТРИКА = медицинские показатели
  • Температура: 38.5°C
  • Давление: 140/90
  • Пульс: 95 уд/мин
  • Сахар: 6.2 ммоль/л

ЛОГ = медицинская карта
  • 10:00 - Пациент жалуется на головную боль
  • 10:15 - Назначен анализ крови
  • 10:45 - Обнаружено повышенное СОЭ
  • 11:00 - Назначен антибиотик амоксициллин 500мг
```

### Определения

text

```
┌────────────────────────────────────────────────────────┐
│ ЛОГ                                                    │
│ ───                                                    │
│ Запись о СОБЫТИИ (что произошло, когда, почему)        │
│                                                        │
│ Отвечает на вопросы:                                   │
│ • ЧТО случилось?                                       │
│ • КОГДА это произошло?                                 │
│ • КТО это сделал?                                      │
│ • ПОЧЕМУ произошла ошибка?                             │
└────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────┐
│ МЕТРИКА                                                │
│ ───────                                                │
│ ЧИСЛО + timestamp (измерение в момент времени)         │
│                                                        │
│ Отвечает на вопросы:                                   │
│ • СКОЛЬКО?                                             │
│ • КАК БЫСТРО?                                          │
│ • КАКОЙ ПРОЦЕНТ?                                       │
│ • Есть ли ТРЕНД?                                       │
└────────────────────────────────────────────────────────┘
```

### Примеры

JSON

```
// ═══ ЛОГ (текст/JSON) ═══

[2024-01-20 10:15:30] ERROR: Database connection failed
[2024-01-20 10:15:31] INFO: User 'john' logged in from 192.168.1.50
[2024-01-20 10:15:32] WARN: Slow query took 5.2 seconds

{
  "timestamp": "2024-01-20T10:15:30Z",
  "level": "error",
  "service": "payment-api",
  "message": "Payment failed",
  "user_id": 1001,
  "order_id": "ORD-12345",
  "amount": 99.99,
  "error_code": "INSUFFICIENT_FUNDS",
  "card_last4": "4242"
}
```

prometheus

```
# ═══ МЕТРИКА (число) ═══

http_requests_total{method="GET", status="200"} 1547
cpu_usage_percent{host="server-01"} 45.2
memory_used_bytes{host="server-01"} 8589934592
payment_amount_usd{status="success"} 1250.50
```

### Сравнительная таблица

text

```
┌──────────────────┬────────────────────┬────────────────────┐
│ Критерий         │ Логи               │ Метрики            │
├──────────────────┼────────────────────┼────────────────────┤
│ Формат           │ Текст / JSON       │ Число + labels     │
│ Структура        │ Гибкая             │ Жесткая            │
│ Размер записи    │ 100-1000 байт      │ 10-50 байт         │
│ Объём хранения   │ GB – TB            │ MB – GB            │
│ Срок хранения    │ 7-30 дней          │ 6-24 месяца        │
│ Индексирование   │ Full-text search   │ Time-series DB     │
│ Агрегация        │ Сложная            │ Простая            │
│ Алертинг         │ По паттернам       │ По порогам         │
│ Визуализация     │ Таблицы, поиск     │ Графики, дашборды  │
│ Главный вопрос   │ "ЧТО произошло?"   │ "СКОЛЬКО? КАК?"    │
│ Цель             │ Debugging, аудит   │ Monitoring, SLA    │
│ Инструменты      │ ELK, Loki, Splunk  │ Prometheus, Zabbix │
│ Стоимость        │ Дорого (объём)     │ Дешево             │
└──────────────────┴────────────────────┴────────────────────┘
```

### Как они работают вместе

text

```
Сценарий: Сайт упал!

┌─────────────────────────────────────────────────────────┐
│ Шаг 1: МЕТРИКА обнаруживает проблему                    │
│                                                         │
│   http_errors_per_second = 150  (было 0.5)             │
│   ────────────────────────────────                      │
│            │                                            │
│            ▼                                            │
│   🚨 АЛЕРТ: "Error rate spike on payment-service!"      │
└─────────────────────────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────────┐
│ Шаг 2: Инженер идёт в ЛОГИ                              │
│                                                         │
│   Фильтр: service="payment" AND level="error"           │
│           AND timestamp > now-5m                        │
│                                                         │
│   Находит:                                              │
│   [10:15:30] ERROR: SSL handshake failed                │
│   [10:15:30] Connection to db.example.com:5432 timeout  │
│   [10:15:31] Certificate verification failed            │
│   [10:15:31] x509: certificate has expired              │
│   [10:15:31] Not after: 2024-01-20 10:00:00 UTC         │
└─────────────────────────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────────┐
│ Шаг 3: ПРИЧИНА найдена                                  │
│                                                         │
│   Истёк SSL-сертификат базы данных!                     │
│   Решение: обновить сертификат                          │
└─────────────────────────────────────────────────────────┘
```

### Вывод

text

```
Метрики без логов = градусник без диагноза
  "У вас температура 39°, но я не знаю почему"

Логи без метрик = микроскоп без панорамы
  "Вижу конкретный случай, но не общую картину"

НА ПРАКТИКЕ НУЖНЫ ОБА!
```

---

## 2. Примеры метрик

### Типы метрик в Prometheus

text

```
┌───────────┬──────────────────────────────────────────────┐
│ Counter   │ Монотонный счетчик (только растёт)           │
│           │                                              │
│           │ Как одометр в машине: 10 → 15 → 23 → 45     │
│           │ Никогда не уменьшается!                      │
│           │                                              │
│           │ Примеры:                                     │
│           │ • http_requests_total                        │
│           │ • errors_total                               │
│           │ • bytes_sent_total                           │
│           │                                              │
│           │ Запросы в PromQL:                            │
│           │ rate(http_requests_total[5m])  # req/sec     │
├───────────┼──────────────────────────────────────────────┤
│ Gauge     │ Значение которое растёт и падает             │
│           │                                              │
│           │ Как термометр: 20 → 45 → 30 → 60 → 15        │
│           │                                              │
│           │ Примеры:                                     │
│           │ • cpu_usage_percent                          │
│           │ • memory_used_bytes                          │
│           │ • active_connections                         │
│           │ • queue_size                                 │
│           │ • temperature_celsius                        │
│           │                                              │
│           │ Запросы в PromQL:                            │
│           │ avg(cpu_usage_percent)  # средняя            │
├───────────┼──────────────────────────────────────────────┤
│ Histogram │ Распределение значений по корзинам (buckets) │
│           │                                              │
│           │ Время ответа API:                            │
│           │ ┌──────────────────────────────────────┐     │
│           │ │ bucket    count  (накопительно)     │     │
│           │ ├──────────────────────────────────────┤     │
│           │ │ le="0.1"    500  (500 < 0.1s)       │     │
│           │ │ le="0.5"    800  (800 < 0.5s)       │     │
│           │ │ le="1.0"    950  (950 < 1.0s)       │     │
│           │ │ le="+Inf"  1000  (все запросы)      │     │
│           │ └──────────────────────────────────────┘     │
│           │                                              │
│           │ Пример: http_request_duration_seconds        │
│           │                                              │
│           │ Запросы в PromQL:                            │
│           │ histogram_quantile(0.95, ...)  # p95         │
├───────────┼──────────────────────────────────────────────┤
│ Summary   │ Квантили (процентили) на стороне клиента     │
│           │                                              │
│           │ p50 = 0.2s  (50% запросов быстрее)          │
│           │ p95 = 0.8s  (95% запросов быстрее)          │
│           │ p99 = 1.5s  (99% запросов быстрее)          │
│           │                                              │
│           │ ⚠️ Нельзя агрегировать!                      │
│           │ Используй Histogram вместо Summary           │
└───────────┴──────────────────────────────────────────────┘
```

### Примеры метрик по категориям

text

```
════════════════════════════════════════════════════════════
  СИСТЕМНЫЕ МЕТРИКИ (Infrastructure)
════════════════════════════════════════════════════════════

CPU:
─────
node_cpu_seconds_total{cpu="0",mode="idle"}     78345.67   (Counter)
node_load1                                      2.5        (Gauge)
node_load5                                      1.8        (Gauge)
node_load15                                     1.2        (Gauge)

Память:
───────
node_memory_MemTotal_bytes                      16777216000 (Gauge)
node_memory_MemAvailable_bytes                  8589934592  (Gauge)
node_memory_MemFree_bytes                       4294967296  (Gauge)
node_memory_Buffers_bytes                       1073741824  (Gauge)
node_memory_Cached_bytes                        3221225472  (Gauge)

Диск:
─────
node_filesystem_size_bytes{mountpoint="/"}      107374182400 (Gauge)
node_filesystem_free_bytes{mountpoint="/"}      53687091200  (Gauge)
node_disk_read_bytes_total{device="sda"}        1099511627776 (Counter)
node_disk_written_bytes_total{device="sda"}     549755813888  (Counter)
node_disk_io_time_seconds_total{device="sda"}   3600          (Counter)

Сеть:
─────
node_network_receive_bytes_total{device="eth0"} 10995116277760 (Counter)
node_network_transmit_bytes_total{device="eth0"} 5497558138880 (Counter)
node_network_receive_packets_total{device="eth0"} 1000000000  (Counter)
node_network_receive_errs_total{device="eth0"}    5           (Counter)


════════════════════════════════════════════════════════════
  МЕТРИКИ ПРИЛОЖЕНИЯ (Application)
════════════════════════════════════════════════════════════

HTTP сервер:
────────────
http_requests_total{method="GET",status="200"}      15470    (Counter)
http_requests_total{method="POST",status="201"}     89       (Counter)
http_requests_total{method="GET",status="500"}      3        (Counter)
http_request_duration_seconds{endpoint="/api"}     0.25     (Histogram)
http_requests_in_progress                           42       (Gauge)
http_response_size_bytes                            1024     (Histogram)

База данных:
────────────
db_connections_active                               25       (Gauge)
db_connections_idle                                 75       (Gauge)
db_connections_max                                  100      (Gauge)
db_queries_total{operation="select"}                1000000  (Counter)
db_query_duration_seconds{operation="select"}       0.05     (Histogram)
db_slow_queries_total                               150      (Counter)
db_deadlocks_total                                  2        (Counter)

Кэш (Redis/Memcached):
──────────────────────
cache_hits_total                                    950000   (Counter)
cache_misses_total                                  50000    (Counter)
cache_hit_ratio                                     0.95     (Gauge)
cache_memory_used_bytes                             2147483648 (Gauge)
cache_evictions_total                               1000     (Counter)
cache_keys_count                                    500000   (Gauge)

Очереди (RabbitMQ/Kafka):
─────────────────────────
queue_messages_ready{queue="emails"}                1000     (Gauge)
queue_messages_unacked{queue="emails"}              50       (Gauge)
queue_publish_total{queue="emails"}                 500000   (Counter)
queue_consume_total{queue="emails"}                 499000   (Counter)
queue_consumer_count{queue="emails"}                5        (Gauge)


════════════════════════════════════════════════════════════
  БИЗНЕС-МЕТРИКИ (Business KPI)
════════════════════════════════════════════════════════════

E-commerce:
───────────
orders_created_total                                10000    (Counter)
orders_completed_total                              9500     (Counter)
orders_cancelled_total                              500      (Counter)
revenue_total_usd                                   500000   (Counter)
cart_abandoned_total                                2000     (Counter)
product_views_total{product_id="123"}               50000    (Counter)
checkout_duration_seconds                           45       (Histogram)

SaaS:
─────
signups_total                                       500      (Counter)
subscriptions_active{plan="premium"}                10000    (Gauge)
subscriptions_cancelled_total                       50       (Counter)
monthly_recurring_revenue_usd                       50000    (Gauge)
churn_rate                                          0.05     (Gauge)
trial_conversions_total                             100      (Counter)


════════════════════════════════════════════════════════════
  KUBERNETES МЕТРИКИ
════════════════════════════════════════════════════════════

Pods:
─────
kube_pod_status_phase{phase="Running"}              1        (Gauge)
kube_pod_status_phase{phase="Pending"}              0        (Gauge)
kube_pod_container_restarts_total                   5        (Counter)
kube_pod_container_status_ready                     1        (Gauge)

Nodes:
──────
kube_node_status_condition{condition="Ready"}       1        (Gauge)
kube_node_status_allocatable_cpu_cores              8        (Gauge)
kube_node_status_allocatable_memory_bytes           16777216000 (Gauge)


════════════════════════════════════════════════════════════
  SLI/SLO МЕТРИКИ (Site Reliability)
════════════════════════════════════════════════════════════

Availability:
─────────────
service_up                                          1        (Gauge)
uptime_seconds_total                                2592000  (Counter)

Latency:
────────
request_duration_seconds{quantile="0.5"}            0.1      (Summary)
request_duration_seconds{quantile="0.95"}           0.5      (Summary)
request_duration_seconds{quantile="0.99"}           1.2      (Summary)

Error Rate:
───────────
requests_total                                      1000000  (Counter)
requests_failed_total                               50000    (Counter)
error_rate                                          0.05     (Gauge)

Throughput:
───────────
requests_per_second                                 5000     (Gauge)
transactions_per_second                             1000     (Gauge)


════════════════════════════════════════════════════════════
  БЕЗОПАСНОСТЬ
════════════════════════════════════════════════════════════

failed_login_attempts_total                         150      (Counter)
blocked_ips_total                                   25       (Counter)
ssl_certificate_expiry_seconds                      2592000  (Gauge) # 30 дн
ssl_handshake_errors_total                          5        (Counter)
brute_force_attacks_detected_total                  10       (Counter)
```

### Формат в Prometheus

prometheus

```
# TYPE http_requests_total counter
# HELP http_requests_total Total number of HTTP requests
http_requests_total{method="GET",status="200",endpoint="/api/users"} 1547
http_requests_total{method="POST",status="201",endpoint="/api/users"} 89
http_requests_total{method="GET",status="500",endpoint="/api/orders"} 3

# TYPE node_cpu_seconds_total counter
node_cpu_seconds_total{cpu="0",mode="idle"} 78345.67
node_cpu_seconds_total{cpu="0",mode="system"} 1234.56
node_cpu_seconds_total{cpu="0",mode="user"} 5678.90

# TYPE node_memory_MemAvailable_bytes gauge
node_memory_MemAvailable_bytes 8589934592

# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.1"} 500
http_request_duration_seconds_bucket{le="0.5"} 800
http_request_duration_seconds_bucket{le="1.0"} 950
http_request_duration_seconds_bucket{le="+Inf"} 1000
http_request_duration_seconds_sum 450.5
http_request_duration_seconds_count 1000
```

---

## 3. Чем отличаются Zabbix и Prometheus?

### Философия

text

```
ZABBIX (2001)                    PROMETHEUS (2012)
─────────────                    ─────────────────
Традиционный                     Cloud-native
Централизованный                 Децентрализованный
All-in-one решение               Модульная экосистема
Готовое из коробки               Собери сам
```

### Архитектура

text

```
═══════════════════════════════════════════════════════════
  ZABBIX: Централизованная, Push + Pull
═══════════════════════════════════════════════════════════

  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐
  │ Server 1  │  │ Server 2  │  │ Server 3  │  │ Switch    │
  │           │  │           │  │           │  │ (SNMP)    │
  │  AGENT    │  │  AGENT    │  │  AGENT    │  │           │
  │  :10050   │  │  :10050   │  │  :10050   │  │  :161     │
  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘
        │              │              │              │
        │ active       │ active       │ passive      │
        │ push         │ push         │ pull(SNMP)   │
        │              │              │              │
        └──────────────┴──────────────┴──────────────┘
                                 │
                                 ▼
                     ┌───────────────────────┐
                     │   ZABBIX SERVER       │
                     │                       │
                     │  ┌─────────────────┐  │
                     │  │ Zabbix Core     │  │ ← Сбор
                     │  │ (C язык)        │  │   Обработка
                     │  └─────────────────┘  │   Триггеры
                     │                       │
                     │  ┌─────────────────┐  │
                     │  │ MySQL/PostgreSQL│  │ ← Хранение
                     │  │ TimescaleDB     │  │
                     │  └─────────────────┘  │
                     │                       │
                     │  ┌─────────────────┐  │
                     │  │ Web Frontend    │  │ ← Визуализация
                     │  │ (PHP)           │  │   Алертинг
                     │  └─────────────────┘  │   Управление
                     │                       │
                     └───────────────────────┘
                               │
                               ▼
                     Email, SMS, Slack, PagerDuty


═══════════════════════════════════════════════════════════
  PROMETHEUS: Децентрализованная, Pull
═══════════════════════════════════════════════════════════

  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐
  │ Service 1 │  │ Service 2 │  │ Service 3 │  │ MySQL     │
  │ :8080     │  │ :8080     │  │ :8080     │  │           │
  │ /metrics  │  │ /metrics  │  │ /metrics  │  │           │
  └─────▲─────┘  └─────▲─────┘  └─────▲─────┘  └─────┬─────┘
        │              │              │              │
        │              │              │              ▼
        │   pull       │   pull       │          ┌────────┐
        │   (15s)      │   (15s)      │   pull   │ mysqld │
        │              │              │  (15s)   │exporter│
        │              │              │          │ :9104  │
        └──────────────┴──────────────┴──────────┴───▲────┘
                                                     │
                        ┌────────────────────────────┘
                        │
                 ┌──────┴──────────┐
                 │   PROMETHEUS    │
                 │     SERVER      │
                 │                 │
                 │ ┌─────────────┐ │ ← Только:
                 │ │ Scraper     │ │   Сбор (pull)
                 │ │ (Pull)      │ │   Хранение (TSDB)
                 │ └─────────────┘ │   Запросы (PromQL)
                 │                 │
                 │ ┌─────────────┐ │
                 │ │ TSDB        │ │
                 │ │ (встроенная)│ │
                 │ └─────────────┘ │
                 │                 │
                 │ ┌─────────────┐ │
                 │ │ PromQL API  │ │
                 │ └─────────────┘ │
                 └────┬──────┬─────┘
                      │      │
                ┌─────┴──┐ ┌─┴────────────┐
                │Grafana │ │ AlertManager │ ← Отдельные
                │        │ │              │   компоненты
                │(визуал)│ │ → Slack      │
                │        │ │ → PagerDuty  │
                └────────┘ └──────────────┘
```

### Подробное сравнение

text

```
┌─────────────────────┬──────────────────┬──────────────────┐
│ Критерий            │ Zabbix           │ Prometheus       │
├─────────────────────┼──────────────────┼──────────────────┤
│ Год создания        │ 2001             │ 2012             │
│ Компания            │ Zabbix LLC       │ SoundCloud→CNCF  │
│ Лицензия            │ GPL v2           │ Apache 2.0       │
│ Написан на          │ C + PHP          │ Go               │
├─────────────────────┼──────────────────┼──────────────────┤
│ Модель сбора        │ Push + Pull      │ Pull             │
│                     │ (гибридная)      │ (преимущественно)│
│ Агенты              │ Обязательны ⭐   │ Опциональны      │
│ Порты               │ 10050, 10051     │ 9090 (сервер)    │
│                     │                  │ 9xxx (exporters) │
├─────────────────────┼──────────────────┼──────────────────┤
│ Архитектура         │ Централизованная │ Федерация        │
│ Хранилище           │ MySQL/PostgreSQL │ TSDB (встроенная)│
│                     │ TimescaleDB      │                  │
│ Retention           │ Гибкий ⭐        │ 15 дней (default)│
│ Долгосрочное хран.  │ Да (SQL) ⭐      │ Thanos/Cortex    │
├─────────────────────┼──────────────────┼──────────────────┤
│ Язык запросов       │ GUI + API        │ PromQL ⭐        │
│ Визуализация        │ Встроенная ⭐    │ Grafana          │
│ Алертинг            │ Встроенный ⭐    │ AlertManager     │
│ Дашборды            │ Встроенные       │ Grafana          │
│ Карты сети          │ Да ⭐            │ Нет              │
├─────────────────────┼──────────────────┼──────────────────┤
│ Настройка           │ GUI (мышкой) ⭐  │ YAML (код)       │
│ Автодобавление      │ Auto-registration│ Service Discovery│
│ Шаблоны             │ Да ⭐            │ Да               │
│ Массовые операции   │ Удобно ⭐        │ Через код        │
├─────────────────────┼──────────────────┼──────────────────┤
│ SNMP                │ ⭐⭐⭐⭐⭐ Native  │ ⭐⭐ Exporter     │
│ IPMI                │ ⭐⭐⭐⭐⭐ Native  │ ⭐⭐ Exporter     │
│ JMX                 │ ⭐⭐⭐⭐ Native    │ ⭐⭐ Exporter     │
│ Сетевое оборудование│ ⭐⭐⭐⭐⭐ Отлично │ ⭐⭐ Средне       │
│ Docker              │ ⭐⭐⭐ Хорошо     │ ⭐⭐⭐⭐⭐ Отлично │
│ Kubernetes          │ ⭐⭐ Слабо        │ ⭐⭐⭐⭐⭐ Стандарт │
│ Cloud Native        │ ⭐⭐ Слабо        │ ⭐⭐⭐⭐⭐ Стандарт │
│ Custom метрики      │ ⭐⭐⭐ Можно      │ ⭐⭐⭐⭐⭐ Легко    │
├─────────────────────┼──────────────────┼──────────────────┤
│ Масштабирование     │ ⭐⭐⭐ Vertical   │ ⭐⭐⭐⭐⭐ Horizontal│
│ High Availability   │ ⭐⭐⭐ Сложно     │ ⭐⭐⭐⭐ Федерация │
│ Производительность  │ ⭐⭐⭐ 10k метрик │ ⭐⭐⭐⭐ 1M+ метрик│
├─────────────────────┼──────────────────┼──────────────────┤
│ Экосистема          │ ⭐⭐⭐ Своя       │ ⭐⭐⭐⭐⭐ Огромная │
│ Exporters           │ Ограничено       │ 200+ официальных │
│ Интеграции          │ ⭐⭐⭐⭐ Много    │ ⭐⭐⭐⭐⭐ Больше  │
│ Сообщество          │ ⭐⭐⭐⭐ Активное │ ⭐⭐⭐⭐⭐ Огромное │
├─────────────────────┼──────────────────┼──────────────────┤
│ Кривая обучения     │ ⭐⭐⭐ Средняя    │ ⭐⭐⭐⭐ Крутая    │
│ Документация        │ ⭐⭐⭐⭐ Хорошая  │ ⭐⭐⭐⭐⭐ Отличная │
│ Поддержка           │ Платная ⭐       │ Community        │
└─────────────────────┴──────────────────┴──────────────────┘
```

### Когда что выбирать

text

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━┓  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ ZABBIX выбирай когда:     ┃  ┃ PROMETHEUS выбирай когда: ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━┫  ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                           ┃  ┃                           ┃
┃ ✅ Физические серверы      ┃  ┃ ✅ Kubernetes / Docker     ┃
┃ ✅ Виртуальные машины      ┃  ┃ ✅ Микросервисы            ┃
┃ ✅ Сетевое оборудование    ┃  ┃ ✅ Cloud-native приложения ┃
┃    (свитчи, роутеры)      ┃  ┃ ✅ Динамическая инфра      ┃
┃ ✅ SNMP-устройства         ┃  ┃ ✅ Auto-scaling            ┃
┃ ✅ IPMI мониторинг         ┃  ┃ ✅ Service Discovery       ┃
┃ ✅ Традиционный ДЦ         ┃  ┃ ✅ Custom метрики          ┃
┃ ✅ Нужен готовый UI        ┃  ┃ ✅ GitOps / IaC            ┃
┃ ✅ Сложные сценарии        ┃  ┃ ✅ PromQL запросы          ┃
┃    алертинга              ┃  ┃ ✅ Огромная экосистема     ┃
┃ ✅ GUI настройка           ┃  ┃ ✅ Высокая нагрузка        ┃
┃ ✅ Карты сети              ┃  ┃ ✅ Time-series анализ      ┃
┃ ✅ Платная поддержка       ┃  ┃                           ┃
┃                           ┃  ┃                           ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━┛  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ НА ПРАКТИКЕ часто используют ОБА:                        ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                                                          ┃
┃  Zabbix      → Инфраструктура, сеть, железо              ┃
┃  Prometheus  → Приложения, K8s, микросервисы             ┃
┃  Grafana     → Единый UI для всего                       ┃
┃                                                          ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```

---

## 4. Что такое Grafana?

### Определение

text

```
┌──────────────────────────────────────────────────────────┐
│  GRAFANA — Open-source платформа для:                    │
│                                                          │
│  • Визуализации метрик и логов                           │
│  • Создания дашбордов                                    │
│  • Алертинга                                             │
│  • Исследования данных (Explore)                         │
│                                                          │
│  Grafana САМА НЕ ХРАНИТ данные!                          │
│  Она подключается к источникам (datasources)             │
└──────────────────────────────────────────────────────────┘

Аналогия:
  Grafana = Телевизор  (показывает картинку)
  Prometheus/Loki = Телеканалы (источник сигнала)
```

### Архитектура

text

```
┌──────────────────────────────────────────────────────────┐
│                    GRAFANA SERVER                         │
│                                                          │
│  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓  │
│  ┃ Dashboard: Production Monitoring                  ┃  │
│  ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫  │
│  ┃                                                   ┃  │
│  ┃  ┌────────┐ ┌────────┐ ┌────────┐ ┌───────────┐  ┃  │
│  ┃  │  CPU   │ │  RAM   │ │  RPS   │ │  Errors   │  ┃  │
│  ┃  │  45%   │ │  72%   │ │  5000  │ │   12      │  ┃  │
│  ┃  │   🟢   │ │   🟡   │ │   🟢   │ │   🔴      │  ┃  │
│  ┃  └────────┘ └────────┘ └────────┘ └───────────┘  ┃  │
│  ┃                                                   ┃  │
│  ┃  ┌──────────────────────────────────────────┐    ┃  │
│  ┃  │        HTTP Request Rate                 │    ┃  │
│  ┃  │   5000 ┤    ╱╲      ╱╲                  │    ┃  │
│  ┃  │        │   ╱  ╲    ╱  ╲    ╱╲           │    ┃  │
│  ┃  │   3000 ├  ╱    ╲  ╱    ╲  ╱  ╲          │    ┃  │
│  ┃  │        │ ╱      ╲╱      ╲╱    ╲         │    ┃  │
│  ┃  │   1000 ┼─────────────────────────────────│    ┃  │
│  ┃  │        12:00   14:00   16:00   18:00     │    ┃  │
│  ┃  └──────────────────────────────────────────┘    ┃  │
│  ┃                                                   ┃  │
│  ┃  ┌──────────────┐  ┌──────────────────────────┐  ┃  │
│  ┃  │ Top Errors   │  │ Slow Endpoints           │  ┃  │
│  ┃  │              │  │                          │  ┃  │
│  ┃  │ ████ DB: 8   │  │ 1. /api/search    1.2s   │  ┃  │
│  ┃  │ ███  Auth: 3 │  │ 2. /api/report    0.8s   │  ┃  │
│  ┃  │ █    Pay: 1  │  │ 3. /api/export    0.7s   │  ┃  │
│  ┃  └──────────────┘  └──────────────────────────┘  ┃  │
│  ┃                                                   ┃  │
│  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛  │
│                                                          │
│  Alerting Rules:                                         │
│  🔔 CPU > 80% → Slack #alerts                            │
│  🔔 Error rate > 5% → PagerDuty                          │
│  🔔 Disk > 90% → Email ops@company.com                   │
│                                                          │
└────────────────────┬─────────────────────────────────────┘
                     │
     ┌───────────────┼───────────────┬──────────────┐
     │               │               │              │
     ▼               ▼               ▼              ▼
┌──────────┐  ┌──────────┐  ┌──────────────┐  ┌────────┐
│Prometheus│  │   Loki   │  │Elasticsearch │  │  MySQL │
│          │  │          │  │              │  │        │
│(метрики) │  │  (логи)  │  │    (логи)    │  │(данные)│
└──────────┘  └──────────┘  └──────────────┘  └────────┘
```

### Источники данных (Datasources)

text

```
════════════════════════════════════════════════════════════
  МЕТРИКИ (Time-Series)
════════════════════════════════════════════════════════════

• Prometheus        ⭐ основной для K8s/Cloud-native
• InfluxDB          ⭐ IoT, временные ряды
• Graphite             классический
• OpenTSDB             масштабируемый
• TimescaleDB          PostgreSQL extension
• Zabbix (plugin)      интеграция с Zabbix
• CloudWatch (AWS)  ⭐ AWS мониторинг
• Azure Monitor     ⭐ Azure мониторинг
• Google Cloud      ⭐ GCP мониторинг
• Datadog              SaaS мониторинг


════════════════════════════════════════════════════════════
  ЛОГИ
════════════════════════════════════════════════════════════

• Loki              ⭐ Grafana-native, как Prometheus для логов
• Elasticsearch     ⭐ ELK stack
• Splunk               enterprise логи
• CloudWatch Logs      AWS логи


════════════════════════════════════════════════════════════
  ТРЕЙСЫ (Distributed Tracing)
════════════════════════════════════════════════════════════

• Tempo             ⭐ Grafana-native
• Jaeger            ⭐ CNCF стандарт
• Zipkin               Twitter open-source
• AWS X-Ray            AWS трейсинг


════════════════════════════════════════════════════════════
  БАЗЫ ДАННЫХ
════════════════════════════════════════════════════════════

• MySQL             ⭐
• PostgreSQL        ⭐
• Microsoft SQL Server
• ClickHouse           аналитика
• MongoDB              NoSQL
• Redis                in-memory DB


════════════════════════════════════════════════════════════
  ДРУГИЕ
════════════════════════════════════════════════════════════

• JSON API             любой HTTP API
• CSV                  статические данные
• Google Sheets        коллаборация
• TestData             демо/тестирование
• SimpleJson           custom datasource


ВСЕГО: 150+ источников через официальные и community плагины
```

### Ключевые возможности

text

```
1. ДАШБОРДЫ
   ──────────
   • Графики (Graph, Time series)
   • Stat-панели (число + цвет)
   • Таблицы
   • Heatmaps (тепловые карты)
   • Гистограммы
   • Logs panel
   • Переменные ($host, $environment)

2. АЛЕРТИНГ
   ────────
   • Правила на основе запросов
   • Contact points: Email, Slack, PagerDuty, Telegram
   • Notification policies
   • Silences (временное отключение)

3. EXPLORE
   ───────
   • Ручное исследование метрик
   • Ручное исследование логов
   • Trace viewer

4. PROVISIONING
   ────────────
   • Дашборды как код (JSON/YAML)
   • Datasources как код
   • Алерты как код
   • GitOps workflow

5. RBAC (Teams & Permissions)
   ──────────────────────────
   • Organizations
   • Teams
   • Folders
   • Permissions (Viewer, Editor, Admin)

6. ПЛАГИНЫ
   ────────
   • Datasource plugins
   • Panel plugins
   • App plugins
```

---

## 5. Как установить и настроить Zabbix-агент?

### Установка (Ubuntu/Debian)

Bash

```
# ═══ Шаг 1: Добавить репозиторий Zabbix ═══
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_7.0-1+ubuntu22.04_all.deb
sudo apt update

# ═══ Шаг 2: Установить агент ═══
# Agent2 — новая версия на Go (рекомендуется)
sudo apt install zabbix-agent2

# Или старый Agent (на C)
sudo apt install zabbix-agent
```

### Установка (CentOS/RHEL)

Bash

```
# RHEL 8/9
sudo rpm -Uvh https://repo.zabbix.com/zabbix/7.0/rhel/8/x86_64/zabbix-release-7.0-1.el8.noarch.rpm
sudo yum clean all
sudo yum install zabbix-agent2

# RHEL 7
sudo rpm -Uvh https://repo.zabbix.com/zabbix/7.0/rhel/7/x86_64/zabbix-release-7.0-1.el7.noarch.rpm
sudo yum install zabbix-agent2
```

### Настройка конфигурации

Bash

```
sudo nano /etc/zabbix/zabbix_agent2.conf
```

ini

```
### ОСНОВНЫЕ ПАРАМЕТРЫ ###

# IP-адрес Zabbix Server (откуда разрешены подключения)
# Можно указать несколько через запятую
Server=192.168.1.100

# IP-адрес для активных проверок (агент сам подключается к серверу)
ServerActive=192.168.1.100

# Имя хоста - ДОЛЖНО СОВПАДАТЬ с именем в веб-интерфейсе!
Hostname=web-server-01

# Порт агента (по умолчанию 10050)
ListenPort=10050

# IP адрес для прослушивания (0.0.0.0 = все интерфейсы)
ListenIP=0.0.0.0


### ЛОГИРОВАНИЕ ###

# Путь к лог-файлу
LogFile=/var/log/zabbix/zabbix_agent2.log

# Размер лог-файла (MB)
LogFileSize=10

# Уровень логирования (0-5, где 4=debug)
DebugLevel=3


### БЕЗОПАСНОСТЬ ###

# Разрешить выполнение удаленных команд
# 0 - запрещено, 1 - разрешено
EnableRemoteCommands=0

# Таймауты
Timeout=30


### ДОПОЛНИТЕЛЬНО ###

# Файлы с пользовательскими параметрами
Include=/etc/zabbix/zabbix_agent2.d/*.conf
```

### Два режима работы агента

text

```
┌──────────────────────────────────────────────────────────┐
│ PASSIVE MODE (пассивный)                                 │
│ ────────────────────────────                              │
│ Сервер опрашивает агента                                 │
│                                                          │
│  ┌────────────┐             ┌─────────────┐             │
│  │  Zabbix    │  "Дай CPU"  │   Agent     │             │
│  │  Server    │────────────→│   :10050    │             │
│  │            │             │             │             │
│  │            │←────────────│             │             │
│  │            │   "45.2%"   │             │             │
│  └────────────┘             └─────────────┘             │
│                                                          │
│  Параметр: Server=192.168.1.100                          │
│  Интервал: настраивается на сервере                      │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│ ACTIVE MODE (активный)                                   │
│ ────────────────────────                                  │
│ Агент сам отправляет данные на сервер                    │
│                                                          │
│  ┌────────────┐             ┌─────────────┐             │
│  │  Zabbix    │             │   Agent     │             │
│  │  Server    │             │             │             │
│  │            │             │  1. Запрос  │             │
│  │            │←────────────│     списка  │             │
│  │            │  "Что       │     метрик  │             │
│  │            │   собирать?"│             │             │
│  │            │             │             │             │
│  │            │────────────→│             │             │
│  │            │  "CPU,RAM"  │             │             │
│  │            │             │             │             │
│  │            │             │  2. Отправка│             │
│  │            │←────────────│     данных  │             │
│  │            │"CPU=45.2%"  │             │             │
│  └────────────┘             └─────────────┘             │
│                                                          │
│  Параметр: ServerActive=192.168.1.100                    │
│  Преимущество: работает через NAT/firewall               │
└──────────────────────────────────────────────────────────┘
```

### Запуск и проверка

Bash

```
# ═══ Запуск агента ═══
sudo systemctl start zabbix-agent2
sudo systemctl enable zabbix-agent2

# ═══ Проверка статуса ═══
sudo systemctl status zabbix-agent2

# Вывод должен быть:
# ● zabbix-agent2.service - Zabbix Agent 2
#    Loaded: loaded (/lib/systemd/system/zabbix-agent2.service; enabled)
#    Active: active (running) since Mon 2024-01-20 10:00:00 UTC; 5min ago

# ═══ Проверка лога ═══
sudo tail -f /var/log/zabbix/zabbix_agent2.log

# ═══ Проверка порта ═══
sudo ss -tulpn | grep 10050
# tcp   LISTEN  0  128  0.0.0.0:10050  0.0.0.0:*  users:(("zabbix_agent2",pid=1234))

# ═══ Открыть порт в файрволе (если нужно) ═══
sudo ufw allow 10050/tcp
# или для firewalld:
sudo firewall-cmd --permanent --add-port=10050/tcp
sudo firewall-cmd --reload
```

### Тестирование с сервера

Bash

```
# Выполнять на машине с Zabbix Server!

# ═══ Установить утилиту zabbix_get (если нет) ═══
sudo apt install zabbix-get   # Ubuntu
sudo yum install zabbix-get   # RHEL

# ═══ Пинг агента ═══
zabbix_get -s 192.168.1.50 -k "agent.ping"
# Ответ: 1

# ═══ Имя хоста ═══
zabbix_get -s 192.168.1.50 -k "system.hostname"
# Ответ: web-server-01

# ═══ Версия агента ═══
zabbix_get -s 192.168.1.50 -k "agent.version"
# Ответ: 7.0.0

# ═══ CPU нагрузка (1 минута) ═══
zabbix_get -s 192.168.1.50 -k "system.cpu.load[percpu,avg1]"
# Ответ: 0.45

# ═══ Свободная память ═══
zabbix_get -s 192.168.1.50 -k "vm.memory.size[available]"
# Ответ: 8432672768  (в байтах)

# ═══ Свободное место на диске / ═══
zabbix_get -s 192.168.1.50 -k "vfs.fs.size[/,free]"
# Ответ: 50073919488  (в байтах)

# ═══ Количество процессов ═══
zabbix_get -s 192.168.1.50 -k "proc.num[]"
# Ответ: 125
```

### Добавление хоста в веб-интерфейсе

text

```
Zabbix Web UI → Data collection → Hosts → Create host

┌─────────────────────────────────────────────────────────┐
│ HOST                                                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Host name:        web-server-01                         │
│ Visible name:     Web Server 01                         │
│ Host groups:      [Select]  Linux servers               │
│ Description:      Production web server                 │
│                                                         │
├─────────────────────────────────────────────────────────┤
│ INTERFACES                                              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Type:   [Agent] ▼                                       │
│ IP address:  192.168.1.50                               │
│ DNS name:                                               │
│ Connect to:  (•) IP  ( ) DNS                            │
│ Port:        10050                                      │
│                                                         │
├─────────────────────────────────────────────────────────┤
│ TEMPLATES                                               │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Link new templates:                                     │
│ [Linux by Zabbix agent] ← Автоматически добавляет:     │
│                           • CPU мониторинг               │
│                           • Память                       │
│                           • Диски                        │
│                           • Сеть                         │
│                           • Процессы                     │
│                           • Готовые триггеры             │
│                           • Готовые графики              │
│                                                         │
│ [                                     ] [Select]        │
│                                                         │
├─────────────────────────────────────────────────────────┤
│ ENCRYPTION                                              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Connections to host:    ( ) No encryption               │
│ Connections from host:  ( ) No encryption               │
│                                                         │
│                         [Add]    [Cancel]               │
└─────────────────────────────────────────────────────────┘
```

### Пользовательские метрики (UserParameter)

Bash

```
# ═══ Создать файл с пользовательскими параметрами ═══
sudo nano /etc/zabbix/zabbix_agent2.d/custom.conf
```

ini

```
### ФОРМАТ: UserParameter=ключ,команда ###

# Количество процессов nginx
UserParameter=nginx.processes,ps aux | grep nginx | grep -v grep | wc -l

# Размер лог-файла приложения (в байтах)
UserParameter=app.log.size,stat -c %s /var/log/app/app.log

# Количество строк в логе с ERROR за последний час
UserParameter=app.log.errors,grep -c ERROR /var/log/app/app.log

# Количество активных сессий (через API)
UserParameter=app.sessions.active,curl -s http://localhost:8080/api/health | jq -r .sessions

# Проверка доступности порта
UserParameter=port.check[*],ss -tuln | grep -q ":$1 " && echo 1 || echo 0

# Температура CPU (требует lm-sensors)
UserParameter=cpu.temperature,sensors | grep 'Core 0' | awk '{print $3}' | tr -d '+°C'

# С параметрами [*] - $1, $2, $3...
UserParameter=custom.check[*],/usr/local/bin/check.sh "$1" "$2"

# Сложный пример: использование БД
UserParameter=mysql.queries,mysql -u zabbix -pPASSWORD -e "SHOW GLOBAL STATUS LIKE 'Questions'" | awk 'NR==2 {print $2}'
```

Bash

```
# ═══ Перезапуск после изменений ═══
sudo systemctl restart zabbix-agent2

# ═══ Тестирование с сервера ═══
zabbix_get -s 192.168.1.50 -k "nginx.processes"
# Ответ: 4

zabbix_get -s 192.168.1.50 -k "app.log.size"
# Ответ: 1048576

zabbix_get -s 192.168.1.50 -k "port.check[80]"
# Ответ: 1
```

Bash

```
# ═══ Пример скрипта /usr/local/bin/check.sh ═══
#!/bin/bash
# $1 - тип проверки
# $2 - параметр

case "$1" in
    disk)
        df -h "$2" | awk 'NR==2 {print $5}' | tr -d '%'
        ;;
    service)
        systemctl is-active "$2" && echo 1 || echo 0
        ;;
    *)
        echo "Unknown check type"
        exit 1
        ;;
esac
```

---

## 6. Что такое Prometheus Exporter?

### Проблема и решение

text

```
┌──────────────────────────────────────────────────────────┐
│ ПРОБЛЕМА                                                 │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Prometheus собирает метрики по HTTP: GET /metrics       │
│                                                          │
│  НО многие системы НЕ УМЕЮТ отдавать метрики             │
│  в формате Prometheus:                                   │
│                                                          │
│  • MySQL     → свой формат (SHOW STATUS)                 │
│  • PostgreSQL → pg_stat_*                                │
│  • Redis     → INFO                                      │
│  • Nginx     → stub_status                               │
│  • Linux     → /proc, /sys                               │
│                                                          │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│ РЕШЕНИЕ — Exporter                                       │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Маленькая программа-"переводчик", которая:              │
│                                                          │
│  1. Подключается к сервису                               │
│  2. Собирает внутренние метрики                          │
│  3. Конвертирует в формат Prometheus                     │
│  4. Отдаёт на /metrics по HTTP                           │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Как это работает

text

```
┌──────────┐         ┌──────────────┐         ┌────────────┐
│  MySQL   │   SQL   │   mysqld     │  HTTP   │ Prometheus │
│  Server  │←────────│   exporter   │←────────│   Server   │
│  :3306   │ запрос  │   :9104      │  GET    │            │
│          │         │              │ /metrics│            │
│          │────────→│  Переводчик: │────────→│            │
│          │ данные  │  MySQL → Pro │ метрики │            │
│          │         │  metheus fmt │ в Prom  │            │
└──────────┘         └──────────────┘ формате └────────────┘
                            ▲
                            │
                     Exporter делает:
                     1. SHOW GLOBAL STATUS
                     2. Парсит вывод
                     3. Форматирует:
                        mysql_global_status_queries{} 1000
```

### Популярные экспортеры

text

```
┌──────────────────────┬────────┬─────────────────────────────┐
│ Exporter             │ Порт   │ Что мониторит               │
├──────────────────────┼────────┼─────────────────────────────┤
│ node_exporter        │ :9100  │ Linux: CPU, RAM, Disk, Net  │
│ windows_exporter     │ :9182  │ Windows системные метрики   │
│ mysqld_exporter      │ :9104  │ MySQL / MariaDB             │
│ postgres_exporter    │ :9187  │ PostgreSQL                  │
│ redis_exporter       │ :9121  │ Redis                       │
│ mongodb_exporter     │ :9216  │ MongoDB                     │
│ elasticsearch_export │ :9114  │ Elasticsearch               │
│ nginx_exporter       │ :9113  │ Nginx (stub_status)         │
│ apache_exporter      │ :9117  │ Apache (server-status)      │
│ haproxy_exporter     │ :9101  │ HAProxy                     │
│ rabbitmq_exporter    │ :9419  │ RabbitMQ                    │
│ kafka_exporter       │ :9308  │ Apache Kafka                │
│ blackbox_exporter    │ :9115  │ HTTP/DNS/TCP/ICMP ping      │
│ snmp_exporter        │ :9116  │ SNMP устройства             │
│ jmx_exporter         │ :9404  │ Java JMX                    │
│ cadvisor             │ :8080  │ Docker контейнеры           │
│ kube-state-metrics   │ :8080  │ Kubernetes объекты          │
├──────────────────────┴────────┴─────────────────────────────┤
│ ВСЕГО: 200+ официальных exporters                           │
│ https://prometheus.io/docs/instrumenting/exporters/         │
└─────────────────────────────────────────────────────────────┘
```

### Пример: node_exporter

Bash

```
# ═══ Установка ═══
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xzf node_exporter-*.tar.gz
cd node_exporter-*/

# ═══ Запуск ═══
./node_exporter
# INFO ts=2024-01-20T10:00:00.000Z caller=node_exporter.go:115 level=info msg="Starting node_exporter" version="1.7.0"
# INFO ts=2024-01-20T10:00:00.001Z caller=node_exporter.go:116 level=info msg="Build context" build_context="(go=go1.21, platform=linux/amd64)"
# INFO ts=2024-01-20T10:00:00.002Z caller=tls_config.go:274 level=info msg="Listening on" address=:9100

# ═══ Проверка ═══
curl http://localhost:9100/metrics
```

### Что отдаёт node_exporter

prometheus

```
# HELP node_cpu_seconds_total Seconds the CPUs spent in each mode.
# TYPE node_cpu_seconds_total counter
node_cpu_seconds_total{cpu="0",mode="idle"} 78345.67
node_cpu_seconds_total{cpu="0",mode="system"} 1234.56
node_cpu_seconds_total{cpu="0",mode="user"} 5678.90
node_cpu_seconds_total{cpu="0",mode="iowait"} 123.45

# HELP node_memory_MemTotal_bytes Memory information field MemTotal_bytes.
# TYPE node_memory_MemTotal_bytes gauge
node_memory_MemTotal_bytes 16777216000

# HELP node_memory_MemAvailable_bytes Memory information field MemAvailable_bytes.
# TYPE node_memory_MemAvailable_bytes gauge
node_memory_MemAvailable_bytes 8589934592

# HELP node_filesystem_free_bytes Filesystem free space in bytes.
# TYPE node_filesystem_free_bytes gauge
node_filesystem_free_bytes{device="/dev/sda1",fstype="ext4",mountpoint="/"} 50000000000

# HELP node_network_receive_bytes_total Network device statistic receive_bytes.
# TYPE node_network_receive_bytes_total counter
node_network_receive_bytes_total{device="eth0"} 10995116277760

# HELP node_disk_read_bytes_total The total number of bytes read successfully.
# TYPE node_disk_read_bytes_total counter
node_disk_read_bytes_total{device="sda"} 1099511627776

# HELP node_load1 1m load average.
# TYPE node_load1 gauge
node_load1 2.5

# HELP node_boot_time_seconds Node boot time, in unixtime.
# TYPE node_boot_time_seconds gauge
node_boot_time_seconds 1705737600

... и ещё 800+ метрик!
```

### Настройка в Prometheus

YAML

```
# prometheus.yml

global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  # ═══ Node Exporter (системные метрики) ═══
  - job_name: 'linux-servers'
    static_configs:
      - targets:
        - 'server-01:9100'
        - 'server-02:9100'
        - 'server-03:9100'
        labels:
          env: 'production'

  # ═══ MySQL Exporter ═══
  - job_name: 'mysql'
    static_configs:
      - targets:
        - 'db-master:9104'
        - 'db-replica-1:9104'
        - 'db-replica-2:9104'

  # ═══ Nginx Exporter ═══
  - job_name: 'nginx'
    static_configs:
      - targets: ['web-01:9113', 'web-02:9113']

  # ═══ Blackbox Exporter (проверка доступности) ═══
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]  # HTTP GET проверка
    static_configs:
      - targets:
        - https://example.com
        - https://api.example.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115

  # ═══ Kubernetes (автообнаружение!) ═══
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
```

### Написать свой exporter (Python)

Python

```
from prometheus_client import start_http_server, Gauge, Counter, Histogram
import time
import random
import requests

# ═══ Определяем метрики ═══

# Gauge - значение которое может расти и падать
ACTIVE_USERS = Gauge(
    'myapp_active_users_total',
    'Number of currently active users'
)

QUEUE_SIZE = Gauge(
    'myapp_queue_size',
    'Number of items in queue',
    ['queue_name']  # С label
)

# Counter - только растёт
REQUESTS = Counter(
    'myapp_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

ERRORS = Counter(
    'myapp_errors_total',
    'Total errors',
    ['error_type']
)

# Histogram - распределение значений
RESPONSE_TIME = Histogram(
    'myapp_response_time_seconds',
    'HTTP response time in seconds',
    buckets=[0.1, 0.5, 1.0, 2.0, 5.0]
)


# ═══ Функция сбора метрик ═══
def collect_metrics():
    """Собираем данные из приложения"""
    while True:
        try:
            # Получаем данные из API приложения
            response = requests.get('http://localhost:8080/api/stats', timeout=5)
            data = response.json()
            
            # Обновляем метрики
            ACTIVE_USERS.set(data.get('active_users', 0))
            
            # Gauge с labels
            for queue_name, size in data.get('queues', {}).items():
                QUEUE_SIZE.labels(queue_name=queue_name).set(size)
            
            # Инкремент Counter
            REQUESTS.labels(
                method='GET',
                endpoint='/api/stats',
                status='200'
            ).inc()
            
            # Histogram
            RESPONSE_TIME.observe(response.elapsed.total_seconds())
            
        except Exception as e:
            print(f"Error collecting metrics: {e}")
            ERRORS.labels(error_type='collection_failed').inc()
        
        time.sleep(15)  # Собираем каждые 15 сек


if __name__ == '__main__':
    # Запускаем HTTP сервер на порту 8000
    start_http_server(8000)
    print("Exporter started on :8000")
    print("Metrics available at http://localhost:8000/metrics")
    
    # Запускаем сбор метрик
    collect_metrics()
```

Bash

```
# Запуск
python3 myapp_exporter.py
# Exporter started on :8000
# Metrics available at http://localhost:8000/metrics

# Проверка
curl http://localhost:8000/metrics

# Вывод:
# # HELP myapp_active_users_total Number of currently active users
# # TYPE myapp_active_users_total gauge
# myapp_active_users_total 150.0
#
# # HELP myapp_queue_size Number of items in queue
# # TYPE myapp_queue_size gauge
# myapp_queue_size{queue_name="emails"} 42.0
# myapp_queue_size{queue_name="notifications"} 15.0
#
# # HELP myapp_requests_total Total HTTP requests
# # TYPE myapp_requests_total counter
# myapp_requests_total{endpoint="/api/stats",method="GET",status="200"} 100.0
```

### Systemd service для exporter

ini

```
# /etc/systemd/system/myapp-exporter.service

[Unit]
Description=My Application Prometheus Exporter
After=network.target

[Service]
Type=simple
User=prometheus
ExecStart=/usr/local/bin/myapp_exporter
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Bash

```
sudo systemctl daemon-reload
sudo systemctl start myapp-exporter
sudo systemctl enable myapp-exporter
```

---

## 7. Что такое Prometheus Pushgateway?

### Проблема

text

```
┌──────────────────────────────────────────────────────────┐
│ Prometheus работает по PULL модели                       │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Каждые 15 секунд:                                       │
│  Prometheus → GET /metrics → Target                      │
│                                                          │
│  НО что если задача живёт МЕНЬШЕ 15 секунд?              │
│                                                          │
│  ┌──────────────┐                                        │
│  │  Cron Job    │  Запустился → 5 сек работы → Умер     │
│  │  (backup)    │                                        │
│  └──────────────┘                                        │
│        ▲                                                 │
│        │                                                 │
│   Prometheus приходит через 15 сек...                    │
│   а процесса уже нет! Метрики потеряны! 😱               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Решение — Pushgateway

text

```
┌──────────────────────────────────────────────────────────┐
│ Pushgateway — промежуточное хранилище для метрик         │
│              от КОРОТКОЖИВУЩИХ задач                     │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  1. Задача запускается                                   │
│  2. Делает работу                                        │
│  3. PUSH-ит метрики в Pushgateway                        │
│  4. Умирает                                              │
│  5. Prometheus забирает метрики из Pushgateway (pull)    │
│                                                          │
└──────────────────────────────────────────────────────────┘

  ┌─────────────┐                              ┌────────────┐
  │  Cron Job   │  push                        │ Prometheus │
  │  (5 сек)    │──────→ ┌──────────────┐     │   Server   │
  └─────────────┘        │ Pushgateway  │←────│            │
                         │   :9091      │ pull│            │
  ┌─────────────┐  push  │              │     └────────────┘
  │ Batch Job   │──────→ │  Хранит      │
  │ (import)    │        │  метрики     │
  └─────────────┘        │  до          │
                         │  следующего  │
  ┌─────────────┐  push  │  push        │
  │ Lambda      │──────→ │              │
  │ Function    │        │              │
  └─────────────┘        └──────────────┘
                                ▲
                                │ GET /metrics
                         Prometheus забирает
                         метрики по расписанию
```

### Установка

Bash

```
# ═══ Скачать ═══
wget https://github.com/prometheus/pushgateway/releases/download/v1.7.0/pushgateway-1.7.0.linux-amd64.tar.gz
tar xzf pushgateway-*.tar.gz
cd pushgateway-*/

# ═══ Запустить ═══
./pushgateway
# INFO ts=2024-01-20T10:00:00.000Z caller=main.go:80 level=info msg="Starting pushgateway" version="1.7.0"
# INFO ts=2024-01-20T10:00:00.001Z caller=main.go:114 level=info listen_address=:9091

# ═══ Или через Docker ═══
docker run -d -p 9091:9091 prom/pushgateway

# ═══ Проверка ═══
curl http://localhost:9091/metrics
```

### Отправка метрик

Bash

```
# ═══ Простая метрика ═══
echo "backup_duration_seconds 120.5" | \
  curl --data-binary @- http://pushgateway:9091/metrics/job/backup_job

# URL формат:
# /metrics/job/<job_name>/instance/<instance_name>


# ═══ Несколько метрик сразу ═══
cat <<EOF | curl --data-binary @- \
  http://pushgateway:9091/metrics/job/data_import/instance/server-01

# TYPE import_records_total counter
import_records_total 15000
# TYPE import_duration_seconds gauge
import_duration_seconds 345.7
# TYPE import_errors_total counter
import_errors_total 3
# TYPE import_timestamp gauge
import_timestamp $(date +%s)
EOF


# ═══ С дополнительными labels ═══
cat <<EOF | curl --data-binary @- \
  http://pushgateway:9091/metrics/job/deploy/environment/production/version/2.1.0

deployment_status 1
deployment_duration_seconds 45
EOF


# ═══ Удаление метрик ═══
# Удалить все метрики job
curl -X DELETE http://pushgateway:9091/metrics/job/old_job

# Удалить метрики конкретного instance
curl -X DELETE http://pushgateway:9091/metrics/job/backup/instance/server-01

# Удалить ВСЁ
curl -X PUT http://pushgateway:9091/api/v1/admin/wipe
```

### Практический пример — скрипт бэкапа

Bash

```
#!/bin/bash
# /usr/local/bin/backup.sh
# Запуск из cron: 0 2 * * * /usr/local/bin/backup.sh

set -euo pipefail

PUSHGATEWAY="http://pushgateway:9091"
JOB="database_backup"
DATABASE="production_db"
BACKUP_DIR="/backup"

# ═══ Засекаем время ═══
START_TIME=$(date +%s)

# ═══ Функция отправки метрик ═══
send_metrics() {
    local success=$1
    local duration=$2
    local size=$3
    
    cat <<EOF | curl -s --data-binary @- \
      ${PUSHGATEWAY}/metrics/job/${JOB}/database/${DATABASE}
# HELP backup_duration_seconds How long the backup took
# TYPE backup_duration_seconds gauge
backup_duration_seconds ${duration}

# HELP backup_size_bytes Size of backup file in bytes
# TYPE backup_size_bytes gauge
backup_size_bytes ${size}

# HELP backup_success Whether backup succeeded (1=yes, 0=no)
# TYPE backup_success gauge
backup_success ${success}

# HELP backup_last_timestamp Unix timestamp of last backup attempt
# TYPE backup_last_timestamp gauge
backup_last_timestamp $(date +%s)
EOF
}

# ═══ Выполняем бэкап ═══
BACKUP_FILE="${BACKUP_DIR}/${DATABASE}_$(date +%Y%m%d_%H%M%S).sql.gz"

if pg_dump "$DATABASE" | gzip > "$BACKUP_FILE" 2>/tmp/backup_error.log; then
    # Успех
    SUCCESS=1
    SIZE=$(stat -c %s "$BACKUP_FILE")
    
    # Удаляем старые бэкапы (старше 7 дней)
    find "$BACKUP_DIR" -name "${DATABASE}_*.sql.gz" -mtime +7 -delete
else
    # Ошибка
    SUCCESS=0
    SIZE=0
    cat /tmp/backup_error.log >&2
fi

# ═══ Считаем длительность ═══
END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))

# ═══ Отправляем метрики ═══
send_metrics "$SUCCESS" "$DURATION" "$SIZE"

# ═══ Выводим результат ═══
if [ "$SUCCESS" -eq 1 ]; then
    echo "✓ Backup completed: ${DURATION}s, size=$(numfmt --to=iec $SIZE)"
    exit 0
else
    echo "✗ Backup failed after ${DURATION}s"
    exit 1
fi
```

Bash

```
# ═══ Настройка cron ═══
sudo crontab -e

# Бэкап каждый день в 2 часа ночи
0 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
```

### Python пример

Python

```
from prometheus_client import CollectorRegistry, Gauge, push_to_gateway
import time
import sys

# ═══ Создаём отдельный registry ═══
registry = CollectorRegistry()

# ═══ Определяем метрики ═══
duration = Gauge(
    'etl_duration_seconds',
    'Duration of ETL job in seconds',
    registry=registry
)

records_processed = Gauge(
    'etl_records_processed',
    'Number of records processed',
    registry=registry
)

records_failed = Gauge(
    'etl_records_failed',
    'Number of records that failed',
    registry=registry
)

success = Gauge(
    'etl_success',
    'Whether ETL job succeeded (1=yes, 0=no)',
    registry=registry
)

# ═══ Выполняем ETL ═══
start_time = time.time()
processed = 0
failed = 0

try:
    # Здесь ваша бизнес-логика
    for i in range(10000):
        try:
            # process_record(i)
            processed += 1
        except Exception:
            failed += 1
    
    # Успех
    success.set(1)

except Exception as e:
    print(f"ETL failed: {e}", file=sys.stderr)
    success.set(0)

finally:
    # ═══ Устанавливаем значения метрик ═══
    duration.set(time.time() - start_time)
    records_processed.set(processed)
    records_failed.set(failed)
    
    # ═══ Пушим в Pushgateway ═══
    push_to_gateway(
        'pushgateway:9091',
        job='etl_pipeline',
        registry=registry,
        grouping_key={'database': 'production', 'table': 'users'}
    )
    
    print(f"ETL complete: {processed} processed, {failed} failed")
```

### Настройка Prometheus

YAML

```
# prometheus.yml

scrape_configs:
  - job_name: 'pushgateway'
    honor_labels: true  # ← ВАЖНО! Сохранять labels из push
    static_configs:
      - targets: ['pushgateway:9091']
```

### Когда использовать / не использовать

text

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ ✅ ИСПОЛЬЗОВАТЬ:            ┃  ┃ ❌ НЕ ИСПОЛЬЗОВАТЬ:        ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫  ┣━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                            ┃  ┃                           ┃
┃ • Cron jobs                ┃  ┃ • Долгоживущие сервисы    ┃
┃ • Batch задачи             ┃  ┃ • Микросервисы            ┃
┃ • ETL процессы             ┃  ┃ • Web приложения          ┃
┃ • Миграции БД              ┃  ┃ • API серверы             ┃
┃ • CI/CD пайплайны          ┃  ┃ • Замена exporter-ов      ┃
┃ • Lambda/Cloud Functions   ┃  ┃ • Обход файрвола          ┃
┃ • Задачи <15 секунд        ┃  ┃ • Как message queue       ┃
┃ • Скрипты которые умирают  ┃  ┃                           ┃
┃                            ┃  ┃                           ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```

### ⚠️ Важные ограничения

text

```
1. НЕ удаляет метрики автоматически
   ───────────────────────────────
   Если job перестал отправлять метрики,
   старые значения останутся навсегда!
   
   Решение: явное удаление
   curl -X DELETE http://pushgateway:9091/metrics/job/old_job

2. Теряется информация UP/DOWN
   ────────────────────────────
   Prometheus видит что Pushgateway жив (up=1)
   Но НЕ ЗНАЕТ жив ли сам cron job!
   
   Решение: добавлять timestamp и алертить на устаревшие данные
   time() - backup_last_timestamp > 86400  # > 24 часов

3. Single Point of Failure
   ───────────────────────
   Pushgateway упал → метрики не принимаются
   
   Решение: HA setup (несколько Pushgateway + LB)

4. НЕ для постоянного стрима
   ─────────────────────────
   Pushgateway НЕ очередь сообщений!
   Используй только для эпизодических задач

5. Конфликт labels
   ───────────────
   Два push с одинаковыми job/instance → второй перезапишет первый
```

---

## Итоговая архитектура

text

```
┌──────────────────────────────────────────────────────────────┐
│                     ПОЛНАЯ КАРТИНА                            │
└──────────────────────────────────────────────────────────────┘

┌─────────────────────┐         ┌─────────────────────────────┐
│ ДОЛГОЖИВУЩИЕ        │         │ КОРОТКОЖИВУЩИЕ              │
│ СЕРВИСЫ             │         │ ЗАДАЧИ                      │
├─────────────────────┤         ├─────────────────────────────┤
│                     │         │                             │
│ ┌────────────────┐  │         │ ┌────────────────┐          │
│ │ Web App        │  │         │ │ Cron: Backup   │          │
│ │ :8080/metrics  │  │         │ │ (5 минут)      │          │
│ └────────▲───────┘  │         │ └───────┬────────┘          │
│          │          │         │         │ push              │
│          │ pull     │         │         ▼                   │
│          │          │         │ ┌───────────────────┐       │
│ ┌────────┴───────┐  │         │ │   Pushgateway     │       │
│ │ MySQL          │  │         │ │     :9091         │       │
│ └────────┬───────┘  │         │ └──────────▲────────┘       │
│          │          │         │            │ pull           │
│    ┌─────┴──────┐   │         │            │                │
│    │ mysqld     │   │         │ ┌──────────┴────────┐       │
│    │ exporter   │   │         │ │ CI/CD Pipeline    │       │
│    │ :9104      │   │         │ │ (деплой)          │       │
│    └─────▲──────┘   │         │ └───────────────────┘       │
│          │          │         │                             │
└──────────┼──────────┘         └─────────────────────────────┘
           │ pull                            │ pull
           │                                 │
    ┌──────┴─────────────────────────────────┘
    │
┌───▼────────────────────────────────────────────────────────┐
│                 PROMETHEUS SERVER                          │
│                                                            │
│  • Сбор метрик (scrape_interval: 15s)                      │
│  • Хранение в TSDB (retention: 15 days)                    │
│  • Выполнение запросов PromQL                              │
│  • Вычисление алертов                                      │
└────┬──────────────────────────┬────────────────────────────┘
     │                          │
     ▼                          ▼
┌────────────────┐      ┌───────────────────────┐
│    GRAFANA     │      │   ALERTMANAGER        │
│                │      │                       │
│ • Дашборды     │      │ • Группировка         │
│ • Графики      │      │ • Дедупликация        │
│ • Таблицы      │      │ • Роутинг             │
│ • Переменные   │      │                       │
└────────────────┘      └───────┬───────────────┘
                                │
                    ┌───────────┼───────────┐
                    ▼           ▼           ▼
              ┌─────────┐ ┌─────────┐ ┌─────────┐
              │  Slack  │ │  Email  │ │PagerDuty│
              └─────────┘ └─────────┘ └─────────┘
```