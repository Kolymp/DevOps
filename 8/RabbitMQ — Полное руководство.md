## 1. Основные концепции RabbitMQ

### Введение в RabbitMQ

RabbitMQ — это **message broker** (брокер сообщений), который реализует протокол **AMQP** (Advanced Message Queuing Protocol). В отличие от Kafka, которая является распределённым логом событий, RabbitMQ — это **традиционный message queue**, ориентированный на **маршрутизацию сообщений** и **гарантированную доставку**.

**Философия RabbitMQ:**

text

```
Kafka:    "Я — журнал событий. Храню всё, читай что хочешь."
RabbitMQ: "Я — умный почтальон. Знаю, КОМУ и КАК доставить каждое письмо."
```

RabbitMQ особенно хороша в сценариях, где нужна:

- **Сложная маршрутизация** сообщений (по паттернам, заголовкам, приоритетам)
- **Гарантированная доставка** с подтверждениями
- **Низкая латентность** (микросекунды)
- **Разнообразные паттерны** обмена сообщениями

---

### Publisher (Издатель / Продюсер)

**Publisher** (также называемый **Producer**) — это приложение или сервис, который **создаёт и отправляет** сообщения в RabbitMQ.

**Подробное описание:**

Publisher — это **отправная точка** жизненного цикла сообщения. Он создаёт сообщение, задаёт его свойства (приоритет, TTL, тип контента) и отправляет в **Exchange** (биржу обмена). Publisher **не знает**, в какую конкретно очередь попадёт сообщение — это решает Exchange на основе правил маршрутизации.

text

```
┌────────────────────────────────────────────────────────────────────┐
│                         PUBLISHER                                  │
│                                                                    │
│  Publisher создаёт сообщение:                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Message:                                                     │  │
│  │   Body:    {"orderId": 12345, "amount": 99.99}               │  │
│  │   Headers: {priority: 5, timestamp: 1705318800}              │  │
│  │   Properties:                                                │  │
│  │     - content_type: "application/json"                       │  │
│  │     - delivery_mode: 2 (persistent)                          │  │
│  │     - routing_key: "order.created.eu"                        │  │
│  │     - expiration: "60000" (TTL 60 сек)                       │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  Publisher отправляет в EXCHANGE (не в очередь напрямую!)          │
│  ────────────────────────────────────────────────────────▶         │
│                                                            │       │
│                                                   [EXCHANGE]       │
│                                                            │       │
│                              Exchange решает, в какую      │       │
│                              очередь (или очереди)         │       │
│                              направить сообщение           │       │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Анатомия Publisher:**

Python

```
# Пример на Python (библиотека pika)
import pika
import json

# ════════════════════════════════════════════════════════════
# ПОДКЛЮЧЕНИЕ К RABBITMQ
# ════════════════════════════════════════════════════════════

credentials = pika.PlainCredentials('username', 'password')
parameters = pika.ConnectionParameters(
    host='rabbitmq-server.example.com',
    port=5672,
    virtual_host='/',  # виртуальный хост (изоляция)
    credentials=credentials,
    heartbeat=60,      # heartbeat для keep-alive соединения
    blocked_connection_timeout=300
)

connection = pika.BlockingConnection(parameters)
channel = connection.channel()

# ════════════════════════════════════════════════════════════
# ОБЪЯВЛЕНИЕ EXCHANGE (если ещё не существует)
# ════════════════════════════════════════════════════════════

channel.exchange_declare(
    exchange='orders_exchange',
    exchange_type='topic',    # тип exchange (direct/topic/fanout/headers)
    durable=True,             # exchange переживёт перезагрузку брокера
    auto_delete=False         # не удалять автоматически
)

# ════════════════════════════════════════════════════════════
# СОЗДАНИЕ И ОТПРАВКА СООБЩЕНИЯ
# ════════════════════════════════════════════════════════════

message = {
    "orderId": 12345,
    "customerId": "user-789",
    "amount": 99.99,
    "currency": "USD",
    "timestamp": "2024-01-15T10:30:00Z"
}

message_body = json.dumps(message)

# Свойства сообщения
properties = pika.BasicProperties(
    content_type='application/json',   # тип контента
    content_encoding='utf-8',
    
    # delivery_mode:
    # 1 = transient (в памяти, быстро, НЕ переживёт перезагрузку)
    # 2 = persistent (на диске, медленнее, переживёт перезагрузку)
    delivery_mode=2,
    
    priority=5,                        # приоритет (0-255, если очередь поддерживает)
    correlation_id='abc-123',          # для связывания запроса/ответа
    reply_to='response_queue',         # куда отправить ответ (RPC паттерн)
    expiration='60000',                # TTL сообщения (60 секунд в миллисекундах)
    message_id='msg-' + str(uuid.uuid4()),
    timestamp=int(time.time()),
    type='order.created',              # тип события
    user_id='order-service',           # ID пользователя (должен совпадать с authenticated user)
    app_id='order-service-v1.2.3',    # ID приложения
    
    # Пользовательские заголовки
    headers={
        'x-region': 'eu',
        'x-priority-customer': True
    }
)

# ОТПРАВКА в Exchange с routing key
channel.basic_publish(
    exchange='orders_exchange',        # в какой exchange
    routing_key='order.created.eu',    # routing key (для маршрутизации)
    body=message_body,                 # тело сообщения
    properties=properties,
    
    # mandatory=True означает: если сообщение НЕ может быть
    # доставлено ни в одну очередь → вернуть отправителю (basic.return)
    mandatory=True
)

print(f"✅ Сообщение отправлено: {message}")

# ════════════════════════════════════════════════════════════
# ПОДТВЕРЖДЕНИЕ ДОСТАВКИ (Publisher Confirms)
# ════════════════════════════════════════════════════════════

# Для надёжности можно включить Publisher Confirms:
# Брокер подтверждает КАЖДОЕ сообщение
channel.confirm_delivery()

try:
    channel.basic_publish(
        exchange='orders_exchange',
        routing_key='order.created.eu',
        body=message_body,
        properties=properties,
        mandatory=True
    )
    print("✅ Сообщение подтверждено брокером")
except pika.exceptions.UnroutableError:
    print("❌ Сообщение не может быть доставлено (нет подходящей очереди)")
except pika.exceptions.NackError:
    print("❌ Брокер отклонил сообщение (NACK)")

connection.close()
```

**Ключевые свойства сообщения:**

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                   СВОЙСТВА СООБЩЕНИЯ (Properties)                    ║
╠═══════════════════╦══════════════════════════════════════════════════╣
║   Свойство        ║   Описание                                       ║
╠═══════════════════╬══════════════════════════════════════════════════╣
║ content_type      ║ MIME-тип: application/json, text/plain, etc.     ║
╠═══════════════════╬══════════════════════════════════════════════════╣
║ delivery_mode     ║ 1 = transient (в памяти, может потеряться)       ║
║                   ║ 2 = persistent (на диске, гарантированно)        ║
╠═══════════════════╬══════════════════════════════════════════════════╣
║ priority          ║ Приоритет 0-255 (если очередь priority queue)    ║
╠═══════════════════╬══════════════════════════════════════════════════╣
║ correlation_id    ║ Для связывания запроса с ответом (RPC)           ║
╠═══════════════════╬══════════════════════════════════════════════════╣
║ reply_to          ║ Имя очереди для ответа (RPC паттерн)             ║
╠═══════════════════╬══════════════════════════════════════════════════╣
║ expiration        ║ TTL сообщения в миллисекундах (строка!)          ║
║                   ║ "60000" = удалить через 60 секунд, если не       ║
║                   ║ обработано                                        ║
╠═══════════════════╬══════════════════════════════════════════════════╣
║ message_id        ║ Уникальный ID сообщения (для идемпотентности)    ║
╠═══════════════════╬══════════════════════════════════════════════════╣
║ timestamp         ║ Время создания (Unix timestamp)                  ║
╠═══════════════════╬══════════════════════════════════════════════════╣
║ type              ║ Тип события: order.created, payment.processed    ║
╠═══════════════════╬══════════════════════════════════════════════════╣
║ user_id           ║ ID аутентифицированного пользователя             ║
║                   ║ (RabbitMQ проверяет совпадение с connection!)    ║
╠═══════════════════╬══════════════════════════════════════════════════╣
║ app_id            ║ ID приложения-отправителя                        ║
╠═══════════════════╬══════════════════════════════════════════════════╣
║ headers           ║ Произвольные key-value пары (для headers        ║
║                   ║ exchange маршрутизации)                          ║
╚═══════════════════╩══════════════════════════════════════════════════╝
```

**Publisher НЕ знает о Queue:**

Важный принцип RabbitMQ — **Publisher отправляет в Exchange, а не в Queue напрямую**. Это обеспечивает **слабую связанность** (loose coupling):

text

```
Publisher знает:
  ✅ Exchange name ("orders_exchange")
  ✅ Routing key ("order.created.eu")
  
Publisher НЕ знает:
  ❌ Имена очередей
  ❌ Сколько очередей получит сообщение (может быть 0, 1 или 10)
  ❌ Кто подписан на эти очереди
  
Это позволяет ИЗМЕНЯТЬ топологию (добавлять/удалять очереди)
БЕЗ изменения кода Publisher'а!
```

---

### Consumer (Потребитель / Подписчик)

**Consumer** — это приложение, которое **получает и обрабатывает** сообщения из очередей RabbitMQ.

**Подробное описание:**

Consumer подписывается на одну или несколько **очередей** (не на Exchange!) и получает сообщения для обработки. В отличие от Kafka (pull-модель), RabbitMQ использует **push-модель** — брокер **активно отправляет** сообщения Consumer'у, как только они появляются в очереди.

text

```
┌────────────────────────────────────────────────────────────────────┐
│                         CONSUMER                                   │
│                                                                    │
│  [Queue: orders_eu] ────▶ Consumer получает сообщение              │
│                                                                    │
│  Consumer обрабатывает:                                            │
│  1. Получил сообщение (basic.deliver)                              │
│  2. Декодировал JSON                                               │
│  3. Выполнил бизнес-логику (сохранил в БД, отправил email, etc.)   │
│  4. Отправил ПОДТВЕРЖДЕНИЕ брокеру (ACK)                           │
│                                                                    │
│  Брокер ◀──── ACK ──────── Consumer                                │
│                                                                    │
│  После ACK:                                                        │
│  • RabbitMQ УДАЛЯЕТ сообщение из очереди ✅                        │
│  • Сообщение считается успешно обработанным                        │
│                                                                    │
│  Если Consumer упал ДО отправки ACK:                               │
│  • RabbitMQ ПЕРЕОТПРАВИТ сообщение другому Consumer'у              │
│  • Сообщение НЕ потеряно! ✅                                       │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Анатомия Consumer:**

Python

```
import pika
import json
import time

# ════════════════════════════════════════════════════════════
# ПОДКЛЮЧЕНИЕ К RABBITMQ
# ════════════════════════════════════════════════════════════

credentials = pika.PlainCredentials('username', 'password')
parameters = pika.ConnectionParameters(
    host='rabbitmq-server.example.com',
    port=5672,
    virtual_host='/',
    credentials=credentials,
    heartbeat=60
)

connection = pika.BlockingConnection(parameters)
channel = connection.channel()

# ════════════════════════════════════════════════════════════
# ОБЪЯВЛЕНИЕ ОЧЕРЕДИ (если ещё не существует)
# ════════════════════════════════════════════════════════════

queue_name = 'orders_eu_queue'

channel.queue_declare(
    queue=queue_name,
    durable=True,              # очередь переживёт перезагрузку брокера
    exclusive=False,           # НЕ эксклюзивная (могут подключаться несколько consumers)
    auto_delete=False,         # НЕ удалять автоматически при отключении последнего consumer
    arguments={
        # Максимальная длина очереди (старые сообщения будут отклоняться)
        'x-max-length': 10000,
        
        # Dead Letter Exchange (куда отправлять отклонённые сообщения)
        'x-dead-letter-exchange': 'dlx_exchange',
        'x-dead-letter-routing-key': 'failed.orders',
        
        # TTL для сообщений в очереди (миллисекунды)
        'x-message-ttl': 3600000,  # 1 час
        
        # Приоритет очереди (0-255, если нужен priority queue)
        'x-max-priority': 10,
        
        # Queue mode
        'x-queue-mode': 'lazy'  # lazy queue (меньше RAM, больше disk I/O)
    }
)

# ════════════════════════════════════════════════════════════
# QoS (QUALITY OF SERVICE) — контроль потока
# ════════════════════════════════════════════════════════════

# Prefetch count — сколько неподтверждённых сообщений может
# одновременно находиться у Consumer'а
channel.basic_qos(
    prefetch_count=10  # обрабатывать максимум 10 сообщений параллельно
)

# Это защищает от перегрузки Consumer'а.
# Если Consumer медленный — RabbitMQ не будет отправлять ему
# больше 10 неподтверждённых сообщений.

# ════════════════════════════════════════════════════════════
# CALLBACK ФУНКЦИЯ — обработка сообщения
# ════════════════════════════════════════════════════════════

def on_message_callback(ch, method, properties, body):
    """
    Вызывается каждый раз, когда приходит новое сообщение.
    
    Args:
        ch: channel
        method: метаданные доставки (delivery_tag, routing_key, exchange)
        properties: свойства сообщения (content_type, headers, etc.)
        body: тело сообщения (bytes)
    """
    
    print(f"\n{'='*60}")
    print(f"📨 Получено сообщение:")
    print(f"   Exchange: {method.exchange}")
    print(f"   Routing Key: {method.routing_key}")
    print(f"   Delivery Tag: {method.delivery_tag}")
    print(f"   Content Type: {properties.content_type}")
    print(f"   Priority: {properties.priority}")
    print(f"   Message ID: {properties.message_id}")
    
    try:
        # Декодировать JSON
        message = json.loads(body)
        print(f"   Body: {message}")
        
        # ═══════════════════════════════════════════════════════
        # БИЗНЕС-ЛОГИКА ОБРАБОТКИ
        # ═══════════════════════════════════════════════════════
        
        order_id = message['orderId']
        amount = message['amount']
        
        # Пример: сохранить в БД
        # db.save_order(order_id, amount)
        
        # Пример: отправить email
        # send_email(customer_email, order_details)
        
        # Симуляция обработки
        print(f"   🔄 Обрабатываю заказ #{order_id} на сумму ${amount}...")
        time.sleep(1)  # имитация работы
        
        print(f"   ✅ Заказ #{order_id} успешно обработан!")
        
        # ═══════════════════════════════════════════════════════
        # ПОДТВЕРЖДЕНИЕ (ACK)
        # ═══════════════════════════════════════════════════════
        
        # ВАЖНО: отправить ACK только после УСПЕШНОЙ обработки!
        ch.basic_ack(delivery_tag=method.delivery_tag)
        
        print(f"   📤 Отправлен ACK для delivery_tag={method.delivery_tag}")
        
    except json.JSONDecodeError as e:
        print(f"   ❌ Ошибка декодирования JSON: {e}")
        
        # NACK (negative acknowledgment) — отклонить сообщение
        # requeue=False → отправить в Dead Letter Exchange (если настроен)
        # requeue=True  → вернуть в очередь для повторной попытки
        ch.basic_nack(delivery_tag=method.delivery_tag, requeue=False)
        
    except Exception as e:
        print(f"   ❌ Ошибка обработки: {e}")
        
        # REJECT — отклонить одно сообщение
        # Аналогично NACK, но без параметра multiple
        ch.basic_reject(delivery_tag=method.delivery_tag, requeue=False)
        
    print(f"{'='*60}\n")

# ════════════════════════════════════════════════════════════
# ПОДПИСКА НА ОЧЕРЕДЬ (начать получать сообщения)
# ════════════════════════════════════════════════════════════

channel.basic_consume(
    queue=queue_name,
    on_message_callback=on_message_callback,
    
    # auto_ack=False — РУЧНОЕ подтверждение (рекомендуется!)
    # auto_ack=True  — автоматическое подтверждение (рискованно!)
    auto_ack=False,
    
    # exclusive=True — только ЭТОТ consumer может читать из очереди
    exclusive=False,
    
    # consumer_tag — уникальный ID этого consumer (для управления)
    consumer_tag='order-processor-1'
)

print(f"🎧 Consumer запущен. Ожидание сообщений из '{queue_name}'...")
print("   Для остановки нажмите CTRL+C\n")

# ════════════════════════════════════════════════════════════
# ЗАПУСК ЦИКЛА ОБРАБОТКИ
# ════════════════════════════════════════════════════════════

try:
    # Бесконечный цикл — ожидание и обработка сообщений
    channel.start_consuming()
    
except KeyboardInterrupt:
    print("\n⏹️  Остановка Consumer...")
    channel.stop_consuming()
    
finally:
    connection.close()
    print("✅ Соединение закрыто")
```

**Режимы подтверждения (Acknowledgment):**

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                   РЕЖИМЫ ПОДТВЕРЖДЕНИЯ                               ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  1. AUTO-ACK (автоматическое подтверждение)                          ║
║     basic_consume(auto_ack=True)                                     ║
║                                                                      ║
║     • RabbitMQ УДАЛЯЕТ сообщение СРАЗУ после отправки Consumer'у     ║
║     • Не ждёт подтверждения обработки                                ║
║                                                                      ║
║     ✅ Быстро (нет network round-trip для ACK)                       ║
║     ❌ ОПАСНО: если Consumer упал — сообщение ПОТЕРЯНО навсегда!     ║
║                                                                      ║
║     📌 Применение: некритичные данные (метрики, логи)                ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  2. MANUAL ACK (ручное подтверждение) — РЕКОМЕНДУЕТСЯ!               ║
║     basic_consume(auto_ack=False)                                    ║
║                                                                      ║
║     Consumer получает сообщение → обрабатывает → отправляет ACK      ║
║                                                                      ║
║     • ch.basic_ack(delivery_tag)    — успешно обработано ✅          ║
║     • ch.basic_nack(delivery_tag, requeue=True/False)  — ошибка      ║
║     • ch.basic_reject(delivery_tag, requeue=True/False) — отклонить  ║
║                                                                      ║
║     ✅ Надёжно: сообщение удаляется ТОЛЬКО после обработки           ║
║     ✅ Если Consumer упал → RabbitMQ переотправит другому Consumer'у ║
║     ❌ Чуть медленнее (дополнительный network round-trip)            ║
║                                                                      ║
║     📌 Применение: критичные данные (заказы, платежи, транзакции)    ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  3. NACK vs REJECT                                                   ║
║                                                                      ║
║     NACK (Negative Acknowledgment):                                  ║
║       ch.basic_nack(                                                 ║
║           delivery_tag=tag,                                          ║
║           multiple=False,      # False = только это сообщение        ║
║                                # True = это + все предыдущие         ║
║           requeue=False        # False = в DLX / удалить            ║
║                                # True = вернуть в очередь            ║
║       )                                                              ║
║                                                                      ║
║     REJECT (более старый способ, аналог NACK для одного сообщения):  ║
║       ch.basic_reject(                                               ║
║           delivery_tag=tag,                                          ║
║           requeue=False                                              ║
║       )                                                              ║
║                                                                      ║
║     requeue=True — ОПАСНО: может создать бесконечный цикл!           ║
║     Если сообщение битое → Consumer будет получать его снова и снова ║
║                                                                      ║
║     РЕКОМЕНДАЦИЯ: requeue=False + Dead Letter Exchange               ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Prefetch Count (QoS) — детальное объяснение:**

text

```
БЕЗ PREFETCH (по умолчанию):

RabbitMQ отправляет ВСЕ сообщения из очереди Consumer'у сразу.
Если в очереди 10,000 сообщений → все 10,000 уйдут к Consumer'у.

Проблема:
  • Consumer перегружен (OutOfMemory)
  • Другие Consumer'ы ПРОСТАИВАЮТ (нет сообщений)
  • Если Consumer упал → 10,000 сообщений переотправятся → снова перегрузка

┌──────────────────────────────────────────────────────────────┐
│  Queue: [msg1, msg2, msg3, ..., msg10000]                    │
│                                                              │
│         ▼ ВСЕ 10,000 сразу!                                  │
│  Consumer A (перегружен) 💥                                  │
│  Consumer B (простаивает) 😴                                  │
│  Consumer C (простаивает) 😴                                  │
└──────────────────────────────────────────────────────────────┘


С PREFETCH (channel.basic_qos(prefetch_count=10)):

RabbitMQ отправляет максимум 10 неподтверждённых сообщений каждому Consumer'у.

┌──────────────────────────────────────────────────────────────┐
│  Queue: [msg1, msg2, msg3, ..., msg10000]                    │
│                                                              │
│  Consumer A: получил msg1-10   (обрабатывает)               │
│  Consumer B: получил msg11-20  (обрабатывает)               │
│  Consumer C: получил msg21-30  (обрабатывает)               │
│                                                              │
│  Как только Consumer A отправит ACK для msg1 →               │
│  RabbitMQ отправит ему msg31                                 │
│                                                              │
│  ✅ Равномерная нагрузка между Consumer'ами                  │
│  ✅ Защита от перегрузки                                      │
└──────────────────────────────────────────────────────────────┘

РЕКОМЕНДАЦИЯ: prefetch_count = 1-50
  • Быстрые задачи (< 1 сек) → prefetch_count = 20-50
  • Медленные задачи (> 10 сек) → prefetch_count = 1-5
```

**Competing Consumers (конкурирующие потребители):**

Несколько Consumer'ов подписываются на **одну и ту же очередь**. RabbitMQ распределяет сообщения между ними методом **round-robin** (по очереди).

text

```
┌────────────────────────────────────────────────────────────────┐
│  Queue: [msg1, msg2, msg3, msg4, msg5, msg6]                   │
│                                                                │
│           ┌──▶ Consumer A: msg1, msg4  (обрабатывает)          │
│           │                                                    │
│  Round-   ├──▶ Consumer B: msg2, msg5  (обрабатывает)          │
│  Robin    │                                                    │
│           └──▶ Consumer C: msg3, msg6  (обрабатывает)          │
│                                                                │
│  • Каждое сообщение → ОДНОМУ Consumer'у                        │
│  • Параллельная обработка ✅                                   │
│  • Горизонтальное масштабирование ✅                            │
│                                                                │
└────────────────────────────────────────────────────────────────┘

Добавить больше Consumer'ов = увеличить throughput!
```

---

### Exchange (Биржа обмена)

**Exchange** — это **маршрутизатор сообщений** в RabbitMQ. Это центральный компонент, который **принимает сообщения от Publisher'ов** и **решает, в какие очереди** их отправить на основе правил маршрутизации.

**Ключевая идея:**

Publisher НЕ отправляет сообщения в Queue напрямую. Он отправляет в **Exchange**, а Exchange уже направляет сообщения в **одну или несколько Queue** на основе:

- **Типа Exchange** (direct, topic, fanout, headers)
- **Routing key** (ключа маршрутизации)
- **Bindings** (связей между Exchange и Queue)

text

```
┌────────────────────────────────────────────────────────────────────┐
│                          EXCHANGE                                  │
│                     (центр маршрутизации)                          │
│                                                                    │
│  Publisher ──msg (routing_key="order.created.eu")──▶ EXCHANGE      │
│                                                         │          │
│                                   Exchange анализирует:           │
│                                   • routing key                    │
│                                   • bindings                       │
│                                   • тип exchange                   │
│                                                         │          │
│                         Решение: отправить в Queue A   │          │
│                                                         ▼          │
│                                                    [Queue A]       │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Типы Exchange — подробное описание:**

#### 1. Direct Exchange — точная маршрутизация

Сообщение отправляется в Queue, у которой **binding key точно совпадает** с **routing key** сообщения.

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                       DIRECT EXCHANGE                                ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  Принцип: routing_key == binding_key → доставка                      ║
║                                                                      ║
║  ┌────────────────────────────────────────────────────────────────┐  ║
║  │                 Direct Exchange "logs"                         │  ║
║  │                                                                │  ║
║  │  Bindings:                                                     │  ║
║  │  • "error"   → [Error Queue]                                   │  ║
║  │  • "warning" → [Warning Queue]                                 │  ║
║  │  • "info"    → [Info Queue]                                    │  ║
║  └────────────────────────────────────────────────────────────────┘  ║
║                                                                      ║
║  Publisher отправляет:                                               ║
║  ───────────────────────────────────────────────────────────         ║
║  • routing_key="error"   → [Error Queue]   ✅                        ║
║  • routing_key="warning" → [Warning Queue] ✅                        ║
║  • routing_key="debug"   → ❌ НЕ доставлено (нет binding)            ║
║                                                                      ║
║  📌 Применение:                                                      ║
║     • Логирование по уровням                                         ║
║     • Task queue (каждый тип задачи → своя очередь)                 ║
║     • Simple routing                                                 ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Код:**

Python

```
# Объявление Direct Exchange
channel.exchange_declare(
    exchange='logs_direct',
    exchange_type='direct',
    durable=True
)

# Создание очередей
channel.queue_declare(queue='error_logs', durable=True)
channel.queue_declare(queue='warning_logs', durable=True)
channel.queue_declare(queue='info_logs', durable=True)

# Bindings — связываем Exchange с Queue
channel.queue_bind(
    exchange='logs_direct',
    queue='error_logs',
    routing_key='error'
)

channel.queue_bind(
    exchange='logs_direct',
    queue='warning_logs',
    routing_key='warning'
)

channel.queue_bind(
    exchange='logs_direct',
    queue='info_logs',
    routing_key='info'
)

# Publisher отправляет
channel.basic_publish(
    exchange='logs_direct',
    routing_key='error',      # → error_logs
    body='Database connection failed!'
)

channel.basic_publish(
    exchange='logs_direct',
    routing_key='info',       # → info_logs
    body='User logged in successfully'
)
```

#### 2. Topic Exchange — маршрутизация по паттерну

Сообщение отправляется в Queue, у которой **binding key соответствует паттерну** routing key. Используются **wildcards**:

- `*` (звёздочка) — заменяет **ровно одно** слово
- `#` (решётка) — заменяет **ноль или более** слов

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                       TOPIC EXCHANGE                                 ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  Routing key состоит из слов, разделённых точками:                  ║
║  "region.country.city" или "order.created.premium"                   ║
║                                                                      ║
║  ┌────────────────────────────────────────────────────────────────┐  ║
║  │                 Topic Exchange "events"                        │  ║
║  │                                                                │  ║
║  │  Bindings:                                                     │  ║
║  │  • "order.*.eu"       → [EU Orders Queue]                      │  ║
║  │  • "order.created.#"  → [All Created Orders Queue]             │  ║
║  │  • "#.premium"        → [Premium Events Queue]                 │  ║
║  │  • "*.deleted.*"      → [Deleted Events Queue]                 │  ║
║  └────────────────────────────────────────────────────────────────┘  ║
║                                                                      ║
║  Примеры маршрутизации:                                              ║
║  ─────────────────────────────────────────────────────────           ║
║                                                                      ║
║  routing_key: "order.created.eu"                                     ║
║  ✅ "order.*.eu"       → [EU Orders Queue]                           ║
║  ✅ "order.created.#"  → [All Created Orders Queue]                  ║
║  ❌ "#.premium"        (не заканчивается на premium)                 ║
║  ❌ "*.deleted.*"      (не deleted)                                  ║
║                                                                      ║
║  routing_key: "payment.updated.premium"                              ║
║  ❌ "order.*.eu"       (не order)                                    ║
║  ❌ "order.created.#"  (не order)                                    ║
║  ✅ "#.premium"        → [Premium Events Queue]                      ║
║  ❌ "*.deleted.*"      (не deleted)                                  ║
║                                                                      ║
║  routing_key: "user.deleted.eu"                                      ║
║  ❌ "order.*.eu"       (не order)                                    ║
║  ❌ "order.created.#"  (не order)                                    ║
║  ❌ "#.premium"        (не premium)                                  ║
║  ✅ "*.deleted.*"      → [Deleted Events Queue]                      ║
║                                                                      ║
║  📌 Применение:                                                      ║
║     • Event-driven архитектура (микросервисы)                        ║
║     • Логирование с фильтрацией (app.*.error, *.critical.*)         ║
║     • Мультирегиональные системы (eu.*.*, us.*.*)                   ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Код:**

Python

```
# Объявление Topic Exchange
channel.exchange_declare(
    exchange='events_topic',
    exchange_type='topic',
    durable=True
)

# Создание очередей
channel.queue_declare(queue='eu_orders', durable=True)
channel.queue_declare(queue='all_created_orders', durable=True)
channel.queue_declare(queue='premium_events', durable=True)

# Bindings с паттернами
channel.queue_bind(
    exchange='events_topic',
    queue='eu_orders',
    routing_key='order.*.eu'     # order.<что угодно>.eu
)

channel.queue_bind(
    exchange='events_topic',
    queue='all_created_orders',
    routing_key='order.created.#'  # order.created.<что угодно>
)

channel.queue_bind(
    exchange='events_topic',
    queue='premium_events',
    routing_key='#.premium'        # <что угодно>.premium
)

# Publisher отправляет
channel.basic_publish(
    exchange='events_topic',
    routing_key='order.created.eu',
    body='{"orderId": 123}'
)
# → попадёт в "eu_orders" И "all_created_orders" (обе!)

channel.basic_publish(
    exchange='events_topic',
    routing_key='payment.processed.premium',
    body='{"amount": 999.99}'
)
# → попадёт в "premium_events"
```

#### 3. Fanout Exchange — broadcast всем

Сообщение отправляется **во ВСЕ привязанные очереди**. Routing key **игнорируется**.

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                       FANOUT EXCHANGE                                ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  Принцип: broadcast — каждая привязанная Queue получает копию        ║
║                                                                      ║
║  ┌────────────────────────────────────────────────────────────────┐  ║
║  │                 Fanout Exchange "notifications"                │  ║
║  │                                                                │  ║
║  │  Bindings:                                                     │  ║
║  │  • [Email Queue]                                               │  ║
║  │  • [SMS Queue]                                                 │  ║
║  │  • [Push Notifications Queue]                                  │  ║
║  │  • [Audit Log Queue]                                           │  ║
║  └────────────────────────────────────────────────────────────────┘  ║
║                                                                      ║
║  Publisher отправляет:                                               ║
║  ───────────────────────────────────────────────────────────         ║
║  routing_key="ANY" (игнорируется)                                    ║
║                                                                      ║
║  Сообщение попадает:                                                 ║
║  ✅ [Email Queue]                                                    ║
║  ✅ [SMS Queue]                                                      ║
║  ✅ [Push Notifications Queue]                                       ║
║  ✅ [Audit Log Queue]                                                ║
║                                                                      ║
║  📌 Применение:                                                      ║
║     • Рассылка уведомлений всем каналам                              ║
║     • Репликация данных                                              ║
║     • Broadcasting событий                                           ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Код:**

Python

```
# Объявление Fanout Exchange
channel.exchange_declare(
    exchange='notifications_fanout',
    exchange_type='fanout',
    durable=True
)

# Создание очередей
channel.queue_declare(queue='email_queue', durable=True)
channel.queue_declare(queue='sms_queue', durable=True)
channel.queue_declare(queue='push_queue', durable=True)

# Bindings (routing_key не важен для fanout)
channel.queue_bind(exchange='notifications_fanout', queue='email_queue')
channel.queue_bind(exchange='notifications_fanout', queue='sms_queue')
channel.queue_bind(exchange='notifications_fanout', queue='push_queue')

# Publisher отправляет
channel.basic_publish(
    exchange='notifications_fanout',
    routing_key='',  # игнорируется
    body='User registered: user@example.com'
)
# → попадёт во ВСЕ 3 очереди одновременно
```

#### 4. Headers Exchange — маршрутизация по заголовкам

Маршрутизация основывается на **заголовках сообщения** (headers), а не на routing key. Более гибкая, но редко используется (медленнее, чем topic).

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                       HEADERS EXCHANGE                               ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  Binding указывает: какие заголовки должны совпадать                 ║
║                                                                      ║
║  ┌────────────────────────────────────────────────────────────────┐  ║
║  │  Headers Exchange "tasks"                                      │  ║
║  │                                                                │  ║
║  │  Bindings:                                                     │  ║
║  │  [High Priority Queue]                                         │  ║
║  │    x-match: all                                                │  ║
║  │    priority: high                                              │  ║
║  │    region: eu                                                  │  ║
║  │                                                                │  ║
║  │  [Image Processing Queue]                                      │  ║
║  │    x-match: any                                                │  ║
║  │    type: image                                                 │  ║
║  │    type: video                                                 │  ║
║  └────────────────────────────────────────────────────────────────┘  ║
║                                                                      ║
║  x-match:                                                            ║
║  • all — ВСЕ заголовки должны совпадать (AND)                       ║
║  • any — ХОТЯ БЫ ОДИН заголовок должен совпадать (OR)              ║
║                                                                      ║
║  Publisher отправляет с headers:                                     ║
║  ─────────────────────────────────────────                           ║
║  {priority: "high", region: "eu", type: "order"}                     ║
║  ✅ → [High Priority Queue] (priority=high AND region=eu)            ║
║                                                                      ║
║  {type: "image", size: "large"}                                      ║
║  ✅ → [Image Processing Queue] (type=image OR type=video)            ║
║                                                                      ║
║  📌 Применение:                                                      ║
║     • Сложная маршрутизация (когда topic недостаточно)              ║
║     • Фильтрация по множеству критериев                             ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Код:**

Python

```
# Объявление Headers Exchange
channel.exchange_declare(
    exchange='tasks_headers',
    exchange_type='headers',
    durable=True
)

# Создание очередей
channel.queue_declare(queue='high_priority_eu', durable=True)

# Binding с условиями на заголовки
channel.queue_bind(
    exchange='tasks_headers',
    queue='high_priority_eu',
    arguments={
        'x-match': 'all',      # ВСЕ заголовки должны совпадать
        'priority': 'high',
        'region': 'eu'
    }
)

# Publisher отправляет с заголовками
channel.basic_publish(
    exchange='tasks_headers',
    routing_key='',  # игнорируется в headers exchange
    body='Process urgent EU order',
    properties=pika.BasicProperties(
        headers={
            'priority': 'high',
            'region': 'eu',
            'customer_type': 'premium'
        }
    )
)
# → попадёт в 'high_priority_eu' (priority=high AND region=eu совпали)
```

**Default Exchange (безымянный):**

RabbitMQ имеет **встроенный Exchange** с пустым именем `""`, который работает как **Direct Exchange** с особенностью: routing key == имя очереди.

Python

```
# Отправка в Default Exchange
# routing_key совпадает с именем очереди
channel.basic_publish(
    exchange='',  # пустая строка = default exchange
    routing_key='my_queue_name',
    body='Hello'
)

# Это автоматически доставит в очередь "my_queue_name"
# БЕЗ явного binding!
```

---

### Queue (Очередь)

**Queue** — это **буфер хранения сообщений**, куда Exchange направляет сообщения, и откуда Consumer'ы их забирают. Очередь — это единственное место, где сообщения **физически хранятся** в RabbitMQ.

**Подробное описание:**

Queue — это **FIFO структура данных** (First In, First Out), но RabbitMQ поддерживает и **Priority Queue** (сообщения с высоким приоритетом обрабатываются первыми).

text

```
┌────────────────────────────────────────────────────────────────────┐
│                            QUEUE                                   │
│                   (физическое хранилище сообщений)                  │
│                                                                    │
│  Exchange ──▶ [ msg5 | msg4 | msg3 | msg2 | msg1 ] ──▶ Consumer   │
│               ◀──────────── FIFO ───────────────▶                  │
│                вход                         выход                 │
│                                                                    │
│  Сообщения хранятся:                                               │
│  • В ПАМЯТИ (быстро, НЕ переживёт перезагрузку)                    │
│  • НА ДИСКЕ (медленнее, переживёт перезагрузку)                    │
│                                                                    │
│  Когда Consumer отправляет ACK → сообщение УДАЛЯЕТСЯ              │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Свойства очереди:**

Python

```
channel.queue_declare(
    queue='my_queue',
    
    # ════════════════════════════════════════════════════════════
    # DURABLE — переживёт перезагрузку брокера
    # ════════════════════════════════════════════════════════════
    # True  = очередь сохраняется на диск (метаданные, не сообщения!)
    # False = очередь исчезнет при перезагрузке
    durable=True,
    
    # ════════════════════════════════════════════════════════════
    # EXCLUSIVE — эксклюзивная очередь
    # ════════════════════════════════════════════════════════════
    # True  = только ЭТОТ connection может использовать очередь
    #         автоматически удалится при закрытии connection
    # False = любой connection может использовать
    exclusive=False,
    
    # ════════════════════════════════════════════════════════════
    # AUTO_DELETE — автоматическое удаление
    # ════════════════════════════════════════════════════════════
    # True  = очередь удалится, когда последний consumer отключится
    # False = очередь остаётся даже без consumers
    auto_delete=False,
    
    # ════════════════════════════════════════════════════════════
    # ARGUMENTS — расширенные настройки
    # ════════════════════════════════════════════════════════════
    arguments={
        # Максимальная длина очереди (количество сообщений)
        # Что делать при превышении:
        # • drop-head (удалить старое сообщение) — default
        # • reject-publish (отклонить новое сообщение)
        'x-max-length': 10000,
        
        # Максимальный размер очереди (байты)
        'x-max-length-bytes': 1048576000,  # 1 GB
        
        # TTL сообщений в очереди (миллисекунды)
        # Сообщения старше этого времени удаляются
        'x-message-ttl': 3600000,  # 1 час
        
        # TTL самой очереди (автоудаление неиспользуемой очереди)
        'x-expires': 1800000,  # удалить очередь через 30 мин без использования
        
        # Dead Letter Exchange — куда отправлять "мёртвые" сообщения
        'x-dead-letter-exchange': 'dlx',
        'x-dead-letter-routing-key': 'failed.tasks',
        
        # Приоритет сообщений (0-255)
        # Очередь становится Priority Queue
        'x-max-priority': 10,
        
        # Queue Mode:
        # • default — хранить в RAM (быстро)
        # • lazy    — хранить на диске (меньше RAM, больше I/O)
        'x-queue-mode': 'lazy',
        
        # Overflow behaviour при x-max-length:
        # • drop-head          — удалить СТАРОЕ сообщение (FIFO)
        # • reject-publish     — отклонить НОВОЕ сообщение
        # • reject-publish-dlx — отклонить + отправить в DLX
        'x-overflow': 'drop-head'
    }
)
```

**Типы очередей:**

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║  1. CLASSIC QUEUE (классическая, по умолчанию)                       ║
║     ═══════════════════════════════════════                          ║
║                                                                      ║
║     • Сообщения в RAM (или на диске, если durable)                   ║
║     • Одна master нода (не распределённая)                           ║
║     • Репликация через mirroring (устаревший способ)                 ║
║                                                                      ║
║     arguments={'x-queue-type': 'classic'}                            ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  2. QUORUM QUEUE (кворумная, рекомендуется для production!)          ║
║     ═══════════════════════════════════════════════════              ║
║                                                                      ║
║     • Распределённая очередь (Raft consensus)                        ║
║     • Данные РЕПЛИЦИРУЮТСЯ на несколько нод                          ║
║     • Переживает падение нод (высокая доступность)                   ║
║     • Гарантия: at-least-once (может быть дубликат)                  ║
║     • Чуть медленнее, но НАДЁЖНЕЕ                                    ║
║                                                                      ║
║     arguments={'x-queue-type': 'quorum'}                             ║
║                                                                      ║
║     📌 Применение: критичные данные (заказы, платежи)                ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  3. STREAM QUEUE (потоковая, RabbitMQ 3.9+)                          ║
║     ═══════════════════════════════════════════                      ║
║                                                                      ║
║     • Append-only лог (как Kafka!)                                   ║
║     • Сообщения НЕ удаляются после чтения                            ║
║     • Можно ПЕРЕЧИТЫВАТЬ (replay)                                    ║
║     • Высокий throughput (миллионы msg/sec)                          ║
║     • Consumer управляет offset'ом                                   ║
║                                                                      ║
║     arguments={'x-queue-type': 'stream'}                             ║
║                                                                      ║
║     📌 Применение: event sourcing, логи, аналитика                   ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Quorum Queue — подробно:**

Python

```
# Создание Quorum Queue (рекомендуется для production!)
channel.queue_declare(
    queue='orders_quorum',
    durable=True,  # ОБЯЗАТЕЛЬНО для quorum
    arguments={
        'x-queue-type': 'quorum',  # тип очереди
        
        # Quorum-специфичные настройки:
        'x-quorum-initial-group-size': 3,  # размер кворума (обычно 3 или 5)
        'x-max-in-memory-length': 0,       # 0 = хранить всё на диске (надёжнее)
        'x-max-in-memory-bytes': 0
    }
)

# Quorum Queue автоматически:
# • Реплицируется на N нод (N = x-quorum-initial-group-size)
# • Использует Raft для консенсуса
# • Переживает падение (N-1)/2 нод
#   (для 3 нод → переживёт падение 1 ноды)
#   (для 5 нод → переживёт падение 2 нод)
```

**Dead Letter Exchange (DLX) — обработка неудавшихся сообщений:**

DLX — это механизм для **перенаправления "мёртвых" сообщений** в специальную очередь для анализа или повторной обработки.

Сообщение становится "мёртвым" если:

- Consumer отклонил его (NACK / REJECT с requeue=false)
- Истёк TTL сообщения
- Превышена длина очереди (overflow)

Python

```
# ════════════════════════════════════════════════════════════
# Настройка DLX
# ════════════════════════════════════════════════════════════

# 1. Создать DLX Exchange
channel.exchange_declare(
    exchange='dlx_exchange',
    exchange_type='direct',
    durable=True
)

# 2. Создать Dead Letter Queue
channel.queue_declare(queue='failed_messages', durable=True)

# 3. Bind DLX Exchange → Dead Letter Queue
channel.queue_bind(
    exchange='dlx_exchange',
    queue='failed_messages',
    routing_key='failed'
)

# 4. Создать ОСНОВНУЮ очередь с DLX
channel.queue_declare(
    queue='main_queue',
    durable=True,
    arguments={
        'x-dead-letter-exchange': 'dlx_exchange',      # куда отправлять "мёртвые"
        'x-dead-letter-routing-key': 'failed',         # с каким routing key
        'x-message-ttl': 60000  # сообщения "умирают" через 60 сек
    }
)

# Теперь:
# • Сообщение в main_queue не обработано за 60 сек → DLX → failed_messages
# • Consumer отклонил сообщение (NACK) → DLX → failed_messages
```

**Мониторинг очередей:**

Bash

```
# CLI команды для управления очередями

# Список всех очередей
rabbitmqctl list_queues

# Детальная информация
rabbitmqctl list_queues name messages consumers memory

# Вывод:
# Listing queues for vhost / ...
# name              messages  consumers  memory
# orders_queue      1523      3          45632000
# payments_queue    0         1          8192000
# notifications     245       0          12800000

# messages   — количество сообщений в очереди
# consumers  — количество подключённых consumers
# memory     — использование памяти (байты)

# Очистить очередь (удалить все сообщения)
rabbitmqctl purge_queue orders_queue

# Удалить очередь
rabbitmqctl delete_queue orders_queue
```

---

### Binding (Привязка)

**Binding** — это **правило маршрутизации**, которое связывает **Exchange** с **Queue**. Binding определяет: при каких условиях сообщение из Exchange попадёт в Queue.

**Подробное описание:**

Binding — это **"мост"** между Exchange и Queue. Он содержит:

- **Binding key** (ключ привязки) — паттерн или точное значение
- **Arguments** (опционально) — дополнительные параметры для headers exchange

text

```
┌────────────────────────────────────────────────────────────────────┐
│                           BINDING                                  │
│                    (связь Exchange ↔ Queue)                        │
│                                                                    │
│  ┌──────────────┐       Binding        ┌──────────────┐            │
│  │              │  (routing_key="eu")  │              │            │
│  │  Exchange    │─────────────────────▶│    Queue     │            │
│  │  "orders"    │                      │  "orders_eu" │            │
│  └──────────────┘                      └──────────────┘            │
│                                                                    │
│  Если сообщение приходит с routing_key="eu" →                      │
│  Exchange направляет в Queue "orders_eu" ✅                        │
│                                                                    │
│  Если сообщение приходит с routing_key="us" →                      │
│  НЕ попадает в Queue "orders_eu" ❌                                │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Создание Binding:**

Python

```
# ════════════════════════════════════════════════════════════
# Привязка Queue к Exchange
# ════════════════════════════════════════════════════════════

channel.queue_bind(
    exchange='orders_exchange',  # из какого Exchange
    queue='orders_eu_queue',     # в какую Queue
    routing_key='order.*.eu'     # при каком routing_key (паттерн для topic)
)

# Одну очередь можно привязать к одному Exchange
# НЕСКОЛЬКО РАЗ с разными routing keys!

channel.queue_bind(
    exchange='orders_exchange',
    queue='high_priority_queue',
    routing_key='order.*.premium'
)

channel.queue_bind(
    exchange='orders_exchange',
    queue='high_priority_queue',
    routing_key='order.*.urgent'
)

# Теперь high_priority_queue получит сообщения:
# • routing_key="order.created.premium" ✅
# • routing_key="order.updated.urgent"  ✅
# • routing_key="order.created.eu"      ❌
```

**Binding с arguments (для Headers Exchange):**

Python

```
channel.queue_bind(
    exchange='tasks_headers',
    queue='critical_tasks',
    arguments={
        'x-match': 'all',
        'priority': 'critical',
        'region': 'eu'
    }
)

# Сообщение попадёт в critical_tasks ТОЛЬКО если
# headers содержат: priority=critical AND region=eu
```

**Unbinding (отвязка):**

Python

```
# Удалить binding
channel.queue_unbind(
    exchange='orders_exchange',
    queue='orders_eu_queue',
    routing_key='order.*.eu'
)

# После этого сообщения с routing_key="order.*.eu"
# НЕ будут попадать в orders_eu_queue
```

**Exchange-to-Exchange Binding:**

RabbitMQ поддерживает привязку **Exchange к Exchange** (федерация). Это создаёт **цепочки маршрутизации**.

Python

```
# Создать два Exchange
channel.exchange_declare(exchange='source_exchange', exchange_type='topic')
channel.exchange_declare(exchange='destination_exchange', exchange_type='direct')

# Привязать Exchange к Exchange
channel.exchange_bind(
    source='source_exchange',
    destination='destination_exchange',
    routing_key='order.#'
)

# Теперь сообщения из source_exchange (с routing_key order.#)
# будут ПЕРЕНАПРАВЛЕНЫ в destination_exchange
```

---

## 2. Отказоустойчивость кластера RabbitMQ

### Зачем нужен кластер?

Одиночный RabbitMQ сервер — это **single point of failure**. Если сервер упадёт:

- ❌ Все очереди недоступны
- ❌ Все сообщения (если в памяти) потеряны
- ❌ Producer'ы и Consumer'ы не могут подключиться

**Кластер RabbitMQ** обеспечивает:

- ✅ **Высокую доступность** (High Availability) — очереди реплицируются
- ✅ **Горизонтальное масштабирование** — больше нод = больше throughput
- ✅ **Отказоустойчивость** — кластер переживает падение нод

### Архитектура кластера

text

```
┌────────────────────────────────────────────────────────────────────┐
│                   RABBITMQ CLUSTER                                 │
│                                                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │   Node 1     │  │   Node 2     │  │   Node 3     │             │
│  │  rabbit@srv1 │  │  rabbit@srv2 │  │  rabbit@srv3 │             │
│  │              │  │              │  │              │             │
│  │  Metadata    │  │  Metadata    │  │  Metadata    │             │
│  │  (replicated)│  │  (replicated)│  │  (replicated)│             │
│  │              │  │              │  │              │             │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘             │
│         │                 │                 │                     │
│         └────── Cluster Communication ──────┘                     │
│                   (Erlang distribution)                            │
│                                                                    │
│  • Все ноды знают друг о друге                                     │
│  • Метаданные (список очередей, exchanges) реплицируются на ВСЕ    │
│  • Данные очередей (сообщения) — зависит от типа очереди           │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Что реплицируется в кластере:**

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║  МЕТАДАННЫЕ (реплицируются автоматически на ВСЕ ноды):              ║
║  ══════════════════════════════════════════════════════              ║
║  • Список очередей                                                   ║
║  • Список exchanges                                                  ║
║  • Bindings                                                          ║
║  • Users и permissions                                               ║
║  • Virtual hosts                                                     ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  ДАННЫЕ ОЧЕРЕДЕЙ (сообщения) — зависит от типа:                     ║
║  ════════════════════════════════════════════════════                ║
║                                                                      ║
║  Classic Queue (по умолчанию):                                       ║
║  • Сообщения хранятся на ОДНОЙ ноде (master)                        ║
║  • НЕ реплицируются автоматически                                   ║
║  • Если мастер упал → очередь недоступна ❌                          ║
║  • Для HA нужен Mirroring (устаревший способ)                       ║
║                                                                      ║
║  Quorum Queue (рекомендуется!):                                      ║
║  • Сообщения РЕПЛИЦИРУЮТСЯ на несколько нод (Raft)                  ║
║  • Переживает падение нод ✅                                         ║
║  • Нет единой "master" ноды                                         ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### Создание кластера — пошагово

**Предварительные требования:**

1. **Erlang Cookie** должен быть одинаковым на всех нодах
2. Ноды должны **видеть друг друга** по сети (hostname resolution)
3. Порты открыты: **4369** (epmd), **25672** (inter-node), **5672** (AMQP)

**Шаг 1: Установка RabbitMQ на каждую ноду**

Bash

```
# На каждой ноде (Ubuntu/Debian):
sudo apt update
sudo apt install rabbitmq-server

# Проверка
sudo systemctl status rabbitmq-server
```

**Шаг 2: Синхронизация Erlang Cookie**

Bash

```
# На Node 1:
# Cookie находится в /var/lib/rabbitmq/.erlang.cookie
sudo cat /var/lib/rabbitmq/.erlang.cookie
# Скопировать значение, например: ABCDEFGHIJKLMNOPQRST

# На Node 2 и Node 3:
sudo systemctl stop rabbitmq-server
echo "ABCDEFGHIJKLMNOPQRST" | sudo tee /var/lib/rabbitmq/.erlang.cookie
sudo chmod 400 /var/lib/rabbitmq/.erlang.cookie
sudo chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie
sudo systemctl start rabbitmq-server
```

**Шаг 3: Настройка /etc/hosts (DNS resolution)**

Bash

```
# На ВСЕХ нодах добавить в /etc/hosts:
192.168.1.10   node1 rabbit@node1
192.168.1.11   node2 rabbit@node2
192.168.1.12   node3 rabbit@node3
```

**Шаг 4: Объединение в кластер**

Bash

```
# ════════════════════════════════════════════════════════════
# НА NODE 2:
# ════════════════════════════════════════════════════════════

# Остановить приложение (НЕ полностью сервер!)
sudo rabbitmqctl stop_app

# Присоединиться к кластеру Node 1
sudo rabbitmqctl join_cluster rabbit@node1

# Запустить приложение
sudo rabbitmqctl start_app

# Проверка
sudo rabbitmqctl cluster_status
# Вывод:
# Cluster status of node rabbit@node2 ...
# [{nodes,[{disc,[rabbit@node1,rabbit@node2]}]}]


# ════════════════════════════════════════════════════════════
# НА NODE 3:
# ════════════════════════════════════════════════════════════

sudo rabbitmqctl stop_app
sudo rabbitmqctl join_cluster rabbit@node1
sudo rabbitmqctl start_app

# Проверка на ЛЮБОЙ ноде
sudo rabbitmqctl cluster_status
# Вывод:
# Cluster status of node rabbit@node3 ...
# [{nodes,[{disc,[rabbit@node1,rabbit@node2,rabbit@node3]}]}]
```

**Шаг 5: Включение Management Plugin (Web UI)**

Bash

```
# На ВСЕХ нодах
sudo rabbitmq-plugins enable rabbitmq_management

# Открыть в браузере (любая нода):
# http://node1:15672
# Логин: guest / Пароль: guest (только с localhost!)

# Создать admin пользователя:
sudo rabbitmqctl add_user admin StrongPassword123
sudo rabbitmqctl set_user_tags admin administrator
sudo rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
```

### Quorum Queues — гарантия отказоустойчивости

**Quorum Queue** — это **распределённая** очередь, которая использует протокол **Raft** для репликации данных. Это современный и рекомендуемый способ обеспечения HA.

**Как работает Quorum Queue:**

text

```
┌────────────────────────────────────────────────────────────────────┐
│               QUORUM QUEUE "orders" (3 реплики)                    │
│                                                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │   Node 1     │  │   Node 2     │  │   Node 3     │             │
│  │              │  │              │  │              │             │
│  │  Leader      │  │  Follower    │  │  Follower    │             │
│  │  [msg1,      │  │  [msg1,      │  │  [msg1,      │             │
│  │   msg2,      │  │   msg2,      │  │   msg2,      │             │
│  │   msg3]      │  │   msg3]      │  │   msg3]      │             │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘             │
│         │                 │                 │                     │
│         └─────── Raft Consensus ────────────┘                     │
│                                                                    │
│  Publisher пишет → Leader                                          │
│  Leader реплицирует → Followers                                    │
│  После записи на КВОРУМ (2 из 3) → ACK Publisher'у                 │
│                                                                    │
│  Если Leader (Node 1) упадёт:                                      │
│  • Автоматически выбирается новый Leader из Followers (Node 2)     │
│  • Без потери данных ✅                                            │
│  • Downtime < 10 секунд                                            │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Создание Quorum Queue:**

Python

```
channel.queue_declare(
    queue='orders_quorum',
    durable=True,  # ОБЯЗАТЕЛЬНО для quorum
    arguments={
        'x-queue-type': 'quorum',
        'x-quorum-initial-group-size': 3  # реплики на 3 нодах
    }
)

# Quorum Queue автоматически:
# • Создаёт реплики на 3 нодах
# • Выбирает Leader'а
# • Синхронизирует данные через Raft
```

**Преимущества Quorum Queue:**

text

```
✅ Автоматическая репликация (не нужно настраивать mirroring)
✅ Автоматический failover (не нужно вручную промоутить replica)
✅ Гарантия сохранности данных (записано на кворум → точно не потеряется)
✅ Поддержка at-least-once семантики
✅ Переживает падение (N-1)/2 нод
   • 3 ноды → выдержит падение 1 ноды
   • 5 нод → выдержит падение 2 нод

❌ Чуть медленнее, чем classic queue (из-за репликации)
❌ Требует минимум 3 ноды для production
```

**Mirrored Queues (устаревший способ, НЕ рекомендуется):**

До появления Quorum Queues использовались **Mirrored Queues** (зеркалированные очереди). Они ещё поддерживаются, но **deprecated** — используйте Quorum вместо них.

Bash

```
# Настройка mirroring через policy (НЕ делайте так в новых проектах!)
sudo rabbitmqctl set_policy ha-all "^orders\." \
  '{"ha-mode":"exactly","ha-params":2,"ha-sync-mode":"automatic"}' \
  --priority 1 --apply-to queues

# Это зеркалирует все очереди, начинающиеся с "orders."
# на 2 ноды (master + 1 mirror)

# Проблемы mirrored queues:
# • Сложная конфигурация
# • Нужна ручная promotion при падении master
# • Риск split-brain
# • Хуже производительность

# ВМЕСТО ЭТОГО: используйте Quorum Queues!
```

### Load Balancer и HA Proxy

Для **распределения подключений** клиентов между нодами кластера используйте **Load Balancer**:

text

```
┌────────────────────────────────────────────────────────────────────┐
│                                                                    │
│  Clients (Producers / Consumers)                                   │
│       │                                                            │
│       ▼                                                            │
│  ┌─────────────────────┐                                           │
│  │   Load Balancer     │                                           │
│  │   (HAProxy / Nginx) │                                           │
│  │   rabbitmq.example  │                                           │
│  └─────────┬───────────┘                                           │
│            │                                                       │
│      Round-robin                                                   │
│     ┌──────┼───────┐                                               │
│     │      │       │                                               │
│     ▼      ▼       ▼                                               │
│  Node 1  Node 2  Node 3                                            │
│  :5672   :5672   :5672                                             │
│                                                                    │
│  Если Node 2 упал → HAProxy убирает из ротации                     │
│  Клиенты автоматически перенаправляются на Node 1 и Node 3         │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**HAProxy конфигурация:**

text

```
# /etc/haproxy/haproxy.cfg

frontend rabbitmq_front
    bind *:5672
    mode tcp
    default_backend rabbitmq_back

backend rabbitmq_back
    mode tcp
    balance roundrobin
    option tcp-check
    
    server node1 192.168.1.10:5672 check inter 5s rise 2 fall 3
    server node2 192.168.1.11:5672 check inter 5s rise 2 fall 3
    server node3 192.168.1.12:5672 check inter 5s rise 2 fall 3

# Health check: HAProxy проверяет каждые 5 секунд
# rise 2  — нода считается UP после 2 успешных проверок
# fall 3  — нода считается DOWN после 3 неудачных проверок
```

### Мониторинг кластера

Bash

```
# Статус кластера
sudo rabbitmqctl cluster_status

# Вывод:
# Cluster name: rabbit@node1
# Nodes:
#   disc: [rabbit@node1, rabbit@node2, rabbit@node3]
# Running nodes: [rabbit@node1, rabbit@node2, rabbit@node3]
# Partitions: []  ← ВАЖНО: должно быть пустым!

# Если в Partitions что-то есть → network partition (split-brain)!


# Статус всех очередей
sudo rabbitmqctl list_queues name type policy state

# messages_ready    — сообщений ждут обработки
# messages_unacked  — сообщений у consumers (ещё не ACK)
# consumers         — количество подключённых consumers


# Aliveness test (жива ли нода?)
sudo rabbitmqctl node_health_check

# Вывод:
# Health check passed  ✅
```

**Метрики для Prometheus:**

Bash

```
# Включить Prometheus plugin
sudo rabbitmq-plugins enable rabbitmq_prometheus

# Метрики доступны на:
# http://node1:15692/metrics
```

**Ключевые метрики:**

text

```
rabbitmq_queue_messages           — количество сообщений в очередях
rabbitmq_queue_consumers          — количество consumers
rabbitmq_connections              — количество активных подключений
rabbitmq_channels                 — количество открытых каналов
rabbitmq_node_mem_used            — использование памяти
rabbitmq_node_disk_free           — свободное место на диске
rabbitmq_queue_messages_unacked   — неподтверждённые сообщения
```

---

## 3. Разграничение доступа (Authorization)

### Virtual Hosts (vhosts)

**Virtual Host** — это логическая изоляция внутри одного RabbitMQ сервера. Каждый vhost имеет:

- Свои exchange'ы
- Свои очереди
- Свои bindings
- Свои permissions

Это как **отдельные "квартиры" в одном доме** — полная изоляция.

text

```
┌────────────────────────────────────────────────────────────────────┐
│                   RABBITMQ SERVER                                  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  Virtual Host "/"   (default)                                │  │
│  │  • Exchange: amq.direct                                      │  │
│  │  • Queue: orders                                             │  │
│  │  • Users: guest, admin                                       │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  Virtual Host "/production"                                  │  │
│  │  • Exchange: orders_exchange                                 │  │
│  │  • Queue: orders_queue                                       │  │
│  │  • Users: prod_service                                       │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  Virtual Host "/staging"                                     │  │
│  │  • Exchange: orders_exchange                                 │  │
│  │  • Queue: orders_queue                                       │  │
│  │  • Users: dev_team                                           │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  • Полная изоляция между vhosts                                   │
│  • Одинаковые имена очередей в разных vhosts — НЕ конфликтуют     │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Управление vhosts:**

Bash

```
# Создать vhost
sudo rabbitmqctl add_vhost /production
sudo rabbitmqctl add_vhost /staging

# Список vhosts
sudo rabbitmqctl list_vhosts

# Удалить vhost (ОСТОРОЖНО: удаляет ВСЕ данные!)
sudo rabbitmqctl delete_vhost /staging
```

### Users и Permissions

**User (пользователь)** — это учётная запись для подключения к RabbitMQ. Каждый user имеет:

- **Username** и **password**
- **Tags** (роли): administrator, monitoring, management, policymaker, none
- **Permissions** для каждого vhost

Bash

```
# ════════════════════════════════════════════════════════════
# СОЗДАНИЕ ПОЛЬЗОВАТЕЛЕЙ
# ════════════════════════════════════════════════════════════

# Создать пользователя
sudo rabbitmqctl add_user order_service SecureP@ssw0rd123

# Установить tags (роли)
sudo rabbitmqctl set_user_tags order_service management

# Tags:
# • administrator — полный доступ (управление кластером)
# • monitoring    — только чтение (метрики, статус)
# • management    — доступ к Management UI
# • policymaker   — управление policies
# • none          — без тегов (только AMQP)


# ════════════════════════════════════════════════════════════
# УСТАНОВКА PERMISSIONS
# ════════════════════════════════════════════════════════════

sudo rabbitmqctl set_permissions -p /production order_service \
  "^orders.*"  "^orders.*"  "^orders.*"
  # ───┬──────  ────┬──────  ────┬──────
  #    │            │            └─ read (получать из очередей)
  #    │            └─ write (публиковать в exchanges)
  #    └─ configure (создавать/удалять queues/exchanges)

# Permissions — это REGEX паттерны!

# Примеры:
# ".*"         — доступ ко ВСЕМ ресурсам
# "^orders.*"  — доступ к ресурсам, начинающимся с "orders"
# "^$"         — НЕТ доступа (пустой паттерн)
```

**Типы permissions:**

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                     ТИПЫ PERMISSIONS                                 ║
╠═══════════════╦══════════════════════════════════════════════════════╣
║  Permission   ║  Описание                                            ║
╠═══════════════╬══════════════════════════════════════════════════════╣
║ CONFIGURE     ║ Создавать/удалять/изменять queues и exchanges        ║
║               ║ Объявлять bindings                                   ║
║               ║                                                      ║
║               ║ Операции:                                            ║
║               ║ • queue.declare                                      ║
║               ║ • queue.delete                                       ║
║               ║ • exchange.declare                                   ║
║               ║ • exchange.delete                                    ║
║               ║ • queue.bind / queue.unbind                          ║
╠═══════════════╬══════════════════════════════════════════════════════╣
║ WRITE         ║ Публиковать сообщения в exchanges                    ║
║               ║                                                      ║
║               ║ Операции:                                            ║
║               ║ • basic.publish                                      ║
╠═══════════════╬══════════════════════════════════════════════════════╣
║ READ          ║ Читать сообщения из очередей                         ║
║               ║ Подтверждать/отклонять сообщения                     ║
║               ║                                                      ║
║               ║ Операции:                                            ║
║               ║ • basic.get                                          ║
║               ║ • basic.consume                                      ║
║               ║ • basic.ack / basic.nack                             ║
║               ║ • queue.purge                                        ║
╚═══════════════╩══════════════════════════════════════════════════════╝
```

**Практические примеры:**

Bash

```
# ════════════════════════════════════════════════════════════
# ПРИМЕР 1: Order Service — может ПИСАТЬ в "orders.*"
# ════════════════════════════════════════════════════════════

sudo rabbitmqctl add_user order_service P@ss123
sudo rabbitmqctl set_user_tags order_service none

sudo rabbitmqctl set_permissions -p /production order_service \
  "^$"          \  # НЕ может создавать ресурсы
  "^orders.*"   \  # МОЖЕТ публиковать в exchanges "orders.*"
  "^$"             # НЕ может читать из очередей

# Теперь order_service может:
# ✅ Публиковать в "orders_exchange"
# ❌ Создавать новые queues/exchanges
# ❌ Читать из очередей


# ════════════════════════════════════════════════════════════
# ПРИМЕР 2: Payment Service — может ЧИТАТЬ "orders.*" и ПИСАТЬ "payments.*"
# ════════════════════════════════════════════════════════════

sudo rabbitmqctl add_user payment_service P@ss456
sudo rabbitmqctl set_user_tags payment_service none

sudo rabbitmqctl set_permissions -p /production payment_service \
  "^$"                          \  # НЕ может создавать ресурсы
  "^payments.*"                 \  # МОЖЕТ публиковать в "payments.*"
  "^orders.*|^payments_queue$"     # МОЖЕТ читать из "orders.*" и "payments_queue"


# ════════════════════════════════════════════════════════════
# ПРИМЕР 3: Analytics Service — ТОЛЬКО ЧТЕНИЕ
# ════════════════════════════════════════════════════════════

sudo rabbitmqctl add_user analytics_service P@ss789
sudo rabbitmqctl set_user_tags analytics_service monitoring

sudo rabbitmqctl set_permissions -p /production analytics_service \
  "^analytics_queue$"  \  # МОЖЕТ создать ТОЛЬКО "analytics_queue"
  "^$"                 \  # НЕ может публиковать
  ".*"                    # МОЖЕТ читать из ВСЕХ очередей

# Analytics может читать ВСЁ, но НЕ публиковать


# ════════════════════════════════════════════════════════════
# ПРИМЕР 4: Admin — полный доступ
# ════════════════════════════════════════════════════════════

sudo rabbitmqctl add_user admin SuperSecureP@ss
sudo rabbitmqctl set_user_tags admin administrator

sudo rabbitmqctl set_permissions -p /production admin \
  ".*"  ".*"  ".*"

# Admin может делать ВСЁ
```

**Просмотр permissions:**

Bash

```
# Permissions конкретного пользователя
sudo rabbitmqctl list_user_permissions order_service

# Вывод:
# Listing permissions for user "order_service" ...
# vhost         configure  write       read
# /production   ^$         ^orders.*   ^$

# Permissions для vhost
sudo rabbitmqctl list_permissions -p /production

# Вывод:
# Listing permissions for vhost "/production" ...
# user               configure  write       read
# order_service      ^$         ^orders.*   ^$
# payment_service    ^$         ^payments.* ^orders.*|^payments_queue$
# admin              .*         .*          .*
```

**Изменение пароля:**

Bash

```
# Сменить пароль пользователя
sudo rabbitmqctl change_password order_service NewP@ssw0rd456

# Удалить пользователя
sudo rabbitmqctl delete_user old_service
```

---

## 4. Защита данных от перехвата

### Шифрование in-transit (SSL/TLS)

**SSL/TLS** шифрует **все данные**, передаваемые между клиентами и RabbitMQ, а также между нодами кластера. Без шифрования данные передаются **в открытом виде** по сети.

text

```
БЕЗ TLS:

Publisher ──── {"card": "4111-1111-1111-1111"} ────▶ RabbitMQ
                      ↑
        Злоумышленник видит данные в открытом виде!


С TLS:

Publisher ──── 0xA7F3B2... (зашифровано) ────▶ RabbitMQ
                      ↑
        Злоумышленник видит ТОЛЬКО зашифрованный трафик
```

**Настройка SSL/TLS — пошагово:**

Bash

```
# ════════════════════════════════════════════════════════════
# ШАГ 1: Генерация сертификатов
# ════════════════════════════════════════════════════════════

# RabbitMQ предоставляет удобный скрипт для генерации:
git clone https://github.com/rabbitmq/tls-gen
cd tls-gen/basic

# Генерировать CA и сертификаты сервера/клиента
make
make verify

# Создаёт:
# • testca/      — Certificate Authority
# • server/      — Сертификат сервера
# • client/      — Сертификат клиента (для mutual TLS)

# Скопировать сертификаты
sudo mkdir -p /etc/rabbitmq/ssl
sudo cp result/server_*.pem /etc/rabbitmq/ssl/
sudo cp result/ca_certificate.pem /etc/rabbitmq/ssl/
sudo chmod 644 /etc/rabbitmq/ssl/*
```

**Конфигурация RabbitMQ:**

erlang

```
% /etc/rabbitmq/rabbitmq.conf

% ════════════════════════════════════════════════════════════
% SSL LISTENERS
% ════════════════════════════════════════════════════════════

listeners.ssl.default = 5671

% Пути к сертификатам
ssl_options.cacertfile = /etc/rabbitmq/ssl/ca_certificate.pem
ssl_options.certfile   = /etc/rabbitmq/ssl/server_certificate.pem
ssl_options.keyfile    = /etc/rabbitmq/ssl/server_key.pem

% Проверка сертификата клиента (mutual TLS)
% verify_peer — требовать сертификат от клиента
% verify_none — не требовать
ssl_options.verify     = verify_peer
ssl_options.fail_if_no_peer_cert = true

% Разрешённые TLS версии (ТОЛЬКО современные!)
ssl_options.versions.1 = tlsv1.3
ssl_options.versions.2 = tlsv1.2

% Cipher suites (ТОЛЬКО безопасные!)
ssl_options.ciphers.1 = TLS_AES_256_GCM_SHA384
ssl_options.ciphers.2 = TLS_AES_128_GCM_SHA256
ssl_options.ciphers.3 = TLS_CHACHA20_POLY1305_SHA256

% Проверка имени хоста в сертификате клиента
ssl_options.depth = 2
```

**Перезапустить RabbitMQ:**

Bash

```
sudo systemctl restart rabbitmq-server

# Проверка
sudo rabbitmqctl status | grep ssl
# Должно быть: {listeners,[{ssl,5671}]}
```

**Подключение клиента с TLS:**

Python

```
import pika
import ssl

# ════════════════════════════════════════════════════════════
# ПОДКЛЮЧЕНИЕ С TLS
# ════════════════════════════════════════════════════════════

# SSL контекст
ssl_context = ssl.create_default_context(
    cafile="/path/to/ca_certificate.pem"  # CA сертификат для проверки сервера
)

# Для mutual TLS (если сервер требует сертификат клиента):
ssl_context.load_cert_chain(
    certfile="/path/to/client_certificate.pem",
    keyfile="/path/to/client_key.pem"
)

# Параметры подключения
credentials = pika.PlainCredentials('username', 'password')
parameters = pika.ConnectionParameters(
    host='rabbitmq-server.example.com',
    port=5671,  # SSL порт
    virtual_host='/',
    credentials=credentials,
    ssl_options=pika.SSLOptions(ssl_context)
)

connection = pika.BlockingConnection(parameters)
channel = connection.channel()

# Теперь ВСЯ коммуникация зашифрована! ✅
```

### Шифрование между нодами кластера

По умолчанию ноды RabbitMQ кластера общаются **без шифрования**. Для production нужно включить **inter-node TLS**:

erlang

```
% /etc/rabbitmq/rabbitmq.conf

% Порт для inter-node коммуникации с TLS
cluster_formation.peer_discovery_backend = rabbit_peer_discovery_classic_config
cluster_formation.classic_config.nodes.1 = rabbit@node1
cluster_formation.classic_config.nodes.2 = rabbit@node2
cluster_formation.classic_config.nodes.3 = rabbit@node3

% Inter-node TLS
cluster_ssl_options.cacertfile = /etc/rabbitmq/ssl/ca_certificate.pem
cluster_ssl_options.certfile   = /etc/rabbitmq/ssl/server_certificate.pem
cluster_ssl_options.keyfile    = /etc/rabbitmq/ssl/server_key.pem
cluster_ssl_options.verify     = verify_peer
cluster_ssl_options.fail_if_no_peer_cert = true
```

### End-to-End Encryption (E2EE)

Для **максимальной безопасности** можно шифровать данные **на уровне приложения** (до отправки в RabbitMQ). RabbitMQ будет хранить и передавать **зашифрованные** данные, **не имея возможности** их прочитать.

Python

```
# ════════════════════════════════════════════════════════════
# PUBLISHER: шифрование перед отправкой
# ════════════════════════════════════════════════════════════

from cryptography.fernet import Fernet

# Ключ шифрования (в production — из Vault!)
key = Fernet.generate_key()
cipher = Fernet(key)

# Шифровать данные
plaintext = '{"orderId": 12345, "card": "4111-1111-1111-1111"}'
encrypted = cipher.encrypt(plaintext.encode())

# Отправить зашифрованное
channel.basic_publish(
    exchange='orders',
    routing_key='order.created',
    body=encrypted  # RabbitMQ хранит зашифрованные данные
)


# ════════════════════════════════════════════════════════════
# CONSUMER: расшифровка после получения
# ════════════════════════════════════════════════════════════

def on_message(ch, method, properties, body):
    # Расшифровать
    decrypted = cipher.decrypt(body)
    plaintext = decrypted.decode()
    
    order = json.loads(plaintext)
    print(f"Заказ: {order}")
    
    ch.basic_ack(delivery_tag=method.delivery_tag)
```

### Network Isolation

text

```
┌──────────────────────────────────────────────────────────────────┐
│                    СЕТЕВАЯ ИЗОЛЯЦИЯ                               │
│                                                                  │
│  ┌─────────────────────────────────────────────────┐             │
│  │              PRIVATE SUBNET                      │             │
│  │                                                  │             │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐       │             │
│  │  │RabbitMQ 1│  │RabbitMQ 2│  │RabbitMQ 3│       │             │
│  │  │ :5671    │  │ :5671    │  │ :5671    │       │             │
│  │  └──────────┘  └──────────┘  └──────────┘       │             │
│  │         ▲            ▲            ▲              │             │
│  │         │            │            │              │             │
│  │         └─── ТОЛЬКО из Application Subnet ───   │             │
│  │                      │                           │             │
│  │  ┌──────────────────────────────────────────┐    │             │
│  │  │       APPLICATION SUBNET                 │    │             │
│  │  │  ┌──────────┐  ┌──────────┐              │    │             │
│  │  │  │Service A │  │Service B │              │    │             │
│  │  │  └──────────┘  └──────────┘              │    │             │
│  │  └──────────────────────────────────────────┘    │             │
│  └─────────────────────────────────────────────────┘             │
│                                                                  │
│  Firewall:                                                       │
│  ✅ Application → RabbitMQ: разрешить 5671 (TLS)                 │
│  ✅ RabbitMQ ↔ RabbitMQ: разрешить 25672 (clustering)            │
│  ❌ Internet → RabbitMQ: ЗАПРЕТИТЬ                               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Security Checklist

text

```
╔══════════════════════════════════════════════════════════════════════╗
║              PRODUCTION SECURITY CHECKLIST                           ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  ═══ АУТЕНТИФИКАЦИЯ ═══                                              ║
║  ☐ Отключён guest пользователь (или разрешён ТОЛЬКО с localhost)    ║
║  ☐ Созданы отдельные users для каждого сервиса                      ║
║  ☐ Сильные пароли (или client certificates)                         ║
║                                                                      ║
║  ═══ АВТОРИЗАЦИЯ ═══                                                 ║
║  ☐ Virtual hosts для изоляции окружений (prod/staging/dev)          ║
║  ☐ Минимальные permissions (принцип наименьших привилегий)          ║
║  ☐ Регулярный аудит permissions                                      ║
║                                                                      ║
║  ═══ ШИФРОВАНИЕ ═══                                                  ║
║  ☐ TLS для всех клиентских подключений (порт 5671)                  ║
║  ☐ Inter-node TLS (шифрование между нодами кластера)                ║
║  ☐ Только TLS 1.2+ (отключить TLS 1.0, 1.1)                        ║
║  ☐ Mutual TLS (сертификаты клиентов) для критичных сервисов         ║
║  ☐ E2E шифрование для конфиденциальных данных                       ║
║                                                                      ║
║  ═══ СЕТЬ ═══                                                        ║
║  ☐ RabbitMQ в private subnet (НЕ доступен из интернета)             ║
║  ☐ Firewall: только необходимые порты                                ║
║  ☐ Load Balancer для распределения нагрузки                          ║
║                                                                      ║
║  ═══ ОТКАЗОУСТОЙЧИВОСТЬ ═══                                          ║
║  ☐ Кластер минимум из 3 нод                                         ║
║  ☐ Quorum Queues для критичных очередей                             ║
║  ☐ Мониторинг (Prometheus + Grafana)                                ║
║  ☐ Алерты на критичные события                                       ║
║                                                                      ║
║  ═══ ОПЕРАЦИОННАЯ БЕЗОПАСНОСТЬ ═══                                   ║
║  ☐ Регулярные обновления (security patches)                         ║
║  ☐ Бэкапы конфигурации и definitions                                ║
║  ☐ Audit logging (кто что делал)                                    ║
║  ☐ Rate limiting (защита от DDoS)                                   ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```