## 1. Что такое трассировка запросов и зачем она нужна?

### Проблема в микросервисной архитектуре

text

```
МОНОЛИТ (легко отследить):
─────────────────────────

User → [Monolith App] → Database
         │
         └─ Весь код в одном процессе
            Легко найти где тормозит


МИКРОСЕРВИСЫ (сложно!):
──────────────────────

User → API Gateway → Auth Service → User Service → Database
         │              │              │
         │              │              └→ Cache Service
         │              │
         │              └→ Payment Service → Payment Gateway
         │                      │
         │                      └→ Notification Service → Email API
         │
         └→ Analytics Service → Kafka


❓ Вопросы:
  • Почему запрос медленный? (общее время: 5 секунд)
  • Какой именно сервис тормозит?
  • Какие сервисы вызываются параллельно?
  • Где произошла ошибка в цепочке вызовов?
```

### Что такое трассировка (Distributed Tracing)

text

```
┌──────────────────────────────────────────────────────────┐
│ ТРАССИРОВКА — это сквозное отслеживание запроса через    │
│ всю распределённую систему                               │
│                                                          │
│ Позволяет ответить:                                      │
│ • Какие сервисы обработали запрос?                       │
│ • Сколько времени занял каждый сервис?                   │
│ • Где именно произошла ошибка?                           │
│ • Какие запросы выполнялись параллельно?                 │
└──────────────────────────────────────────────────────────┘
```

### Пример трассировки

text

```
Запрос: POST /api/orders

Trace ID: abc123  (уникальный ID всего запроса)
Duration: 2.5s

┌─────────────────────────────────────────────────────────┐
│ Waterfall (каскад вызовов):                             │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ API Gateway              [████] 0.1s                    │
│   └─ Auth Service        [██] 0.05s                     │
│   └─ Order Service       [████████████] 2.2s            │
│       ├─ User Service    [███] 0.3s                     │
│       │   └─ Database    [██] 0.2s                      │
│       ├─ Inventory       [████] 0.5s                    │
│       │   └─ Database    [███] 0.4s                     │
│       └─ Payment Service [███████████] 1.8s ← МЕДЛЕННО! │
│           ├─ Stripe API  [█████████] 1.5s ← ПРОБЛЕМА!   │
│           └─ Database    [█] 0.1s                       │
│                                                         │
│ 0s    0.5s   1.0s   1.5s   2.0s   2.5s                  │
└─────────────────────────────────────────────────────────┘

Вывод: Payment → Stripe API медленный (1.5 из 2.5 сек)
```

### Зачем нужна трассировка

text

```
✅ DEBUGGING
   Найти медленный сервис в цепочке
   
✅ PERFORMANCE OPTIMIZATION
   Увидеть где тратится время
   
✅ DEPENDENCY MAPPING
   Понять как сервисы связаны
   
✅ ERROR TRACKING
   Найти где именно упал запрос
   
✅ SLA MONITORING
   Контроль latency по каждому компоненту
```

---

## 2. Что такое Trace и Span?

### Иерархия концептов

text

```
┌──────────────────────────────────────────────────────────┐
│ TRACE (трейс)                                            │
│ ───────────────                                          │
│ Весь путь запроса через систему от начала до конца       │
│                                                          │
│ Trace ID: abc123def456                                   │
│ Duration: 2.5s                                           │
│ Spans: 12                                                │
│                                                          │
│   ┌────────────────────────────────────────────────┐     │
│   │ SPAN (спан)                                    │     │
│   │ ─────────────                                  │     │
│   │ Одна операция внутри трейса                    │     │
│   │                                                │     │
│   │ Span ID: span001                               │     │
│   │ Parent Span ID: null (root span)               │     │
│   │ Service: api-gateway                           │     │
│   │ Operation: POST /api/orders                    │     │
│   │ Duration: 2.5s                                 │     │
│   │                                                │     │
│   │   ┌──────────────────────────────────────┐     │     │
│   │   │ CHILD SPAN                           │     │     │
│   │   │ ────────────                         │     │     │
│   │   │ Span ID: span002                     │     │     │
│   │   │ Parent: span001                      │     │     │
│   │   │ Service: auth-service                │     │     │
│   │   │ Operation: validateToken             │     │     │
│   │   │ Duration: 0.05s                      │     │     │
│   │   └──────────────────────────────────────┘     │     │
│   │                                                │     │
│   │   ┌──────────────────────────────────────┐     │     │
│   │   │ CHILD SPAN                           │     │     │
│   │   │ Span ID: span003                     │     │     │
│   │   │ Parent: span001                      │     │     │
│   │   │ Service: order-service               │     │     │
│   │   │ Operation: createOrder               │     │     │
│   │   │ Duration: 2.2s                       │     │     │
│   │   └──────────────────────────────────────┘     │     │
│   └────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────┘
```

### Структура Span

JSON

```
{
  "traceId": "abc123def456",           // ID всего запроса
  "spanId": "span003",                 // ID этой операции
  "parentSpanId": "span001",           // ID родительской операции
  "name": "createOrder",               // Название операции
  "kind": "SERVER",                    // Тип: SERVER, CLIENT, PRODUCER...
  "timestamp": 1705737600000000,       // Время начала (микросекунды)
  "duration": 2200000,                 // Длительность (микросекунды)
  
  "resource": {
    "service.name": "order-service",
    "service.version": "1.2.3",
    "host.name": "order-pod-abc123"
  },
  
  "attributes": {                      // Метаданные
    "http.method": "POST",
    "http.url": "/api/orders",
    "http.status_code": 201,
    "user.id": "user123",
    "order.id": "ord456",
    "db.system": "postgresql"
  },
  
  "events": [                          // События внутри span
    {
      "timestamp": 1705737600500000,
      "name": "validation_completed",
      "attributes": { "result": "ok" }
    }
  ],
  
  "status": {                          // Статус выполнения
    "code": "OK"                       // или ERROR
  }
}
```

### Типы Span

text

```
┌──────────────┬─────────────────────────────────────────┐
│ Тип (kind)   │ Описание                                │
├──────────────┼─────────────────────────────────────────┤
│ SERVER       │ Обработка входящего запроса             │
│              │ Пример: HTTP сервер получил запрос      │
├──────────────┼─────────────────────────────────────────┤
│ CLIENT       │ Исходящий запрос к другому сервису      │
│              │ Пример: HTTP клиент делает запрос       │
├──────────────┼─────────────────────────────────────────┤
│ PRODUCER     │ Отправка сообщения в очередь            │
│              │ Пример: публикация в Kafka              │
├──────────────┼─────────────────────────────────────────┤
│ CONSUMER     │ Получение сообщения из очереди          │
│              │ Пример: чтение из RabbitMQ              │
├──────────────┼─────────────────────────────────────────┤
│ INTERNAL     │ Внутренняя операция                     │
│              │ Пример: вызов функции, DB query         │
└──────────────┴─────────────────────────────────────────┘
```

### Визуализация Trace

text

```
Trace: POST /api/checkout

┌─────────────────────────────────────────────────────────┐
│ Service         Operation          Duration    Status   │
├─────────────────────────────────────────────────────────┤
│ api-gateway     POST /checkout     [███████] 2.5s  OK   │
│ └─ auth         validateToken      [█] 0.1s       OK    │
│ └─ checkout     processCheckout    [██████] 2.2s   OK   │
│    ├─ cart      getCart            [█] 0.2s       OK    │
│    │  └─ redis  GET cart:u123      [█] 0.1s       OK    │
│    ├─ inventory reserveItems       [██] 0.5s      OK    │
│    │  └─ db     UPDATE inventory   [█] 0.3s       OK    │
│    ├─ payment   charge             [████] 1.2s    OK    │
│    │  └─ stripe POST /charges      [███] 1.0s     OK    │
│    └─ shipping  createShipment     [█] 0.3s       OK    │
│       └─ fedex  POST /shipments    [█] 0.2s       OK    │
└─────────────────────────────────────────────────────────┘

Анализ:
✓ Общее время: 2.5s
✓ Самая медленная операция: payment (1.2s)
  └─ Из них Stripe API: 1.0s
✓ Все операции успешны (status=OK)
```

### Context Propagation (передача контекста)

text

```
Как Trace ID и Span ID передаются между сервисами?

┌──────────────┐                    ┌──────────────┐
│  Service A   │                    │  Service B   │
│              │  HTTP Request      │              │
│              │──────────────────→ │              │
│  Trace ID:   │  Headers:          │  Trace ID:   │
│  abc123      │  traceparent:      │  abc123      │
│              │  00-abc123-def456  │              │
│  Span ID:    │                    │  Span ID:    │
│  def456      │                    │  ghi789      │
│              │                    │  Parent:     │
│              │                    │  def456      │
└──────────────┘                    └──────────────┘

W3C Trace Context (стандарт):
traceparent: 00-<trace-id>-<parent-span-id>-<flags>
traceparent: 00-abc123def456-789ghi012jkl-01
             │   │            │            │
             │   │            │            └─ флаги (sampled)
             │   │            └─ parent span id (16 hex)
             │   └─ trace id (32 hex chars)
             └─ version (00)
```

---

## 3. Сравнение инструментов трассировки

### Архитектура систем

text

```
════════════════════════════════════════════════════════════
  JAEGER (Uber, CNCF)
════════════════════════════════════════════════════════════

┌──────────────────────────────────────────────────────────┐
│                                                          │
│  App → Jaeger Agent → Jaeger Collector → Storage → UI   │
│         (sidecar)       (центр. сервис)   (Cassandra    │
│                                            Elasticsearch)│
│                                                          │
│  Компоненты:                                             │
│  • Agent     - принимает spans от приложений (UDP/HTTP)  │
│  • Collector - обработка, валидация, сохранение          │
│  • Query     - API для UI                                │
│  • UI        - веб-интерфейс для просмотра               │
│  • Storage   - Cassandra, Elasticsearch, Badger          │
└──────────────────────────────────────────────────────────┘


════════════════════════════════════════════════════════════
  ZIPKIN (Twitter)
════════════════════════════════════════════════════════════

┌──────────────────────────────────────────────────────────┐
│                                                          │
│  App → Zipkin Collector → Storage → Zipkin UI           │
│         (HTTP/Kafka)       (MySQL, Cassandra,            │
│                             Elasticsearch)               │
│                                                          │
│  Компоненты (всё в одном JAR):                           │
│  • Collector - прием spans                               │
│  • Storage   - MySQL, Cassandra, Elasticsearch, Memory   │
│  • API       - HTTP API для запросов                     │
│  • UI        - веб-интерфейс                             │
└──────────────────────────────────────────────────────────┘


════════════════════════════════════════════════════════════
  TEMPO (Grafana Labs)
════════════════════════════════════════════════════════════

┌──────────────────────────────────────────────────────────┐
│                                                          │
│  App → Tempo → Object Storage → Grafana                 │
│         (OTLP)  (S3, GCS, Azure)  (query via Trace ID)   │
│                                                          │
│  Особенности:                                            │
│  • Только Object Storage (S3, GCS) - дёшево!            │
│  • Нет индексации - поиск ТОЛЬКО по Trace ID            │
│  • Интеграция с Loki и Prometheus (TraceQL)              │
│  • Нативная часть Grafana Stack                          │
└──────────────────────────────────────────────────────────┘
```

### Сравнительная таблица

text

```
┌────────────────────┬──────────────┬──────────────┬──────────────┐
│ Критерий           │ Jaeger       │ Zipkin       │ Tempo        │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Год создания       │ 2015 (Uber)  │ 2012 (Twtr)  │ 2020 (Grafana)│
│ Статус             │ CNCF Graduated│ Open Source │ Open Source  │
│ Язык               │ Go           │ Java         │ Go           │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Протоколы          │ Jaeger       │ Zipkin       │ OTLP ⭐      │
│                    │ Zipkin ⭐    │ JSON, Thrift │ Jaeger       │
│                    │ OpenTelemetry│ OpenTelemetry│ Zipkin       │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Storage            │ Cassandra ⭐ │ MySQL        │ S3/GCS ⭐    │
│                    │ Elasticsearch│ Cassandra    │ Azure Blob   │
│                    │ Badger       │ Elasticsearch│              │
│                    │ Memory       │ Memory       │              │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Индексация         │ Да ⭐        │ Да ⭐        │ Нет!         │
│                    │ (tags, svc)  │ (tags)       │ (только ID)  │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Поиск              │ По тегам ⭐  │ По тегам ⭐  │ Только Trace │
│                    │ По сервису   │ По времени   │ ID           │
│                    │ По операции  │              │              │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ UI                 │ Отличный ⭐  │ Хороший      │ Grafana ⭐   │
│                    │ Standalone   │ Standalone   │ (встроенный) │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Sampling           │ Adaptive ⭐  │ Базовый      │ Tail-based ⭐│
│                    │ Probabilistic│ Probabilistic│ Probabilistic│
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Стоимость хранения │ Средняя      │ Средняя      │ Низкая ⭐    │
│                    │ (Cassandra)  │ (DB)         │ (S3 дёшево)  │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Масштабируемость   │ Отличная ⭐  │ Хорошая      │ Отличная ⭐  │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Интеграция с       │ Prometheus   │ Ограниченная │ Loki ⭐      │
│ метриками/логами   │ (базовая)    │              │ Prometheus ⭐│
│                    │              │              │ (нативная)   │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Query язык         │ GUI          │ GUI          │ TraceQL ⭐   │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Deployment         │ Сложный      │ Простой ⭐   │ Средний      │
│                    │ (много       │ (один JAR)   │              │
│                    │  компонентов)│              │              │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Сообщество         │ Огромное ⭐  │ Большое      │ Растущее     │
│ Документация       │ Отличная ⭐  │ Хорошая      │ Хорошая      │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Use Case           │ Enterprise ⭐ │ Простые      │ Grafana      │
│                    │ Production   │ проекты      │ Stack ⭐     │
│                    │ Large scale  │ Начинающие   │ Cloud-native │
└────────────────────┴──────────────┴──────────────┴──────────────┘
```

### Выбор инструмента

text

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ JAEGER выбирай когда:      ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                            ┃
┃ ✅ Enterprise окружение     ┃
┃ ✅ Нужен поиск по тегам     ┃
┃ ✅ Большие объёмы данных    ┃
┃ ✅ Adaptive sampling        ┃
┃ ✅ Kubernetes              ┃
┃ ✅ CNCF стандарт           ┃
┃ ✅ Standalone решение      ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ ZIPKIN выбирай когда:      ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                            ┃
┃ ✅ Простой старт           ┃
┃ ✅ Легкий деплой (1 JAR)   ┃
┃ ✅ Малые/средние проекты   ┃
┃ ✅ Legacy совместимость    ┃
┃ ✅ Twitter Spring Cloud    ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ TEMPO выбирай когда:       ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                            ┃
┃ ✅ Уже используешь Grafana  ┃
┃ ✅ Нужно дёшево хранить    ┃
┃ ✅ Интеграция с Loki       ┃
┃ ✅ Cloud storage (S3/GCS)  ┃
┃ ✅ TraceQL запросы         ┃
┃ ✅ Единый Grafana Stack    ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```

---

## 4. Что такое OpenTelemetry?

### Определение

text

```
┌──────────────────────────────────────────────────────────┐
│ OpenTelemetry (OTel) — это:                              │
│                                                          │
│ • СТАНДАРТ для сбора телеметрии (метрики, логи, трейсы) │
│ • Единый API и SDK для всех языков                       │
│ • Vendor-neutral (не привязан к конкретному бэкенду)     │
│ • CNCF проект (второй по активности после Kubernetes!)   │
│                                                          │
│ Объединение OpenTracing + OpenCensus                     │
└──────────────────────────────────────────────────────────┘
```

### Проблема до OpenTelemetry

text

```
ДО OpenTelemetry (хаос):
───────────────────────

App написано на Python:
  → Jaeger требует: jaeger-client-python
  → Zipkin требует: py_zipkin
  → Datadog требует: ddtrace
  → New Relic требует: newrelic

Хочешь сменить бэкенд? ПЕРЕПИСЫВАЙ КОД! 😱


ПОСЛЕ OpenTelemetry (универсально):
──────────────────────────────────

App использует OpenTelemetry SDK:
  → opentelemetry-api
  → opentelemetry-sdk
  
Бэкенд выбираешь через конфигурацию:
  export OTEL_EXPORTER=jaeger   ← просто переменная!
  export OTEL_EXPORTER=zipkin
  export OTEL_EXPORTER=tempo
  
Код НЕ меняется! ✅
```

### Архитектура OpenTelemetry

text

```
┌──────────────────────────────────────────────────────────┐
│                   YOUR APPLICATION                        │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  OpenTelemetry API                                 │  │
│  │  (ваш код использует только это)                   │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  OpenTelemetry SDK                                 │  │
│  │  • Traces                                          │  │
│  │  • Metrics                                         │  │
│  │  • Logs (preview)                                  │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  Auto-instrumentation                              │  │
│  │  • HTTP requests                                   │  │
│  │  • Database queries                                │  │
│  │  • gRPC calls                                      │  │
│  │  • Redis, Kafka, etc.                              │  │
│  └────────────────────────────────────────────────────┘  │
└─────────────────────┬────────────────────────────────────┘
                      │ OTLP (OpenTelemetry Protocol)
                      │ gRPC или HTTP
                      ▼
         ┌────────────────────────────┐
         │  OpenTelemetry Collector   │ ← Опционально
         │  (агрегация, фильтрация)   │
         └────────┬───────────────────┘
                  │
      ┌───────────┼───────────┬───────────┐
      ▼           ▼           ▼           ▼
  ┌────────┐ ┌────────┐ ┌────────┐ ┌─────────┐
  │ Jaeger │ │ Tempo  │ │Datadog │ │New Relic│
  └────────┘ └────────┘ └────────┘ └─────────┘
  
  Один код → много бэкендов!
```

### Компоненты OpenTelemetry

text

```
┌─────────────────────────────────────────────────────────┐
│ 1. API                                                  │
│    Интерфейсы для создания spans, метрик, логов         │
│                                                         │
│ 2. SDK                                                  │
│    Реализация API + конфигурация exporters              │
│                                                         │
│ 3. Auto-Instrumentation                                 │
│    Автоматическое добавление трассировки:               │
│    • HTTP frameworks (Flask, FastAPI, Express)          │
│    • Databases (PostgreSQL, MySQL, MongoDB)             │
│    • Messaging (Kafka, RabbitMQ)                        │
│    • gRPC, Redis, Memcached и т.д.                      │
│                                                         │
│ 4. Manual Instrumentation                               │
│    Ручное добавление custom spans                       │
│                                                         │
│ 5. Collector                                            │
│    Центральный компонент для приёма/обработки/экспорта  │
│                                                         │
│ 6. Exporters                                            │
│    Отправка данных в бэкенды (Jaeger, Tempo, etc.)      │
└─────────────────────────────────────────────────────────┘
```

### Поддержка языков

text

```
Официальные SDK:
• C++          ⭐ Stable
• .NET (C#)    ⭐ Stable
• Erlang/Elixir   Stable
• Go           ⭐ Stable
• Java         ⭐ Stable
• JavaScript   ⭐ Stable
• PHP             Stable
• Python       ⭐ Stable
• Ruby            Stable
• Rust         ⭐ Stable
• Swift           Beta

Community SDK:
• Dart, Haskell, Perl, Scala и др.
```

---

## 5. Настройка сбора трассировок

### Python приложение → Jaeger

#### Шаг 1: Установка зависимостей

Bash

```
pip install opentelemetry-api \
            opentelemetry-sdk \
            opentelemetry-instrumentation-flask \
            opentelemetry-instrumentation-requests \
            opentelemetry-exporter-jaeger
```

#### Шаг 2: Код приложения (Flask)

Python

```
from flask import Flask
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
import requests

# ═══ Настройка OpenTelemetry ═══

# Создаём Tracer Provider
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)

# Настраиваем экспорт в Jaeger
jaeger_exporter = JaegerExporter(
    agent_host_name="localhost",  # Jaeger agent
    agent_port=6831,              # UDP port
)

# Добавляем Batch Processor (отправка батчами)
trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(jaeger_exporter)
)

# ═══ Flask приложение ═══
app = Flask(__name__)

# Auto-instrumentation для Flask
FlaskInstrumentor().instrument_app(app)

# Auto-instrumentation для requests библиотеки
RequestsInstrumentor().instrument()


@app.route('/api/users/<user_id>')
def get_user(user_id):
    # Автоматически создаётся span для HTTP запроса
    
    # Создаём custom span для бизнес-логики
    with tracer.start_as_current_span("fetch_user_from_db") as span:
        # Добавляем атрибуты
        span.set_attribute("user.id", user_id)
        span.set_attribute("db.system", "postgresql")
        
        # Имитация запроса к БД
        import time
        time.sleep(0.1)
        
        user = {"id": user_id, "name": "John Doe"}
    
    # Вызов другого сервиса (автоматически создаст child span)
    response = requests.get(f"http://localhost:5001/api/orders/{user_id}")
    
    return {"user": user, "orders": response.json()}


@app.route('/api/orders/<user_id>')
def get_orders(user_id):
    # Ещё один span автоматически
    with tracer.start_as_current_span("fetch_orders_from_db") as span:
        span.set_attribute("user.id", user_id)
        import time
        time.sleep(0.2)
        orders = [{"id": 1, "total": 99.99}]
    
    return {"orders": orders}


if __name__ == '__main__':
    app.run(port=5000, debug=True)
```

#### Шаг 3: Запуск Jaeger (Docker)

Bash

```
docker run -d --name jaeger \
  -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
  -p 5775:5775/udp \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 14268:14268 \
  -p 14250:14250 \
  -p 9411:9411 \
  jaegertracing/all-in-one:latest

# UI доступен на http://localhost:16686
```

#### Шаг 4: Проверка

Bash

```
# Отправить запрос
curl http://localhost:5000/api/users/123

# Открыть Jaeger UI
open http://localhost:16686

# В UI увидишь:
# Service: flask-app
# Trace с 3 spans:
#   1. GET /api/users/123 (Flask span)
#   2. fetch_user_from_db (custom span)
#   3. GET /api/orders/123 (requests span)
```

### Python приложение → Tempo

#### Отличия от Jaeger

Python

```
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter

# Вместо JaegerExporter используем OTLP
otlp_exporter = OTLPSpanExporter(
    endpoint="http://tempo:4317",  # Tempo OTLP endpoint
    insecure=True                  # для dev окружения
)

trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(otlp_exporter)
)
```

#### Docker Compose с Tempo

YAML

```
version: '3.8'

services:
  # Приложение
  app:
    build: .
    ports:
      - "5000:5000"
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://tempo:4317
    depends_on:
      - tempo

  # Tempo
  tempo:
    image: grafana/tempo:latest
    command: [ "-config.file=/etc/tempo.yaml" ]
    ports:
      - "4317:4317"  # OTLP gRPC
      - "4318:4318"  # OTLP HTTP
      - "3200:3200"  # Tempo HTTP
    volumes:
      - ./tempo.yaml:/etc/tempo.yaml

  # Grafana
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
    volumes:
      - ./grafana-datasources.yaml:/etc/grafana/provisioning/datasources/datasources.yaml
```

#### tempo.yaml

YAML

```
server:
  http_listen_port: 3200

distributor:
  receivers:
    otlp:
      protocols:
        grpc:
          endpoint: 0.0.0.0:4317
        http:
          endpoint: 0.0.0.0:4318

storage:
  trace:
    backend: local
    local:
      path: /tmp/tempo/traces

compactor:
  compaction:
    block_retention: 48h
```

#### grafana-datasources.yaml

YAML

```
apiVersion: 1

datasources:
  - name: Tempo
    type: tempo
    access: proxy
    url: http://tempo:3200
    isDefault: true
```

---

## 6. Настройка в Kubernetes

### С использованием OpenTelemetry Operator

#### Шаг 1: Установка оператора

Bash

```
# Добавить Helm repo
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update

# Установить cert-manager (требуется для оператора)
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# Установить оператор
helm install opentelemetry-operator open-telemetry/opentelemetry-operator \
  --namespace opentelemetry-operator-system \
  --create-namespace
```

#### Шаг 2: Деплой Jaeger

YAML

```
# jaeger.yaml
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: jaeger
  namespace: observability
spec:
  strategy: allInOne  # для dev; для prod используй 'production'
  allInOne:
    image: jaegertracing/all-in-one:latest
    options:
      log-level: debug
  storage:
    type: memory  # для prod: elasticsearch или cassandra
  ingress:
    enabled: true
    hosts:
      - jaeger.example.com
  ui:
    options:
      dependencies:
        menuEnabled: true
```

Bash

```
kubectl create namespace observability
kubectl apply -f jaeger.yaml
```

#### Шаг 3: OpenTelemetry Collector

YAML

```
# otel-collector.yaml
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: otel-collector
  namespace: observability
spec:
  mode: deployment  # или daemonset, sidecar
  config: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318
    
    processors:
      batch:
        timeout: 10s
        send_batch_size: 1024
      
      # Добавление resource attributes
      resource:
        attributes:
          - key: cluster.name
            value: prod-cluster
            action: upsert
      
      # Sampling (сохраняем только 10%)
      probabilistic_sampler:
        sampling_percentage: 10
    
    exporters:
      jaeger:
        endpoint: jaeger-collector.observability.svc:14250
        tls:
          insecure: true
      
      # Дополнительно в Tempo
      otlp/tempo:
        endpoint: tempo.observability.svc:4317
        tls:
          insecure: true
      
      # Логирование (для дебага)
      logging:
        loglevel: debug
    
    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [batch, resource, probabilistic_sampler]
          exporters: [jaeger, otlp/tempo, logging]
```

Bash

```
kubectl apply -f otel-collector.yaml
```

#### Шаг 4: Инструментация приложения

YAML

```
# app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
      annotations:
        # ═══ Auto-instrumentation через Operator ═══
        instrumentation.opentelemetry.io/inject-python: "true"
    spec:
      containers:
      - name: app
        image: myapp:latest
        env:
          # ═══ OpenTelemetry конфигурация ═══
          - name: OTEL_SERVICE_NAME
            value: "my-app"
          
          - name: OTEL_EXPORTER_OTLP_ENDPOINT
            value: "http://otel-collector.observability.svc:4317"
          
          - name: OTEL_RESOURCE_ATTRIBUTES
            value: "deployment.environment=production,service.version=1.2.3"
          
          - name: OTEL_TRACES_SAMPLER
            value: "parentbased_traceidratio"
          
          - name: OTEL_TRACES_SAMPLER_ARG
            value: "0.1"  # 10% sampling
```

#### Шаг 5: Инструментация через Sidecar

YAML

```
# instrumentation.yaml
apiVersion: opentelemetry.io/v1alpha1
kind: Instrumentation
metadata:
  name: my-instrumentation
  namespace: default
spec:
  exporter:
    endpoint: http://otel-collector.observability.svc:4317
  
  propagators:
    - tracecontext
    - baggage
  
  sampler:
    type: parentbased_traceidratio
    argument: "0.1"
  
  python:
    image: ghcr.io/open-telemetry/opentelemetry-operator/autoinstrumentation-python:latest
    env:
      - name: OTEL_TRACES_EXPORTER
        value: otlp
      - name: OTEL_METRICS_EXPORTER
        value: none
      - name: OTEL_LOGS_EXPORTER
        value: none
  
  # Для других языков
  java:
    image: ghcr.io/open-telemetry/opentelemetry-operator/autoinstrumentation-java:latest
  
  nodejs:
    image: ghcr.io/open-telemetry/opentelemetry-operator/autoinstrumentation-nodejs:latest
```

### Полный пример (Python FastAPI в K8s)

Python

```
# main.py
from fastapi import FastAPI
from opentelemetry import trace
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
from opentelemetry.sdk.resources import SERVICE_NAME, Resource
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
import os

# ═══ Настройка трассировки ═══
resource = Resource(attributes={
    SERVICE_NAME: os.getenv("OTEL_SERVICE_NAME", "fastapi-app")
})

provider = TracerProvider(resource=resource)
processor = BatchSpanProcessor(
    OTLPSpanExporter(
        endpoint=os.getenv("OTEL_EXPORTER_OTLP_ENDPOINT", "http://localhost:4317"),
        insecure=True
    )
)
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)

tracer = trace.get_tracer(__name__)

# ═══ FastAPI приложение ═══
app = FastAPI()

# Auto-instrumentation
FastAPIInstrumentor.instrument_app(app)


@app.get("/")
async def root():
    return {"message": "Hello World"}


@app.get("/users/{user_id}")
async def get_user(user_id: str):
    # Custom span
    with tracer.start_as_current_span("database_query") as span:
        span.set_attribute("db.system", "postgresql")
        span.set_attribute("user.id", user_id)
        
        # Имитация DB запроса
        import asyncio
        await asyncio.sleep(0.1)
        
        return {"user_id": user_id, "name": "John Doe"}
```

Dockerfile

```
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY main.py .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

YAML

```
# k8s-deployment.yaml
apiVersion: v1
kind: Service
metadata:
  name: fastapi-app
spec:
  selector:
    app: fastapi-app
  ports:
    - port: 80
      targetPort: 8000
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fastapi-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: fastapi-app
  template:
    metadata:
      labels:
        app: fastapi-app
    spec:
      containers:
      - name: app
        image: fastapi-app:latest
        ports:
        - containerPort: 8000
        env:
          - name: OTEL_SERVICE_NAME
            value: "fastapi-app"
          - name: OTEL_EXPORTER_OTLP_ENDPOINT
            value: "http://otel-collector.observability.svc:4317"
          - name: OTEL_RESOURCE_ATTRIBUTES
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
        resources:
          limits:
            memory: "256Mi"
            cpu: "500m"
```

---

## 7. Observability Stack — Сравнение решений

### Три столпа Observability

text

```
┌──────────────────────────────────────────────────────────┐
│                  OBSERVABILITY                            │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐            │
│  │ METRICS │     │  LOGS   │     │ TRACES  │            │
│  └─────────┘     └─────────┘     └─────────┘            │
│      │               │               │                   │
│      │               │               │                   │
│      └───────────────┴───────────────┘                   │
│                      │                                   │
│              ┌───────▼────────┐                          │
│              │  CORRELATION   │                          │
│              │  (связывание)  │                          │
│              └────────────────┘                          │
│                                                          │
│  Пример:                                                 │
│  Trace ID → найти логи с этим Trace ID                   │
│  Trace ID → найти метрики в это же время                 │
│  Метрика скачок → найти traces и логи                    │
└──────────────────────────────────────────────────────────┘
```

### Grafana Stack (Open Source)

text

```
┌──────────────────────────────────────────────────────────┐
│              GRAFANA STACK (LGTM)                         │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────┐   ┌────────────┐   ┌────────────┐       │
│  │   Loki     │   │  Grafana   │   │   Tempo    │       │
│  │  (Logs)    │   │   (UI)     │   │  (Traces)  │       │
│  └─────▲──────┘   └─────▲──────┘   └─────▲──────┘       │
│        │                │                │              │
│        │          ┌─────┴──────┐         │              │
│        │          │ Prometheus │         │              │
│        │          │ (Metrics)  │         │              │
│        │          └────────────┘         │              │
│        │                                 │              │
│        └─────────────┬───────────────────┘              │
│                      │                                   │
│               ┌──────▼────────┐                          │
│               │  Application  │                          │
│               │  (OTel SDK)   │                          │
│               └───────────────┘                          │
│                                                          │
│  Компоненты:                                             │
│  • Prometheus - метрики                                  │
│  • Loki       - логи (индексирует только labels!)        │
│  • Tempo      - трейсы (хранит в S3, без индексации)     │
│  • Grafana    - единый UI для всего                      │
│                                                          │
│  Преимущества:                                           │
│  ✅ Open Source                                           │
│  ✅ Дёшево (S3 storage)                                   │
│  ✅ Единый UI                                             │
│  ✅ TraceQL, LogQL, PromQL                                │
│  ✅ Нативная интеграция между компонентами                │
│                                                          │
│  Недостатки:                                             │
│  ❌ Нужно собирать и настраивать самому                   │
│  ❌ Tempo: поиск только по Trace ID                       │
│  ❌ Нет managed решения (только Grafana Cloud платно)     │
└──────────────────────────────────────────────────────────┘
```

### New Relic (SaaS)

text

```
┌──────────────────────────────────────────────────────────┐
│                   NEW RELIC                               │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │         New Relic Platform (SaaS)                  │  │
│  │                                                    │  │
│  │  • APM (Application Performance Monitoring)       │  │
│  │  • Infrastructure Monitoring                      │  │
│  │  • Logs                                           │  │
│  │  • Distributed Tracing                            │  │
│  │  • Synthetics (мониторинг доступности)            │  │
│  │  • Browser Monitoring                             │  │
│  │  • Mobile Monitoring                              │  │
│  │  • Alerts & Incidents                             │  │
│  └────────────────────────────────────────────────────┘  │
│                          ▲                               │
│                          │ New Relic Agent              │
│                          │                               │
│                   ┌──────┴───────┐                       │
│                   │ Application  │                       │
│                   │ (Auto-instr) │                       │
│                   └──────────────┘                       │
│                                                          │
│  Преимущества:                                           │
│  ✅ Managed (не нужно поддерживать)                       │
│  ✅ Auto-instrumentation out of box                       │
│  ✅ Мощный поиск и анализ                                 │
│  ✅ AI/ML аномалии                                        │
│  ✅ Enterprise support                                    │
│                                                          │
│  Недостатки:                                             │
│  ❌ Дорого (от $99/мес + per GB)                          │
│  ❌ Vendor lock-in                                        │
│  ❌ Данные в облаке New Relic                             │
└──────────────────────────────────────────────────────────┘
```

### Datadog (SaaS)

text

```
┌──────────────────────────────────────────────────────────┐
│                     DATADOG                               │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │           Datadog Platform (SaaS)                  │  │
│  │                                                    │  │
│  │  • Infrastructure Monitoring                      │  │
│  │  • APM & Distributed Tracing ⭐                    │  │
│  │  • Logs                                           │  │
│  │  • Real User Monitoring (RUM)                     │  │
│  │  • Synthetics                                     │  │
│  │  • Network Monitoring                             │  │
│  │  • Security Monitoring                            │  │
│  │  • Serverless Monitoring (Lambda, etc.)           │  │
│  │  • Database Monitoring                            │  │
│  │  • Incident Management                            │  │
│  └────────────────────────────────────────────────────┘  │
│                          ▲                               │
│                          │ Datadog Agent                │
│                          │                               │
│                   ┌──────┴───────┐                       │
│                   │ Application  │                       │
│                   │ (DD Tracer)  │                       │
│                   └──────────────┘                       │
│                                                          │
│  Преимущества:                                           │
│  ✅ Самый мощный функционал                               │
│  ✅ Отличная визуализация                                 │
│  ✅ Лучший APM на рынке                                   │
│  ✅ Интеграции со всем                                    │
│  ✅ Корреляция traces ↔ metrics ↔ logs ⭐                 │
│                                                          │
│  Недостатки:                                             │
│  ❌ ОЧЕНЬ дорого (от $15/host/мес + $1.27/GB logs)        │
│  ❌ Сильный vendor lock-in                                │
│  ❌ Данные только в облаке Datadog                        │
└──────────────────────────────────────────────────────────┘
```

### Сравнительная таблица

text

```
┌──────────────────┬──────────────┬─────────────┬─────────────┐
│ Критерий         │ Grafana Stack│ New Relic   │ Datadog     │
├──────────────────┼──────────────┼─────────────┼─────────────┤
│ Тип              │ Open Source  │ SaaS        │ SaaS        │
│ Хостинг          │ Self-hosted  │ Cloud only  │ Cloud only  │
├──────────────────┼──────────────┼─────────────┼─────────────┤
│ METRICS          │ Prometheus⭐ │ Да          │ Да ⭐       │
│ LOGS             │ Loki         │ Да          │ Да ⭐       │
│ TRACES           │ Tempo/Jaeger │ Да          │ Да ⭐       │
├──────────────────┼──────────────┼─────────────┼─────────────┤
│ Корреляция       │ TraceID ⭐   │ Да          │ Отличная ⭐ │
│ (logs↔traces)    │              │             │             │
├──────────────────┼──────────────┼─────────────┼─────────────┤
│ Auto-instr       │ OTel ⭐      │ Agents      │ Agents ⭐   │
│                  │              │             │             │
├──────────────────┼──────────────┼─────────────┼─────────────┤
│ Query Language   │ PromQL ⭐    │ NRQL        │ DQL         │
│                  │ LogQL ⭐     │             │             │
│                  │ TraceQL ⭐   │             │             │
├──────────────────┼──────────────┼─────────────┼─────────────┤
│ Дашборды         │ Grafana ⭐   │ Встроенные  │ Встроенные⭐│
├──────────────────┼──────────────┼─────────────┼─────────────┤
│ Алертинг         │ Да           │ Да ⭐       │ Да ⭐       │
│ Incident Mgmt    │ OnCall (доп) │ Да          │ Да ⭐       │
├──────────────────┼──────────────┼─────────────┼─────────────┤
│ Стоимость        │ Бесплатно ⭐ │ $$          │ $$$         │
│                  │ (self-host)  │ ~$99+/мес   │ ~$15+/мес   │
├──────────────────┼──────────────┼─────────────┼─────────────┤
│ Vendor Lock-in   │ Нет ⭐       │ Да          │ Да          │
│ Миграция         │ Легко        │ Сложно      │ Сложно      │
├──────────────────┼──────────────┼─────────────┼─────────────┤
│ Масштабиро       │ Отлично ⭐   │ Unlimited   │ Unlimited   │
│                  │ (K8s)        │             │             │
├──────────────────┼──────────────┼─────────────┼─────────────┤
│ Кривая обучения  │ Средняя      │ Низкая ⭐   │ Низкая ⭐   │
│ Сложность setup  │ Высокая      │ Простая ⭐  │ Простая ⭐  │
├──────────────────┼──────────────┼─────────────┼─────────────┤
│ Use Case         │ Любые ⭐     │ Enterprise  │ Enterprise  │
│                  │ K8s          │ Cloud apps  │ Cloud apps⭐│
│                  │ Self-host    │             │             │
└──────────────────┴──────────────┴─────────────┴─────────────┘
```

### Выбор решения

text

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ GRAFANA STACK выбирай когда:                          ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                                                       ┃
┃ ✅ Ограниченный бюджет                                 ┃
┃ ✅ Данные должны быть on-premise                       ┃
┃ ✅ Уже используешь Prometheus/Kubernetes               ┃
┃ ✅ Нужен полный контроль                               ┃
┃ ✅ Есть DevOps команда для поддержки                   ┃
┃ ✅ Open Source принципиально важен                     ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ NEW RELIC выбирай когда:                              ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                                                       ┃
┃ ✅ Нужно быстро внедрить                               ┃
┃ ✅ Нет команды для поддержки infrastructure            ┃
┃ ✅ Средний бюджет                                      ┃
┃ ✅ Фокус на Application Performance                    ┃
┃ ✅ AI/ML аномалии важны                                ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ DATADOG выбирай когда:                                ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                                                       ┃
┃ ✅ Enterprise с большим бюджетом                       ┃
┃ ✅ Нужен лучший в классе APM                           ┃
┃ ✅ Важна корреляция всех сигналов                      ┃
┃ ✅ Сложная cloud-native инфраструктура                 ┃
┃ ✅ Безопасность критична (Security Monitoring)         ┃
┃ ✅ Нужны готовые интеграции со всем                    ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```

---

## 8. Объединение трассировок, метрик и логов в Grafana

### Концепция корреляции

text

```
┌──────────────────────────────────────────────────────────┐
│ ПРОБЛЕМА: три отдельных инструмента                      │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Метрики (Prometheus):   error_rate = 25%                │
│                            │                             │
│                            ▼                             │
│                   "Что-то сломалось!"                    │
│                            │                             │
│                   ┌────────┴────────┐                    │
│                   │                 │                    │
│  Логи (Loki):     │   Трейсы (Tempo):                    │
│  Где ошибки?      │   Какой путь?                        │
│  Но нет связи!    │   Но нет связи!                      │
│                                                          │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│ РЕШЕНИЕ: корреляция через Trace ID                       │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  1. Запрос приходит → создаётся Trace ID: abc123         │
│                                                          │
│  2. Trace ID передаётся везде:                           │
│     • В spans (Tempo)                                    │
│     • В logs   (добавляем в каждый лог)                  │
│     • В metrics (label: trace_id)                        │
│                                                          │
│  3. В Grafana:                                           │
│     Метрика показывает проблему →                        │
│     Клик → открывает traces за этот период →             │
│     Клик на trace → открывает логи с этим trace_id       │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Настройка корреляции в приложении

Python

```
from opentelemetry import trace
from opentelemetry.trace import SpanContext
import logging
import json

# ═══ Настройка логирования с Trace ID ═══

class TraceIDFilter(logging.Filter):
    """Добавляет trace_id и span_id в каждый лог"""
    
    def filter(self, record):
        span = trace.get_current_span()
        if span != trace.INVALID_SPAN:
            ctx = span.get_span_context()
            record.trace_id = format(ctx.trace_id, '032x')
            record.span_id = format(ctx.span_id, '016x')
        else:
            record.trace_id = '0' * 32
            record.span_id = '0' * 16
        return True

# Создаём logger
logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

# JSON formatter для Loki
class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            'timestamp': self.formatTime(record),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
            'trace_id': getattr(record, 'trace_id', ''),
            'span_id': getattr(record, 'span_id', ''),
            'service': 'my-app',
        }
        
        if record.exc_info:
            log_data['exception'] = self.formatException(record.exc_info)
        
        return json.dumps(log_data)

handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
handler.addFilter(TraceIDFilter())
logger.addHandler(handler)


# ═══ Использование в коде ═══

@app.route('/api/users/<user_id>')
def get_user(user_id):
    # Логи автоматически содержат trace_id!
    logger.info(f"Fetching user {user_id}")
    
    try:
        user = db.get_user(user_id)
        logger.info(f"User {user_id} fetched successfully")
        return user
    except Exception as e:
        logger.error(f"Error fetching user {user_id}: {e}", exc_info=True)
        raise
```

### Конфигурация Grafana Data Sources

YAML

```
# grafana-datasources.yaml
apiVersion: 1

datasources:
  # ═══ Prometheus ═══
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    jsonData:
      timeInterval: 15s
      # Связь с Tempo через exemplars
      exemplarTraceIdDestinations:
        - name: trace_id
          datasourceUid: tempo

  # ═══ Loki ═══
  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    jsonData:
      # Связь с Tempo
      derivedFields:
        - datasourceUid: tempo
          matcherRegex: '"trace_id":"(\w+)"'
          name: TraceID
          url: '$${__value.raw}'

  # ═══ Tempo ═══
  - name: Tempo
    type: tempo
    access: proxy
    url: http://tempo:3200
    uid: tempo
    jsonData:
      # Связь с Loki
      tracesToLogs:
        datasourceUid: loki
        filterByTraceID: true
        filterBySpanID: false
        mapTagNamesEnabled: true
        tags: ['service', 'instance']
        
      # Связь с Prometheus
      tracesToMetrics:
        datasourceUid: prometheus
        tags:
          - key: service.name
            value: service
        queries:
          - name: 'Request rate'
            query: 'rate(http_requests_total{service="$${__tags.service}"}[5m])'
      
      # Service Graph (визуализация зависимостей)
      serviceMap:
        datasourceUid: prometheus
      
      # Node Graph
      nodeGraph:
        enabled: true

  # ═══ Pyroscope (Profiling) ═══
  - name: Pyroscope
    type: grafana-pyroscope-datasource
    access: proxy
    url: http://pyroscope:4040
```

### Пример корреляции в действии

#### 1. Начинаем с метрики (Prometheus)

promql

```
# Dashboard: HTTP Errors

# Panel 1: Error rate
sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)

# Видим spike на service="payment"
# Кликаем на график → "View traces"
```

#### 2. Переходим к трейсам (Tempo)

text

```
Grafana автоматически открывает Tempo с фильтром:
  service.name = "payment"
  timestamp = время spike

Видим список traces:
┌────────────────┬──────────┬──────────┬─────────┐
│ Trace ID       │ Duration │ Spans    │ Errors  │
├────────────────┼──────────┼──────────┼─────────┤
│ abc123...      │ 5.2s     │ 12       │ 1       │ ← Медленно!
│ def456...      │ 0.3s     │ 8        │ 0       │
│ ghi789...      │ 15.8s    │ 15       │ 1       │ ← Очень медленно!
└────────────────┴──────────┴──────────┴─────────┘

Кликаем на ghi789...
```

#### 3. Анализируем Trace

text

```
Trace: ghi789...
Duration: 15.8s

┌─────────────────────────────────────────────────────────┐
│ Service         Operation          Duration    Status   │
├─────────────────────────────────────────────────────────┤
│ api-gateway     POST /checkout     [██████] 15.8s ERROR │
│ └─ payment      charge             [██████] 15.2s ERROR │
│    └─ stripe    POST /charges      [█████] 14.9s TIMEOUT│
│       └─ http   connect            [█████] 14.9s ERROR  │
└─────────────────────────────────────────────────────────┘

Видим: Stripe API timeout!
Кликаем "View logs for this trace"
```

#### 4. Смотрим логи (Loki)

text

```
Grafana автоматически открывает Loki с запросом:
  {service="payment"} | json | trace_id="ghi789..."

Логи:
[15:23:45] INFO: Processing payment for order=ORD123
[15:23:46] INFO: Calling Stripe API charge endpoint
[15:23:47] WARN: Stripe API slow response (1s)
[15:23:50] WARN: Stripe API slow response (5s)
[15:24:00] ERROR: Stripe API timeout after 15s
[15:24:00] ERROR: Payment failed for order=ORD123
           exception: requests.exceptions.Timeout
           trace_id: ghi789...
           stripe_request_id: req_abc123

Кликаем на stripe_request_id → открывается Stripe Dashboard
```

#### 5. Корень проблемы найден!

text

```
Путь анализа:
  Метрика spike → Trace показал Stripe timeout → 
  Лог показал stripe_request_id → Stripe Dashboard → 
  Их API упал!

Действие:
  1. Включить retry механизм
  2. Добавить circuit breaker
  3. Уведомить Stripe support
```

### Пример дашборда с корреляцией

JSON

```
{
  "dashboard": {
    "title": "Service Overview with Correlation",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "exemplar": true  // ← показывает traces как точки
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "graph",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])",
            "exemplar": true
          }
        ],
        "fieldConfig": {
          "defaults": {
            "links": [
              {
                "title": "View Traces",
                "url": "/explore?left={\"datasource\":\"tempo\",\"queries\":[{\"query\":\"{service=\\\"${__field.labels.service}\\\"}\"}],\"range\":{\"from\":\"${__from}\",\"to\":\"${__to}\"}}"
              }
            ]
          }
        }
      },
      {
        "title": "Recent Traces",
        "type": "traces",
        "datasource": "Tempo",
        "targets": [
          {
            "query": "{service=\"$service\"}"
          }
        ]
      },
      {
        "title": "Logs",
        "type": "logs",
        "datasource": "Loki",
        "targets": [
          {
            "expr": "{service=\"$service\"} | json | trace_id=\"$trace_id\""
          }
        ]
      }
    ]
  }
}
```

### TraceQL — язык запросов Tempo

text

```
# Найти медленные traces
{ duration > 5s }

# Traces с ошибками
{ status = error }

# По сервису и операции
{ service.name = "payment" && name = "charge" }

# По HTTP статусу
{ http.status_code >= 500 }

# Комбинированный запрос
{
  service.name = "payment" &&
  http.status_code >= 500 &&
  duration > 1s
}
| by(http.method)

# С метриками
{ service.name = "payment" }
| rate() by(http.status_code)
```

### Итоговая схема корреляции

text

```
┌──────────────────────────────────────────────────────────┐
│                      GRAFANA                              │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Dashboard: Production Overview                     │  │
│  ├────────────────────────────────────────────────────┤  │
│  │                                                    │  │
│  │  [GRAPH: Error Rate]  ← Spike! Click →            │  │
│  │     ↓ exemplar link                                │  │
│  │  [LIST: Traces]       ← Click on trace →          │  │
│  │     ↓ trace_id                                     │  │
│  │  [TRACE: Waterfall]   ← Click "View logs" →       │  │
│  │     ↓ trace_id                                     │  │
│  │  [LOGS: Filtered]     ← Root cause found!         │  │
│  │                                                    │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  Связи:                                                  │
│  • Prometheus exemplars → Tempo (trace_id)               │
│  • Tempo → Loki (trace_id in logs)                       │
│  • Tempo → Prometheus (service, time range)              │
│  • Logs → Tempo (trace_id extracted)                     │
└──────────────────────────────────────────────────────────┘
```