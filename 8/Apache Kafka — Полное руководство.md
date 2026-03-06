## 1. Установка Kafka — Детальное руководство

### Введение в установку

Apache Kafka — это сложная распределённая система, и существует несколько способов её установки, каждый из которых подходит для разных сценариев использования. Прежде чем приступить к установке, важно понимать, что Kafka требует Java Runtime Environment (JRE) версии 11 или выше. Это фундаментальное требование, потому что Kafka написана на Scala и Java и компилируется в байт-код JVM.

### Подготовка окружения

**Установка Java:**

Java является обязательной зависимостью для Kafka. Без неё брокер просто не запустится. Kafka активно использует возможности современных версий Java, включая оптимизации garbage collection, которые критичны для производительности при обработке миллионов сообщений в секунду.

Bash

```
# Проверка текущей версии Java
java -version

# Вывод должен показать версию 11 или выше:
# openjdk version "11.0.18" 2023-01-17
# OpenJDK Runtime Environment (build 11.0.18+10)
# OpenJDK 64-Bit Server VM (build 11.0.18+10, mixed mode)

# Если Java не установлена или версия ниже 11:

# Ubuntu/Debian (APT):
sudo apt update
sudo apt install openjdk-11-jdk -y

# После установки проверьте, что JAVA_HOME установлена:
echo $JAVA_HOME
# Если пусто, добавьте в ~/.bashrc:
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin

# macOS (Homebrew):
brew update
brew install openjdk@11
# Homebrew обычно требует дополнительного шага для линковки:
sudo ln -sfn /usr/local/opt/openjdk@11/libexec/openjdk.jdk \
  /Library/Java/JavaVirtualMachines/openjdk-11.jdk

# CentOS/RHEL (YUM):
sudo yum install java-11-openjdk-devel -y

# Проверка после установки:
java -version
javac -version  # компилятор Java (для разработки)
```

### Способ 1: Установка бинарной версии Kafka (для разработки и тестирования)

Этот способ идеально подходит для локальной разработки, экспериментов и обучения. Вы скачиваете готовые бинарные файлы, распаковываете и запускаете Kafka локально на своей машине.

**Шаг 1: Скачивание Kafka**

Bash

```
# Перейдите на https://kafka.apache.org/downloads
# Или используйте wget/curl для скачивания:

# Создайте директорию для Kafka
mkdir -p ~/kafka && cd ~/kafka

# Скачайте последнюю версию (проверьте актуальную на сайте)
# Формат имени: kafka_<scala-version>-<kafka-version>.tgz
wget https://downloads.apache.org/kafka/3.6.1/kafka_2.13-3.6.1.tgz

# Альтернативно через curl:
curl -O https://downloads.apache.org/kafka/3.6.1/kafka_2.13-3.6.1.tgz

# Проверка контрольной суммы (опционально, но рекомендуется):
wget https://downloads.apache.org/kafka/3.6.1/kafka_2.13-3.6.1.tgz.sha512
shasum -a 512 kafka_2.13-3.6.1.tgz
# Сравните вывод с содержимым .sha512 файла
```

**Понимание имени файла:**

- `kafka` — имя проекта
- `2.13` — версия Scala (язык, на котором написана Kafka)
- `3.6.1` — версия самой Kafka

**Шаг 2: Распаковка и структура директорий**

Bash

```
# Распаковать архив
tar -xzf kafka_2.13-3.6.1.tgz

# Перейти в директорию Kafka
cd kafka_2.13-3.6.1

# Изучим структуру директорий:
ls -la

# bin/        — исполняемые скрипты (start/stop серверов, CLI утилиты)
# config/     — конфигурационные файлы
# libs/       — JAR-библиотеки Kafka и зависимостей
# licenses/   — лицензии используемых библиотек
# site-docs/  — документация
```

**Шаг 3: Запуск Kafka в KRaft режиме (современный способ БЕЗ ZooKeeper)**

KRaft (Kafka Raft) — это новая архитектура консенсуса, встроенная в саму Kafka, которая полностью заменяет зависимость от ZooKeeper. Это упрощает развёртывание, улучшает производительность и позволяет масштабировать до миллионов партиций.

Bash

```
# Шаг 3.1: Генерация уникального идентификатора кластера
# Каждый Kafka кластер должен иметь уникальный UUID
KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"

# Этот ID будет что-то вроде:
# MkU3OEVBNTcwNTJENDM2Qk

# Вывести ID для проверки:
echo $KAFKA_CLUSTER_ID

# Шаг 3.2: Форматирование директории для хранения логов
# Это аналог "форматирования диска" — подготовка хранилища
bin/kafka-storage.sh format \
  -t $KAFKA_CLUSTER_ID \
  -c config/kraft/server.properties

# Вывод будет примерно таким:
# Formatting /tmp/kraft-combined-logs with metadata.version 3.6-IV2.

# Что происходит:
# - Создаётся директория для логов (указана в server.properties)
# - Записываются метаданные кластера
# - Инициализируется Raft лог для координации

# Шаг 3.3: Запуск Kafka сервера
bin/kafka-server-start.sh config/kraft/server.properties

# Kafka начнёт инициализацию. Вы увидите много логов:
# [2024-01-15 10:30:00,123] INFO Kafka version: 3.6.1
# [2024-01-15 10:30:00,456] INFO Kafka commitId: ...
# [2024-01-15 10:30:01,789] INFO [KafkaRaftServer nodeId=1] Kafka Server started
# [2024-01-15 10:30:02,012] INFO Awaiting socket connections on 0.0.0.0:9092

# Последняя строка означает, что Kafka готова принимать соединения! ✅

# Kafka теперь слушает на localhost:9092
```

**Понимание KRaft конфигурации (config/kraft/server.properties):**

properties

```
# Роли этого узла: broker (обработка данных) + controller (управление метаданными)
process.roles=broker,controller

# Уникальный ID узла в кластере
node.id=1

# Listener'ы — адреса, на которых Kafka принимает соединения
# CONTROLLER — для внутренней координации между брокерами
# PLAINTEXT — для клиентов (producer/consumer)
listeners=PLAINTEXT://:9092,CONTROLLER://:9093

# Advertised listeners — адреса, которые Kafka возвращает клиентам
# Клиенты будут подключаться именно по этим адресам
advertised.listeners=PLAINTEXT://localhost:9092

# Кворум контроллеров для голосования (Raft)
# Формат: node.id@host:port
# В production будет несколько узлов: 1@host1:9093,2@host2:9093,3@host3:9093
controller.quorum.voters=1@localhost:9093

# Директория для хранения логов сообщений
log.dirs=/tmp/kraft-combined-logs

# Количество партиций по умолчанию при создании топика
num.partitions=1

# Период хранения логов (168 часов = 7 дней)
log.retention.hours=168
```

**Запуск в фоновом режиме:**

Bash

```
# Запустить Kafka в фоне (daemon)
bin/kafka-server-start.sh -daemon config/kraft/server.properties

# Проверить, что процесс запущен:
ps aux | grep kafka

# Логи пишутся в logs/server.log
tail -f logs/server.log

# Остановка:
bin/kafka-server-stop.sh
```

---

### Способ 2: Установка через Docker (рекомендуется для разработки)

Docker-контейнеры — это самый простой и быстрый способ поднять Kafka локально. Вам не нужно устанавливать Java, настраивать переменные окружения или вручную управлять процессами. Всё инкапсулировано в контейнере.

**Предварительные требования:**

Bash

```
# Установите Docker Desktop (macOS/Windows) или Docker Engine (Linux)
# Проверка установки:
docker --version
# Docker version 24.0.7, build afdd53b

docker-compose --version
# Docker Compose version v2.23.0
```

**docker-compose.yml — полная конфигурация:**

Создайте файл `docker-compose.yml` в новой директории:

YAML

```
version: '3.8'

services:
  # ════════════════════════════════════════════════════════════
  #  KAFKA В KRAFT РЕЖИМЕ (без ZooKeeper)
  # ════════════════════════════════════════════════════════════
  kafka:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka
    hostname: kafka
    ports:
      # Порт для клиентов (producer/consumer)
      - "9092:9092"
      # JMX порт для мониторинга (опционально)
      - "9101:9101"
    environment:
      # ─────────────────────────────────────────────────────
      # KRaft Configuration
      # ─────────────────────────────────────────────────────
      
      # Уникальный ID узла
      KAFKA_NODE_ID: 1
      
      # Роли: broker + controller (combined mode для dev)
      KAFKA_PROCESS_ROLES: 'broker,controller'
      
      # Кворум для голосования (в production — несколько узлов)
      KAFKA_CONTROLLER_QUORUM_VOTERS: '1@kafka:29093'
      
      # ─────────────────────────────────────────────────────
      # Listener Configuration (критически важно!)
      # ─────────────────────────────────────────────────────
      
      # Определение протоколов безопасности для каждого listener'а
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: 'CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT'
      
      # Адреса, на которых Kafka слушает соединения
      # PLAINTEXT       — внутри Docker сети (для других контейнеров)
      # CONTROLLER      — для координации между брокерами
      # PLAINTEXT_HOST  — для клиентов с хост-машины
      KAFKA_LISTENERS: 'PLAINTEXT://kafka:29092,CONTROLLER://kafka:29093,PLAINTEXT_HOST://0.0.0.0:9092'
      
      # Адреса, которые Kafka возвращает клиентам
      # Клиенты подключаются именно по этим адресам!
      KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092'
      
      # Какой listener используется для inter-broker коммуникации
      KAFKA_INTER_BROKER_LISTENER_NAME: 'PLAINTEXT'
      
      # Какой listener controller использует
      KAFKA_CONTROLLER_LISTENER_NAMES: 'CONTROLLER'
      
      # ─────────────────────────────────────────────────────
      # Storage Configuration
      # ─────────────────────────────────────────────────────
      
      # Директория для хранения логов внутри контейнера
      KAFKA_LOG_DIRS: '/tmp/kraft-combined-logs'
      
      # ─────────────────────────────────────────────────────
      # Replication & Fault Tolerance (для dev — минимальные)
      # ─────────────────────────────────────────────────────
      
      # Replication factor для служебного топика offsets
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      
      # Минимальное кол-во in-sync реплик для транзакций
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      
      # ─────────────────────────────────────────────────────
      # Performance Tuning
      # ─────────────────────────────────────────────────────
      
      # Задержка перед первым ребалансом consumer group (0 для dev)
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      
      # ─────────────────────────────────────────────────────
      # JMX Monitoring (опционально)
      # ─────────────────────────────────────────────────────
      
      KAFKA_JMX_PORT: 9101
      KAFKA_JMX_HOSTNAME: localhost
      
      # ─────────────────────────────────────────────────────
      # Cluster ID (фиксированный для dev, в prod — генерируется)
      # ─────────────────────────────────────────────────────
      
      CLUSTER_ID: 'MkU3OEVBNTcwNTJENDM2Qk'
      
    volumes:
      # Персистентное хранилище данных
      - kafka-data:/var/lib/kafka/data
    
    # Health check — проверка, что Kafka готова
    healthcheck:
      test: ["CMD", "kafka-broker-api-versions", "--bootstrap-server", "localhost:9092"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ════════════════════════════════════════════════════════════
  #  KAFKA UI — веб-интерфейс для управления (опционально)
  # ════════════════════════════════════════════════════════════
  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    container_name: kafka-ui
    ports:
      - "8080:8080"
    environment:
      # Подключение к Kafka кластеру
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:29092
      
      # Дополнительные настройки UI
      KAFKA_CLUSTERS_0_METRICS_PORT: 9101
      DYNAMIC_CONFIG_ENABLED: 'true'
    
    depends_on:
      kafka:
        condition: service_healthy
    
    # Health check для UI
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:8080/actuator/health"]
      interval: 10s
      timeout: 5s
      retries: 3

# ════════════════════════════════════════════════════════════
#  VOLUMES — персистентное хранилище
# ════════════════════════════════════════════════════════════
volumes:
  kafka-data:
    driver: local
```

**Запуск и управление:**

Bash

```
# Запустить все сервисы в фоновом режиме
docker-compose up -d

# Вывод:
# [+] Running 3/3
#  ✔ Network kafka_default      Created
#  ✔ Container kafka            Started
#  ✔ Container kafka-ui         Started

# Проверить статус контейнеров
docker-compose ps

# NAME       IMAGE                              STATUS       PORTS
# kafka      confluentinc/cp-kafka:7.5.0        Up (healthy) 0.0.0.0:9092->9092/tcp
# kafka-ui   provectuslabs/kafka-ui:latest      Up (healthy) 0.0.0.0:8080->8080/tcp

# Просмотр логов Kafka в реальном времени
docker-compose logs -f kafka

# Просмотр логов UI
docker-compose logs -f kafka-ui

# Открыть Kafka UI в браузере:
# http://localhost:8080
# Вы увидите красивый веб-интерфейс с информацией о топиках, 
# consumer groups, брокерах, возможностью отправлять/читать сообщения

# Остановить все сервисы
docker-compose down

# Остановить И УДАЛИТЬ данные (volume)
docker-compose down -v

# Перезапустить только Kafka
docker-compose restart kafka
```

**Подключение к Kafka из других Docker контейнеров:**

Если вы хотите, чтобы другие ваши сервисы в Docker подключались к Kafka:

YAML

```
# my-app/docker-compose.yml
version: '3.8'

services:
  my-application:
    image: my-app:latest
    environment:
      # Используйте внутреннее имя сервиса и порт
      KAFKA_BOOTSTRAP_SERVERS: kafka:29092
    networks:
      - kafka_default  # подключаемся к сети Kafka

networks:
  kafka_default:
    external: true  # используем существующую сеть
```

**Продвинутая конфигурация — Multi-broker кластер в Docker:**

Для более реалистичного тестирования можно поднять кластер из нескольких брокеров:

YAML

```
version: '3.8'

services:
  kafka-1:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka-1
    ports:
      - "9092:9092"
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: 'broker,controller'
      KAFKA_CONTROLLER_QUORUM_VOTERS: '1@kafka-1:29093,2@kafka-2:29093,3@kafka-3:29093'
      KAFKA_LISTENERS: 'PLAINTEXT://kafka-1:29092,CONTROLLER://kafka-1:29093,PLAINTEXT_HOST://0.0.0.0:9092'
      KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://kafka-1:29092,PLAINTEXT_HOST://localhost:9092'
      KAFKA_INTER_BROKER_LISTENER_NAME: 'PLAINTEXT'
      KAFKA_CONTROLLER_LISTENER_NAMES: 'CONTROLLER'
      KAFKA_LOG_DIRS: '/tmp/kraft-combined-logs'
      CLUSTER_ID: 'MkU3OEVBNTcwNTJENDM2Qk'
    volumes:
      - kafka-1-data:/var/lib/kafka/data

  kafka-2:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka-2
    ports:
      - "9093:9093"
    environment:
      KAFKA_NODE_ID: 2
      KAFKA_PROCESS_ROLES: 'broker,controller'
      KAFKA_CONTROLLER_QUORUM_VOTERS: '1@kafka-1:29093,2@kafka-2:29093,3@kafka-3:29093'
      KAFKA_LISTENERS: 'PLAINTEXT://kafka-2:29092,CONTROLLER://kafka-2:29093,PLAINTEXT_HOST://0.0.0.0:9093'
      KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://kafka-2:29092,PLAINTEXT_HOST://localhost:9093'
      KAFKA_INTER_BROKER_LISTENER_NAME: 'PLAINTEXT'
      KAFKA_CONTROLLER_LISTENER_NAMES: 'CONTROLLER'
      KAFKA_LOG_DIRS: '/tmp/kraft-combined-logs'
      CLUSTER_ID: 'MkU3OEVBNTcwNTJENDM2Qk'
    volumes:
      - kafka-2-data:/var/lib/kafka/data

  kafka-3:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka-3
    ports:
      - "9094:9094"
    environment:
      KAFKA_NODE_ID: 3
      KAFKA_PROCESS_ROLES: 'broker,controller'
      KAFKA_CONTROLLER_QUORUM_VOTERS: '1@kafka-1:29093,2@kafka-2:29093,3@kafka-3:29093'
      KAFKA_LISTENERS: 'PLAINTEXT://kafka-3:29092,CONTROLLER://kafka-3:29093,PLAINTEXT_HOST://0.0.0.0:9094'
      KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://kafka-3:29092,PLAINTEXT_HOST://localhost:9094'
      KAFKA_INTER_BROKER_LISTENER_NAME: 'PLAINTEXT'
      KAFKA_CONTROLLER_LISTENER_NAMES: 'CONTROLLER'
      KAFKA_LOG_DIRS: '/tmp/kraft-combined-logs'
      CLUSTER_ID: 'MkU3OEVBNTcwNTJENDM2Qk'
    volumes:
      - kafka-3-data:/var/lib/kafka/data

volumes:
  kafka-1-data:
  kafka-2-data:
  kafka-3-data:
```

Теперь у вас кластер из 3 брокеров, и можно экспериментировать с репликацией, отказоустойчивостью и ребалансировкой.

---

### Способ 3: Установка со ZooKeeper (старый способ, legacy)

До версии 2.8 Kafka полностью зависела от Apache ZooKeeper для координации и хранения метаданных. Хотя KRaft — это будущее, многие production системы всё ещё используют ZooKeeper, и вам может понадобиться знать, как с ней работать.

**Что такое ZooKeeper и зачем она нужна:**

ZooKeeper — это распределённая система координации, которая предоставляет:

- **Централизованное хранилище конфигурации** — все брокеры Kafka знают, где искать метаданные
- **Выбор лидера** (leader election) — кто из брокеров управляет конкретной партицией
- **Обнаружение сервисов** (service discovery) — какие брокеры живы, какие упали
- **Синхронизацию** — обеспечивает согласованность между брокерами

**Архитектура с ZooKeeper:**

text

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌──────────┐      ┌──────────┐      ┌──────────┐              │
│  │ Kafka    │      │ Kafka    │      │ Kafka    │              │
│  │ Broker 1 │      │ Broker 2 │      │ Broker 3 │              │
│  └────┬─────┘      └────┬─────┘      └────┬─────┘              │
│       │                 │                 │                    │
│       │  Регистрация,   │                 │                    │
│       │  heartbeats,    │                 │                    │
│       │  leader election│                 │                    │
│       │                 │                 │                    │
│       └─────────────────┼─────────────────┘                    │
│                         │                                      │
│                         ▼                                      │
│              ┌─────────────────────┐                           │
│              │   ZooKeeper         │                           │
│              │   Ensemble          │                           │
│              │   (кластер 3-5 нод) │                           │
│              └─────────────────────┘                           │
│                                                                 │
│  ZooKeeper хранит:                                              │
│  • Список живых брокеров                                        │
│  • Метаданные топиков (партиции, replication factor)           │
│  • Конфигурация (retention, compression, etc.)                  │
│  • Leader'ы партиций                                            │
│  • ACL (права доступа)                                          │
│  • Consumer group offsets (старые версии Kafka)                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Запуск Kafka со ZooKeeper (локально):**

Bash

```
cd kafka_2.13-3.6.1

# ШАГ 1: Запустить ZooKeeper
# ZooKeeper должен быть запущен ПЕРВЫМ, т.к. Kafka зависит от неё
bin/zookeeper-server-start.sh config/zookeeper.properties

# Логи ZooKeeper:
# [2024-01-15 10:30:00,123] INFO binding to port 0.0.0.0/0.0.0.0:2181
# [2024-01-15 10:30:00,456] INFO Server environment:zookeeper.version=3.8.0
# ...
# [2024-01-15 10:30:01,789] INFO Started AdminServer on address 0.0.0.0, port 8080

# ZooKeeper слушает на порту 2181 (по умолчанию)

# ШАГ 2: Открыть НОВЫЙ терминал и запустить Kafka
bin/kafka-server-start.sh config/server.properties

# Kafka подключится к ZooKeeper и зарегистрируется
# Логи Kafka:
# [2024-01-15 10:30:05,123] INFO [KafkaServer id=0] started
# [2024-01-15 10:30:05,456] INFO Kafka version: 3.6.1
# ...
# [2024-01-15 10:30:06,789] INFO Registered broker 0 at path /brokers/ids/0 with addresses: PLAINTEXT://localhost:9092

# Kafka готова к работе!
```

**Конфигурация ZooKeeper (config/zookeeper.properties):**

properties

```
# Директория для хранения данных ZooKeeper
dataDir=/tmp/zookeeper

# Порт для клиентских соединений (Kafka подключается сюда)
clientPort=2181

# Максимальное количество клиентских соединений
maxClientCnxns=0

# Настройки синхронизации и таймаутов
tickTime=2000
initLimit=5
syncLimit=2

# Для production кластера ZooKeeper нужно несколько нод:
# server.1=zoo1:2888:3888
# server.2=zoo2:2888:3888
# server.3=zoo3:2888:3888
```

**Конфигурация Kafka для работы с ZooKeeper (config/server.properties):**

properties

```
# Уникальный ID брокера (в кластере все брокеры имеют разные ID)
broker.id=0

# Адреса listener'ов
listeners=PLAINTEXT://localhost:9092

# Директория для логов
log.dirs=/tmp/kafka-logs

# ★ КЛЮЧЕВАЯ НАСТРОЙКА: адрес ZooKeeper
# Kafka подключается к ZooKeeper по этому адресу
zookeeper.connect=localhost:2181

# Если ZooKeeper кластер из нескольких нод:
# zookeeper.connect=zoo1:2181,zoo2:2181,zoo3:2181

# Можно указать namespace (чтобы несколько Kafka кластеров использовали один ZooKeeper):
# zookeeper.connect=localhost:2181/kafka-cluster-1

# Таймауты для соединения с ZooKeeper
zookeeper.connection.timeout.ms=18000
zookeeper.session.timeout.ms=18000

# Количество партиций по умолчанию
num.partitions=1

# Replication factor по умолчанию
default.replication.factor=1

# Retention (как долго хранить сообщения)
log.retention.hours=168  # 7 дней
```

**Docker Compose со ZooKeeper:**

YAML

```
version: '3.8'

services:
  # ════════════════════════════════════════════════════════
  #  ZOOKEEPER
  # ════════════════════════════════════════════════════════
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    container_name: zookeeper
    hostname: zookeeper
    ports:
      - "2181:2181"
    environment:
      # ID узла ZooKeeper (для кластера нужны разные ID)
      ZOOKEEPER_SERVER_ID: 1
      
      # Порт для клиентов (Kafka подключается сюда)
      ZOOKEEPER_CLIENT_PORT: 2181
      
      # Базовый временной интервал (мс)
      ZOOKEEPER_TICK_TIME: 2000
      
      # Лимиты для синхронизации
      ZOOKEEPER_INIT_LIMIT: 5
      ZOOKEEPER_SYNC_LIMIT: 2
      
      # Для кластера ZooKeeper (опционально):
      # ZOOKEEPER_SERVERS: zookeeper1:2888:3888;zookeeper2:2888:3888;zookeeper3:2888:3888
    volumes:
      - zookeeper-data:/var/lib/zookeeper/data
      - zookeeper-logs:/var/lib/zookeeper/log
    healthcheck:
      test: ["CMD", "bash", "-c", "echo ruok | nc localhost 2181 | grep imok"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ════════════════════════════════════════════════════════
  #  KAFKA (с ZooKeeper)
  # ════════════════════════════════════════════════════════
  kafka:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka
    hostname: kafka
    depends_on:
      zookeeper:
        condition: service_healthy
    ports:
      - "9092:9092"
      - "9101:9101"
    environment:
      # Уникальный ID брокера
      KAFKA_BROKER_ID: 1
      
      # ★ Адрес ZooKeeper
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      
      # Listener configuration
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://0.0.0.0:9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      
      # Replication settings
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      
      # JMX monitoring
      KAFKA_JMX_PORT: 9101
      KAFKA_JMX_HOSTNAME: localhost
      
      # Consumer group settings
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      
      # Auto create topics
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'
    volumes:
      - kafka-data:/var/lib/kafka/data
    healthcheck:
      test: ["CMD", "kafka-broker-api-versions", "--bootstrap-server", "localhost:9092"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  zookeeper-data:
  zookeeper-logs:
  kafka-data:
```

**Запуск:**

Bash

```
docker-compose up -d

# Проверка:
docker-compose ps

# NAME         IMAGE                              STATUS
# zookeeper    confluentinc/cp-zookeeper:7.5.0    Up (healthy)
# kafka        confluentinc/cp-kafka:7.5.0        Up (healthy)

# ZooKeeper запустится первой (depends_on)
# Kafka подождёт, пока ZooKeeper станет healthy, затем запустится

# Логи:
docker-compose logs -f zookeeper
docker-compose logs -f kafka
```

**Взаимодействие с ZooKeeper (для отладки):**

Bash

```
# Подключиться к ZooKeeper CLI
docker exec -it zookeeper zookeeper-shell localhost:2181

# Внутри ZooKeeper shell:

# Посмотреть корневую структуру
ls /
# [zookeeper, kafka, consumers, brokers, admin, isr_change_notification, controller, ...]

# Посмотреть зарегистрированные брокеры
ls /brokers/ids
# [0]  (или [0, 1, 2] если кластер)

# Информация о брокере 0
get /brokers/ids/0
# {"listener_security_protocol_map":{"PLAINTEXT":"PLAINTEXT"},"endpoints":["PLAINTEXT://localhost:9092"],"jmx_port":9999,"features":{},"host":"localhost","timestamp":"1705318800123","port":9092,"version":5}

# Список топиков
ls /brokers/topics
# [__consumer_offsets, my-topic, orders]

# Информация о партициях топика
get /brokers/topics/my-topic
# {"version":2,"partitions":{"0":[0],"1":[0],"2":[0]},"adding_replicas":{},"removing_replicas":{}}

# Кто controller (координатор кластера)
get /controller
# {"version":1,"brokerid":0,"timestamp":"1705318800456"}

# Выход
quit
```

---

## 2. Основные концепции Kafka — Глубокое погружение

### Producer (Продюсер) — подробное описание

**Producer** — это клиентское приложение или сервис, который **создаёт** и **отправляет** (публикует) сообщения в Kafka. Producer является **активной стороной** — он сам решает, когда и какие данные отправить.

**Философия Producer:**

Представьте Producer как **журналиста**, который пишет статьи (сообщения) и отправляет их в **редакцию** (Kafka топик). Журналист не знает, кто именно будет читать его статьи, сколько читателей будет, и когда они прочитают. Его задача — написать и отправить. Всё остальное — забота редакции (Kafka).

**Анатомия Producer:**

Java

```
// ════════════════════════════════════════════════════════════
// СОЗДАНИЕ PRODUCER
// ════════════════════════════════════════════════════════════

Properties props = new Properties();

// 1. ОБЯЗАТЕЛЬНО: адреса Kafka брокеров (bootstrap servers)
// Producer сначала подключается к любому из этих брокеров,
// получает метаданные о кластере (список всех брокеров, топиков, партиций),
// затем подключается напрямую к нужным брокерам
props.put("bootstrap.servers", "localhost:9092");
// Для production: "broker1:9092,broker2:9092,broker3:9092"

// 2. ОБЯЗАТЕЛЬНО: сериализатор ключа
// Kafka хранит данные в байтах, нужно преобразовать Java-объект → bytes
props.put("key.serializer", 
    "org.apache.kafka.common.serialization.StringSerializer");

// 3. ОБЯЗАТЕЛЬНО: сериализатор значения
props.put("value.serializer", 
    "org.apache.kafka.common.serialization.StringSerializer");

// ════════════════════════════════════════════════════════════
// НАСТРОЙКИ ПРОИЗВОДИТЕЛЬНОСТИ
// ════════════════════════════════════════════════════════════

// Размер батча (Producer накапливает сообщения в памяти, 
// отправляет батчем для эффективности)
props.put("batch.size", 16384);  // 16 KB

// Как долго ждать, прежде чем отправить неполный батч
props.put("linger.ms", 10);  // 10 мс

// Сжатие (уменьшает сетевой трафик и размер на диске)
props.put("compression.type", "snappy");  // snappy, gzip, lz4, zstd

// Размер буфера Producer (для батчинга)
props.put("buffer.memory", 33554432);  // 32 MB

// ════════════════════════════════════════════════════════════
// НАСТРОЙКИ НАДЁЖНОСТИ
// ════════════════════════════════════════════════════════════

// ACKS — сколько подтверждений ждать от брокеров
// "0"   — не ждать (fire and forget, максимальная скорость, могут теряться)
// "1"   — ждать от leader'а (баланс скорость/надёжность)
// "all" — ждать от всех ISR (максимальная надёжность, медленнее)
props.put("acks", "all");

// Сколько попыток повтора при ошибке
props.put("retries", 3);

// Таймаут запроса
props.put("request.timeout.ms", 30000);  // 30 секунд

// ════════════════════════════════════════════════════════════
// ИДЕМПОТЕНТНОСТЬ (exactly-once semantics)
// ════════════════════════════════════════════════════════════

// Producer присваивает каждому сообщению sequence number,
// брокер дедуплицирует повторы
props.put("enable.idempotence", true);

// Создать Producer
KafkaProducer<String, String> producer = new KafkaProducer<>(props);
```

**Отправка сообщений — три способа:**

Java

```
// ════════════════════════════════════════════════════════════
// СПОСОБ 1: Fire-and-Forget (отправил и забыл)
// ════════════════════════════════════════════════════════════

ProducerRecord<String, String> record = new ProducerRecord<>(
    "my-topic",              // топик
    "user123",               // ключ (опционально, может быть null)
    "{\"action\":\"login\"}" // значение
);

producer.send(record);

// Producer поставит сообщение в очередь и вернёт управление СРАЗУ
// НЕ ждёт подтверждения от Kafka
// Подходит для: логи, метрики (некритично, если потеряется)


// ════════════════════════════════════════════════════════════
// СПОСОБ 2: Синхронная отправка (ждём результата)
// ════════════════════════════════════════════════════════════

try {
    // send() возвращает Future
    // get() блокирует поток до получения ответа от Kafka
    RecordMetadata metadata = producer.send(record).get();
    
    System.out.printf("Сообщение успешно отправлено:%n" +
                      "  Topic: %s%n" +
                      "  Partition: %d%n" +
                      "  Offset: %d%n" +
                      "  Timestamp: %d%n",
        metadata.topic(),
        metadata.partition(),
        metadata.offset(),
        metadata.timestamp()
    );
    
} catch (InterruptedException | ExecutionException e) {
    // Ошибки:
    // - TimeoutException (брокер не ответил)
    // - SerializationException (не удалось сериализовать)
    // - RecordTooLargeException (сообщение слишком большое)
    // - NotLeaderForPartitionException (брокер не leader этой партиции)
    e.printStackTrace();
}

// Подходит для: критичные данные (платежи), нужна гарантия записи


// ════════════════════════════════════════════════════════════
// СПОСОБ 3: Асинхронная отправка с callback (наиболее гибкий)
// ════════════════════════════════════════════════════════════

producer.send(record, new Callback() {
    @Override
    public void onCompletion(RecordMetadata metadata, Exception exception) {
        if (exception == null) {
            // Успех!
            System.out.printf("✅ Отправлено: partition=%d, offset=%d%n",
                metadata.partition(), metadata.offset());
        } else {
            // Ошибка
            System.err.println("❌ Ошибка отправки: " + exception.getMessage());
            
            // Можно реализовать custom retry логику,
            // отправку в DLQ (dead letter queue),
            // алерты и т.д.
        }
    }
});

// Producer НЕ блокируется, callback вызовется когда придёт ответ
// Подходит для: большинство сценариев (баланс производительность/надёжность)
```

**Партиционирование — как Producer выбирает партицию:**

Java

```
// ════════════════════════════════════════════════════════════
// ВАРИАНТ 1: Явное указание партиции
// ════════════════════════════════════════════════════════════

ProducerRecord<String, String> record = new ProducerRecord<>(
    "my-topic",   // топик
    2,            // партиция (явно указываем — партиция #2)
    "user123",    // ключ (игнорируется при явном указании партиции)
    "value"       // значение
);

// Сообщение ГАРАНТИРОВАННО пойдёт в партицию 2


// ════════════════════════════════════════════════════════════
// ВАРИАНТ 2: Партиционирование по ключу (рекомендуется!)
// ════════════════════════════════════════════════════════════

// Если указан ключ, Producer вычисляет:
// partition = hash(key) % number_of_partitions

ProducerRecord<String, String> record1 = new ProducerRecord<>(
    "user-events",
    "user123",     // ключ
    "{\"action\":\"login\"}"
);

ProducerRecord<String, String> record2 = new ProducerRecord<>(
    "user-events",
    "user123",     // ТОТ ЖЕ ключ
    "{\"action\":\"purchase\"}"
);

ProducerRecord<String, String> record3 = new ProducerRecord<>(
    "user-events",
    "user456",     // ДРУГОЙ ключ
    "{\"action\":\"view\"}"
);

producer.send(record1);  // → партиция, скажем, 1
producer.send(record2);  // → партиция 1 (тот же ключ!)
producer.send(record3);  // → партиция, скажем, 0 (другой ключ)

// ★ ВСЕ сообщения с ключом "user123" попадут в ОДНУ партицию
// → порядок сохранён!
// → можно проследить историю действий пользователя


// ════════════════════════════════════════════════════════════
// ВАРИАНТ 3: Round-Robin (ключ отсутствует)
// ════════════════════════════════════════════════════════════

ProducerRecord<String, String> record = new ProducerRecord<>(
    "logs",
    null,          // ключ отсутствует
    "Log message"
);

// Producer распределяет сообщения равномерно по партициям:
// msg1 → partition 0
// msg2 → partition 1
// msg3 → partition 2
// msg4 → partition 0
// ...

// Подходит для: логи, метрики (порядок не важен, важна равномерность)


// ════════════════════════════════════════════════════════════
// ВАРИАНТ 4: Custom Partitioner (своя логика)
// ════════════════════════════════════════════════════════════

// Можно реализовать свой Partitioner:
public class CustomPartitioner implements Partitioner {
    @Override
    public int partition(String topic, Object key, byte[] keyBytes,
                        Object value, byte[] valueBytes,
                        Cluster cluster) {
        
        // Пример: VIP-пользователи → специальная партиция
        if (key != null && key.toString().startsWith("vip-")) {
            return 0;  // VIP партиция
        }
        
        // Обычные пользователи → остальные партиции
        int numPartitions = cluster.partitionCountForTopic(topic);
        return Utils.toPositive(Utils.murmur2(keyBytes)) % (numPartitions - 1) + 1;
    }
    
    @Override
    public void close() {}
    
    @Override
    public void configure(Map<String, ?> configs) {}
}

// Подключить Custom Partitioner:
props.put("partitioner.class", "com.example.CustomPartitioner");
```

**Закрытие Producer:**

Java

```
// ════════════════════════════════════════════════════════════
// ВАЖНО: ВСЕГДА закрывать Producer при завершении
// ════════════════════════════════════════════════════════════

// Закрыть Producer (дождётся отправки всех сообщений в буфере)
producer.close();

// Или с таймаутом:
producer.close(Duration.ofSeconds(10));

// Try-with-resources (рекомендуется):
try (KafkaProducer<String, String> producer = new KafkaProducer<>(props)) {
    producer.send(record);
    // Producer автоматически закроется
}
```

**Продвинутые паттерны Producer:**

Java

```
// ════════════════════════════════════════════════════════════
// ТРАНЗАКЦИИ (Exactly-Once Semantics)
// ════════════════════════════════════════════════════════════

Properties props = new Properties();
// ... базовые настройки ...

// Включить транзакции
props.put("transactional.id", "my-transactional-producer-1");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);

// Инициализировать транзакции
producer.initTransactions();

try {
    // Начать транзакцию
    producer.beginTransaction();
    
    // Отправить несколько сообщений
    producer.send(record1);
    producer.send(record2);
    producer.send(record3);
    
    // Все сообщения отправлены АТОМАРНО —
    // либо ВСЕ видны consumer'у, либо НИЧЕГО
    
    // Закоммитить транзакцию
    producer.commitTransaction();
    
} catch (Exception e) {
    // Откатить транзакцию при ошибке
    producer.abortTransaction();
}

producer.close();
```

---

### Consumer (Консьюмер) — подробное описание

**Consumer** — это клиентское приложение, которое **читает** и **обрабатывает** сообщения из Kafka топиков. В отличие от традиционных message queues (где брокер **push**'ит сообщения клиенту), в Kafka consumer **сам** забирает сообщения (**pull**-модель).

**Философия Consumer:**

Consumer как **читатель новостной ленты** (RSS feed). Лента постоянно обновляется новыми статьями, но **вы сами** решаете:

- Когда проверять обновления (как часто делать poll)
- Откуда начать читать (с самого начала, с конца, с определённой позиции)
- Можете ли перечитать старые статьи (replay)
- Можете остановиться, а потом продолжить с того же места

**Анатомия Consumer:**

Java

```
// ════════════════════════════════════════════════════════════
// СОЗДАНИЕ CONSUMER
// ════════════════════════════════════════════════════════════

Properties props = new Properties();

// 1. ОБЯЗАТЕЛЬНО: адреса брокеров
props.put("bootstrap.servers", "localhost:9092");

// 2. ОБЯЗАТЕЛЬНО: ID consumer group
// Consumers с одинаковым group.id работают ВМЕСТЕ,
// делят партиции между собой
props.put("group.id", "my-consumer-group");

// 3. ОБЯЗАТЕЛЬНО: десериализаторы
props.put("key.deserializer",
    "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer",
    "org.apache.kafka.common.serialization.StringDeserializer");

// ════════════════════════════════════════════════════════════
// УПРАВЛЕНИЕ OFFSET
// ════════════════════════════════════════════════════════════

// Автоматический коммит offset (Consumer периодически сохраняет позицию)
props.put("enable.auto.commit", "true");
props.put("auto.commit.interval.ms", "5000");  // каждые 5 секунд

// Что делать, если offset отсутствует (новая consumer group):
// "earliest" — читать с самого начала топика
// "latest"   — читать только новые сообщения (пропустить историю)
// "none"     — выбросить ошибку
props.put("auto.offset.reset", "earliest");

// ════════════════════════════════════════════════════════════
// ПРОИЗВОДИТЕЛЬНОСТЬ
// ════════════════════════════════════════════════════════════

// Максимальное кол-во записей в одном poll()
props.put("max.poll.records", 500);

// Минимальный размер данных для fetch (Consumer ждёт накопления)
props.put("fetch.min.bytes", 1);

// Максимальное время ожидания для fetch
props.put("fetch.max.wait.ms", 500);

// Heartbeat (Consumer сообщает брокеру "я жив")
props.put("heartbeat.interval.ms", 3000);

// Таймаут сессии (если брокер не получал heartbeat, считает consumer мёртвым)
props.put("session.timeout.ms", 10000);

// Максимальное время между poll() (если превысить — rebalance)
props.put("max.poll.interval.ms", 300000);  // 5 минут

// Создать Consumer
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
```

**Подписка на топики и чтение сообщений:**

Java

```
// ════════════════════════════════════════════════════════════
// ПОДПИСКА НА ТОПИКИ
// ════════════════════════════════════════════════════════════

// Способ 1: Подписка на конкретные топики
consumer.subscribe(Arrays.asList("topic1", "topic2", "topic3"));

// Способ 2: Подписка по паттерну (regex)
consumer.subscribe(Pattern.compile("events-.*"));
// Подпишется на: events-orders, events-payments, events-users, ...

// Способ 3: Ручное назначение партиций (без consumer group)
TopicPartition partition0 = new TopicPartition("my-topic", 0);
TopicPartition partition1 = new TopicPartition("my-topic", 1);
consumer.assign(Arrays.asList(partition0, partition1));

// При assign() Consumer НЕ участвует в consumer group,
// НЕ будет rebalancing


// ════════════════════════════════════════════════════════════
// ОСНОВНОЙ ЦИКЛ ЧТЕНИЯ
// ════════════════════════════════════════════════════════════

try {
    while (true) {
        // Poll — забрать сообщения (блокируется до 100 мс)
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
        
        // records может содержать сообщения из РАЗНЫХ партиций!
        
        for (ConsumerRecord<String, String> record : records) {
            System.out.printf(
                "Получено сообщение:%n" +
                "  Topic:     %s%n" +
                "  Partition: %d%n" +
                "  Offset:    %d%n" +
                "  Key:       %s%n" +
                "  Value:     %s%n" +
                "  Timestamp: %d%n",
                record.topic(),
                record.partition(),
                record.offset(),
                record.key(),
                record.value(),
                record.timestamp()
            );
            
            // Обработка сообщения
            processMessage(record);
        }
        
        // Если enable.auto.commit=true, offset автоматически закоммитится
    }
    
} finally {
    // ВАЖНО: закрыть Consumer при завершении
    consumer.close();
}
```

**Ручное управление offset (для точного контроля):**

Java

```
// ════════════════════════════════════════════════════════════
// РУЧНОЙ КОММИТ OFFSET
// ════════════════════════════════════════════════════════════

Properties props = new Properties();
// ... базовые настройки ...

// ОТКЛЮЧИТЬ автокоммит
props.put("enable.auto.commit", "false");

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("my-topic"));

try {
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
        
        for (ConsumerRecord<String, String> record : records) {
            try {
                // Обработать сообщение
                processMessage(record);
                
                // Сохранить в базу данных
                saveToDatabase(record);
                
                // ПОСЛЕ успешной обработки — коммитим offset
                // Синхронный коммит (блокирует поток до подтверждения)
                consumer.commitSync();
                
            } catch (Exception e) {
                // Если обработка упала — НЕ коммитим offset
                // При следующем запуске Consumer перечитает это сообщение
                System.err.println("Ошибка обработки, offset НЕ закоммичен");
            }
        }
    }
} finally {
    consumer.close();
}


// ════════════════════════════════════════════════════════════
// АСИНХРОННЫЙ КОММИТ (неблокирующий)
// ════════════════════════════════════════════════════════════

consumer.commitAsync(new OffsetCommitCallback() {
    @Override
    public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets,
                          Exception exception) {
        if (exception != null) {
            System.err.println("Ошибка коммита offset: " + exception.getMessage());
        } else {
            System.out.println("Offset успешно закоммичен: " + offsets);
        }
    }
});


// ════════════════════════════════════════════════════════════
// КОММИТ КОНКРЕТНОГО OFFSET
// ════════════════════════════════════════════════════════════

for (ConsumerRecord<String, String> record : records) {
    processMessage(record);
    
    // Коммитим offset ЭТОГО конкретного сообщения
    Map<TopicPartition, OffsetAndMetadata> offsets = new HashMap<>();
    offsets.put(
        new TopicPartition(record.topic(), record.partition()),
        new OffsetAndMetadata(record.offset() + 1)  // +1 потому что коммитим СЛЕДУЮЩИЙ offset
    );
    
    consumer.commitSync(offsets);
}
```

**Перемотка offset (replay сообщений):**

Java

```
// ════════════════════════════════════════════════════════════
// SEEK — перемотать offset на конкретную позицию
// ════════════════════════════════════════════════════════════

consumer.subscribe(Arrays.asList("my-topic"));

// Дождаться назначения партиций
consumer.poll(Duration.ofMillis(0));

// Получить назначенные партиции
Set<TopicPartition> assignedPartitions = consumer.assignment();

// Перемотать на конкретный offset
for (TopicPartition partition : assignedPartitions) {
    consumer.seek(partition, 100);  // читать с offset=100
}

// Теперь poll() вернёт сообщения начиная с offset=100


// ════════════════════════════════════════════════════════════
// SEEK TO BEGINNING — начать с самого начала
// ════════════════════════════════════════════════════════════

consumer.seekToBeginning(assignedPartitions);
// Перечитать ВСЮ историю


// ════════════════════════════════════════════════════════════
// SEEK TO END — пропустить все старые, читать только новые
// ════════════════════════════════════════════════════════════

consumer.seekToEnd(assignedPartitions);


// ════════════════════════════════════════════════════════════
// SEEK по timestamp — найти offset по времени
// ════════════════════════════════════════════════════════════

long timestamp = System.currentTimeMillis() - 86400000;  // 24 часа назад

Map<TopicPartition, Long> timestampsToSearch = new HashMap<>();
for (TopicPartition partition : assignedPartitions) {
    timestampsToSearch.put(partition, timestamp);
}

// Kafka вернёт offset'ы, соответствующие этому времени
Map<TopicPartition, OffsetAndMetadata> offsets = 
    consumer.offsetsForTimes(timestampsToSearch);

for (Map.Entry<TopicPartition, OffsetAndMetadata> entry : offsets.entrySet()) {
    consumer.seek(entry.getKey(), entry.getValue().offset());
}

// Теперь читаем сообщения за последние 24 часа
```

---

### Consumer Group — детальное объяснение

**Consumer Group** — это мощный механизм Kafka для **горизонтального масштабирования** обработки сообщений. Несколько Consumer'ов объединяются в группу, и Kafka **автоматически распределяет** партиции между ними.

**Ключевые принципы:**

1. **Одна партиция → ровно ОДИН consumer в группе** (в момент времени)
2. **Один consumer может обрабатывать НЕСКОЛЬКО партиций**
3. **Разные consumer groups читают НЕЗАВИСИМО**

**Визуализация:**

text

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Topic: "orders" (6 партиций)                      │
│                                                                     │
│  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐        │
│  │ P0   │  │ P1   │  │ P2   │  │ P3   │  │ P4   │  │ P5   │        │
│  └───┬──┘  └───┬──┘  └───┬──┘  └───┬──┘  └───┬──┘  └───┬──┘        │
│      │         │         │         │         │         │           │
└──────┼─────────┼─────────┼─────────┼─────────┼─────────┼───────────┘
       │         │         │         │         │         │
       │         │         │         │         │         │
┌──────┼─────────┼─────────┼─────────┼─────────┼─────────┼───────────┐
│      │         │         │         │         │         │           │
│      ▼         ▼         ▼         ▼         ▼         ▼           │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐      │
│  │   Consumer A    │ │   Consumer B    │ │   Consumer C    │      │
│  │  P0, P1         │ │  P2, P3         │ │  P4, P5         │      │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘      │
│                                                                     │
│          Consumer Group: "order-processing-service"                 │
│          • 3 consumer'а делят 6 партиций                            │
│          • Каждый consumer — по 2 партиции                          │
│          • Параллельная обработка ✅                                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Сценарии распределения:**

text

```
╔═══════════════════════════════════════════════════════════════════╗
║                  СЦЕНАРИЙ 1: Идеальный баланс                     ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  6 партиций, 3 consumer'а → каждому по 2 партиции                 ║
║                                                                   ║
║  Consumer A: [P0, P1]                                             ║
║  Consumer B: [P2, P3]                                             ║
║  Consumer C: [P4, P5]                                             ║
║                                                                   ║
║  ✅ Равномерная нагрузка                                          ║
║  ✅ Максимальный параллелизм                                       ║
║                                                                   ║
╠═══════════════════════════════════════════════════════════════════╣
║                  СЦЕНАРИЙ 2: Consumers меньше партиций            ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  6 партиций, 2 consumer'а → неравномерно                          ║
║                                                                   ║
║  Consumer A: [P0, P1, P2]                                         ║
║  Consumer B: [P3, P4, P5]                                         ║
║                                                                   ║
║  ✅ Всё работает                                                  ║
║  ⚠️  Consumer'ы перегружены (каждый по 3 партиции)                ║
║  💡 Решение: добавить ещё consumer'ов                             ║
║                                                                   ║
╠═══════════════════════════════════════════════════════════════════╣
║                  СЦЕНАРИЙ 3: Consumers БОЛЬШЕ партиций            ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  3 партиции, 5 consumer'ов → не все заняты                        ║
║                                                                   ║
║  Consumer A: [P0]                                                 ║
║  Consumer B: [P1]                                                 ║
║  Consumer C: [P2]                                                 ║
║  Consumer D: []        ← IDLE (ждёт)                              ║
║  Consumer E: []        ← IDLE (ждёт)                              ║
║                                                                   ║
║  ⚠️  2 consumer'а простаивают                                     ║
║  ✅ НО: если Consumer A упадёт → Consumer D сразу подхватит P0    ║
║  💡 Это резерв для отказоустойчивости                             ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝
```

**Независимость consumer groups:**

text

```
┌──────────────────────────────────────────────────────────────────┐
│              Topic: "user-events" (3 партиции)                   │
│                                                                  │
│     P0: [msg0, msg1, msg2, msg3, msg4, msg5, msg6, ...]          │
│     P1: [msg0, msg1, msg2, msg3, msg4, msg5, msg6, ...]          │
│     P2: [msg0, msg1, msg2, msg3, msg4, msg5, msg6, ...]          │
│                                                                  │
└───────────┬───────────────────────┬──────────────────────────────┘
            │                       │
            │                       │
    ┌───────▼────────┐      ┌───────▼────────┐
    │ Consumer Group │      │ Consumer Group │
    │  "analytics"   │      │  "billing"     │
    │                │      │                │
    │ offset(P0)=500 │      │ offset(P0)=200 │
    │ offset(P1)=600 │      │ offset(P1)=150 │
    │ offset(P2)=550 │      │ offset(P2)=180 │
    └────────────────┘      └────────────────┘
         читает                  читает
      быстро (lag=0)          медленно (lag=300)

• Группа "analytics" прочитала до offset=500 в P0
• Группа "billing" только до offset=200 в P0
• Они НЕ ВЛИЯЮТ друг на друга
• Каждая группа хранит СВОЙ offset в топике __consumer_offsets
```

**Rebalancing (перебалансировка) — как это работает:**

Rebalancing — это процесс **перераспределения партиций** между consumer'ами в группе. Происходит когда:

- Consumer добавляется в группу
- Consumer выходит из группы (graceful shutdown или crash)
- Создаются новые партиции в топике
- Consumer не отправляет heartbeat (считается мёртвым)

text

```
═══════════════════════════════════════════════════════════════════
                    ПРИМЕР REBALANCING
═══════════════════════════════════════════════════════════════════

НАЧАЛЬНОЕ СОСТОЯНИЕ: 4 партиции, 2 consumer'а

  Consumer A: [P0, P1]
  Consumer B: [P2, P3]

───────────────────────────────────────────────────────────────────

СОБЫТИЕ: Consumer C ПРИСОЕДИНИЛСЯ к группе

  1. Consumer C отправляет JoinGroup запрос координатору
  2. Координатор фиксирует: "новый участник!"
  3. Координатор инициирует rebalancing:
     - ВСЕ consumer'ы ОСТАНАВЛИВАЮТ чтение
     - Коммитят текущие offset'ы
     - Отписываются от партиций
  
  4. Координатор запускает partition assignment алгоритм
     (по умолчанию — RangeAssignor или RoundRobinAssignor)
  
  5. НОВОЕ РАСПРЕДЕЛЕНИЕ:
     Consumer A: [P0]
     Consumer B: [P1, P2]
     Consumer C: [P3]
  
  6. ВСЕ consumer'ы получают новые назначения
  7. Consumer'ы подписываются на новые партиции
  8. Возобновляют чтение

───────────────────────────────────────────────────────────────────

СОБЫТИЕ: Consumer B УПАЛ (crash)

  1. Координатор не получает heartbeat от Consumer B
  2. Через session.timeout.ms (default 10 сек) считает B мёртвым
  3. Инициирует rebalancing
  
  4. НОВОЕ РАСПРЕДЕЛЕНИЕ (2 живых consumer'а):
     Consumer A: [P0, P1]
     Consumer C: [P2, P3]
  
  5. Партиции P1 и P2, которые были у B, переданы A и C

───────────────────────────────────────────────────────────────────

⚠️  ПРОБЛЕМА: Во время rebalancing consumer'ы НЕ обрабатывают сообщения!
    Это "stop-the-world" пауза.

💡 РЕШЕНИЯ:
   • Минимизировать частоту rebalancing (настроить таймауты)
   • Использовать Sticky Assignor (минимизирует перемещение партиций)
   • Incremental Cooperative Rebalancing (Kafka 2.4+) — перебалансировка
     БЕЗ остановки ВСЕХ consumer'ов

═══════════════════════════════════════════════════════════════════
```

**Static Group Membership (Kafka 2.3+):**

Традиционно, когда consumer перезапускается, он получает **новый** member.id → rebalancing. Static membership позволяет **сохранить** member.id → **избежать** rebalancing при рестартах.

Java

```
Properties props = new Properties();
// ... базовые настройки ...

// Установить статический ID
props.put("group.instance.id", "consumer-1-static-id");

// Теперь при рестарте consumer'а:
// 1. Kafka видит: "Это тот же consumer с ID consumer-1-static-id"
// 2. Возвращает ему ТЕ ЖЕ партиции
// 3. Rebalancing НЕ происходит ✅
// 4. Другие consumer'ы НЕ останавливаются

// Подходит для: stateful обработчиков (с локальным кешем)
```

---

### Topic (Топик) — подробное описание

**Topic** — это **логическая категория** или **канал**, куда публикуются сообщения. Это центральная абстракция Kafka. Топик можно представить как **таблицу в БД** или **поток событий** определённого типа.

**Характеристики топика:**

text

```
╔═══════════════════════════════════════════════════════════════════╗
║                      ЧТО ТАКОЕ ТОПИК?                             ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  • Именованный поток событий                                      ║
║  • Неизменяемый лог (append-only)                                 ║
║  • Распределён по партициям                                       ║
║  • Реплицирован для отказоустойчивости                            ║
║  • Хранится определённое время (retention)                        ║
║  • Может быть сжат (compacted)                                    ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝
```

**Именование топиков — Best Practices:**

text

```
✅ ХОРОШИЕ ИМЕНА:

  user-events               ← события пользователей
  payment-transactions      ← финансовые транзакции
  inventory-updates         ← обновления инвентаря
  logs.application.errors   ← иерархия через точки
  prod.orders.v2            ← окружение + версия


❌ ПЛОХИЕ ИМЕНА:

  topic1                    ← непонятно, что внутри
  UserEvents                ← CamelCase (рекомендуется kebab-case)
  events                    ← слишком общее
  orders_2024_01_15         ← дата в имени (лучше partition по времени)
```

**Создание топика:**

Bash

```
# Создать топик через CLI
bin/kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --topic user-events \
  --partitions 6 \
  --replication-factor 3 \
  --config retention.ms=604800000 \
  --config compression.type=snappy \
  --config min.insync.replicas=2

# Параметры:
# --partitions 6              → 6 партиций (параллелизм)
# --replication-factor 3      → 3 копии каждой партиции
# --config retention.ms       → хранить 7 дней (604800000 мс)
# --config compression.type   → сжимать snappy
# --config min.insync.replicas → минимум 2 ISR для записи
```

**Конфигурация топика:**

Bash

```
# Список топиков
bin/kafka-topics.sh --list \
  --bootstrap-server localhost:9092

# Детальная информация о топике
bin/kafka-topics.sh --describe \
  --bootstrap-server localhost:9092 \
  --topic user-events

# Вывод:
# Topic: user-events   PartitionCount: 6   ReplicationFactor: 3   Configs: retention.ms=604800000
#   Topic: user-events   Partition: 0   Leader: 1   Replicas: 1,2,3   Isr: 1,2,3
#   Topic: user-events   Partition: 1   Leader: 2   Replicas: 2,3,1   Isr: 2,3,1
#   ...
```

**Retention Policy — как долго хранить:**

Kafka может удалять старые сообщения двумя способами:

Java

```
// ═══════════════════════════════════════════════════════════
// СПОСОБ 1: TIME-BASED RETENTION (по времени)
// ═══════════════════════════════════════════════════════════

// Хранить 7 дней
retention.ms=604800000

// Или через hours:
retention.hours=168

// Kafka проверяет каждые 5 минут (по умолчанию),
// удаляет сегменты старше retention.ms


// ═══════════════════════════════════════════════════════════
// СПОСОБ 2: SIZE-BASED RETENTION (по размеру)
// ═══════════════════════════════════════════════════════════

// Максимальный размер партиции — 10 GB
retention.bytes=10737418240

// Когда размер превышен, удаляются СТАРЫЕ сегменты


// ═══════════════════════════════════════════════════════════
// КОМБИНАЦИЯ: действует ТО, что наступит РАНЬШЕ
// ═══════════════════════════════════════════════════════════

retention.ms=604800000      # 7 дней
retention.bytes=10737418240 # 10 GB

// Если за 3 дня накопилось 12 GB → удалится (size limit)
// Если за 7 дней накопилось 5 GB → удалится (time limit)
```

**Log Compaction — умное хранение:**

Для некоторых use case'ов нужно хранить только **последнее** значение для каждого ключа (например, текущее состояние пользователя).

Bash

```
# Включить compaction
bin/kafka-configs.sh --alter \
  --bootstrap-server localhost:9092 \
  --topic user-profile \
  --add-config cleanup.policy=compact

# Теперь Kafka хранит:
# user123: {name: "Alice", age: 30}  ← последнее значение
# user456: {name: "Bob", age: 25}    ← последнее значение

# Старые значения для user123 УДАЛЕНЫ (compaction)
```

**Пример:**

text

```
ДО COMPACTION:

offset  key      value
0       user123  {name: "Alice", age: 25}
1       user456  {name: "Bob", age: 20}
2       user123  {name: "Alice", age: 26}   ← обновление
3       user789  {name: "Charlie", age: 30}
4       user123  {name: "Alice", age: 30}   ← ещё обновление
5       user456  {name: "Bob", age: 25}     ← обновление

─────────────────────────────────────────────────────

ПОСЛЕ COMPACTION:

offset  key      value
4       user123  {name: "Alice", age: 30}   ← ПОСЛЕДНЕЕ для user123
5       user456  {name: "Bob", age: 25}     ← ПОСЛЕДНЕЕ для user456
3       user789  {name: "Charlie", age: 30} ← единственное для user789

Старые записи (offset 0, 1, 2) УДАЛЕНЫ
```

---

### Partition (Партиция) — глубокое погружение

**Partition** — это **физическая единица хранения** и **единица параллелизма** в Kafka. Каждая партиция — это **упорядоченный, неизменяемый лог** сообщений.

**Почему партиции критичны:**

1. **Параллелизм** — можно читать/писать в разные партиции одновременно
2. **Масштабируемость** — партиции распределены по разным серверам
3. **Упорядоченность** — внутри партиции порядок ГАРАНТИРОВАН
4. **Отказоустойчивость** — каждая партиция реплицируется

**Анатомия партиции:**

text

```
┌────────────────────────────────────────────────────────────────────┐
│                  ПАРТИЦИЯ — это лог на диске                       │
│                                                                    │
│  Директория: /var/lib/kafka/data/topic-name-0/                    │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  Segment 0 (00000000000000000000.log)                        │  │
│  │  offset 0-999                                                │  │
│  │  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐    │  │
│  │  │ 0   │ 1   │ 2   │ 3   │ 4   │ ... │ 998 │ 999 │     │    │  │
│  │  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘    │  │
│  │  Размер: 1 GB (по умолчанию)                                │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  Segment 1 (00000000000000001000.log)                        │  │
│  │  offset 1000-1999                                            │  │
│  │  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐    │  │
│  │  │1000 │1001 │1002 │1003 │1004 │ ... │1998 │1999 │     │    │  │
│  │  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘    │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  Segment 2 (active) — текущий сегмент, куда пишут            │  │
│  │  offset 2000-...                                             │  │
│  │  ┌─────┬─────┬─────┬─────┬                                  │  │
│  │  │2000 │2001 │2002 │2003 │ ← новые сообщения добавляются    │  │
│  │  └─────┴─────┴─────┴─────┴                                  │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  Файлы:                                                            │
│  • 00000000000000000000.log       ← данные                         │
│  • 00000000000000000000.index     ← индекс (offset → position)     │
│  • 00000000000000000000.timeindex ← индекс (timestamp → offset)    │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Как Kafka пишет в партицию:**

text

```
1. Producer отправляет сообщение
2. Broker (leader партиции) получает сообщение
3. Присваивает OFFSET (уникальный номер в партиции)
4. Добавляет в АКТИВНЫЙ сегмент (append в конец файла)
5. Синхронизирует с follower'ами (репликация)
6. Возвращает ACK producer'у
```

**Offset — подробно:**

text

```
═══════════════════════════════════════════════════════════════════
                        ЧТО ТАКОЕ OFFSET?
═══════════════════════════════════════════════════════════════════

Offset — это МОНОТОННО ВОЗРАСТАЮЩИЙ числовой идентификатор сообщения
ВНУТРИ партиции.

Partition 0:
  offset 0: {"user": "alice", "action": "login"}
  offset 1: {"user": "bob", "action": "view"}
  offset 2: {"user": "alice", "action": "purchase"}
  offset 3: {"user": "charlie", "action": "logout"}
  ...

Partition 1:
  offset 0: {"user": "dave", "action": "login"}   ← ДРУГОЙ offset 0!
  offset 1: {"user": "eve", "action": "view"}
  ...

ВАЖНО:
  • Offset уникален ТОЛЬКО В РАМКАХ партиции
  • Разные партиции имеют НЕЗАВИСИМЫЕ offset'ы
  • Offset НИКОГДА не переиспользуется (даже после удаления сообщения)

═══════════════════════════════════════════════════════════════════
                   СПЕЦИАЛЬНЫЕ OFFSET'Ы
═══════════════════════════════════════════════════════════════════

Earliest offset (beginning):
  • Самое старое доступное сообщение в партиции
  • Может быть НЕ 0 (если старые сообщения удалены по retention)

Latest offset (end):
  • Следующий offset, который будет записан
  • = offset последнего сообщения + 1

High Water Mark (HWM):
  • Максимальный offset, который РЕПЛИЦИРОВАН на все ISR
  • Consumer'ы видят ТОЛЬКО сообщения до HWM
  • Гарантирует, что consumer не прочитает сообщение,
    которое может потеряться при падении leader'а

Log End Offset (LEO):
  • Offset ПОСЛЕДНЕГО сообщения в логе (включая нереплицированные)
  • Может быть > HWM

═══════════════════════════════════════════════════════════════════
```

**Replication — репликация партиций:**

text

```
┌────────────────────────────────────────────────────────────────────┐
│         Partition 0 с replication factor = 3                       │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  Broker 1 — LEADER                                           │  │
│  │  ┌─────────────────────────────────────────────────────────┐ │  │
│  │  │ offset 0  1  2  3  4  5  6  7  8  9                     │ │  │
│  │  │       [A][B][C][D][E][F][G][H][I][J]                    │ │  │
│  │  └─────────────────────────────────────────────────────────┘ │  │
│  │         ▲                                                    │  │
│  │         │ Producer пишет ТОЛЬКО в Leader                    │  │
│  │         │ Consumer читает ТОЛЬКО из Leader (по умолчанию)   │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│           │ Репликация (синхронизация)                             │
│           │                                                        │
│           ▼                                                        │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  Broker 2 — FOLLOWER                                         │  │
│  │  ┌─────────────────────────────────────────────────────────┐ │  │
│  │  │ offset 0  1  2  3  4  5  6  7  8  9                     │ │  │
│  │  │       [A][B][C][D][E][F][G][H][I][J]                    │ │  │
│  │  └─────────────────────────────────────────────────────────┘ │  │
│  │  In-Sync Replica ✅  (успевает за Leader)                   │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  Broker 3 — FOLLOWER                                         │  │
│  │  ┌─────────────────────────────────────────────────────────┐ │  │
│  │  │ offset 0  1  2  3  4  5  6  7                           │ │  │
│  │  │       [A][B][C][D][E][F][G][H]                          │ │  │
│  │  └─────────────────────────────────────────────────────────┘ │  │
│  │  Out-of-Sync Replica ⚠️  (отстаёт от Leader)                │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ISR (In-Sync Replicas) = {Broker 1, Broker 2}                     │
│                                                                    │
│  Если Broker 1 упадёт:                                             │
│    → Broker 2 станет новым Leader (т.к. in-sync)                   │
│    → Нет потери данных ✅                                          │
│                                                                    │
│  Если Broker 3 догонит (offset 9):                                 │
│    → Broker 3 вернётся в ISR                                       │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Leader Election — выбор нового лидера:**

text

```
═══════════════════════════════════════════════════════════════════
                    СЦЕНАРИЙ: Leader упал
═══════════════════════════════════════════════════════════════════

ДО:
  Leader:    Broker 1
  ISR:       {Broker 1, Broker 2, Broker 3}
  
Broker 1 ПАДАЕТ (hardware failure, restart, network)

1. ZooKeeper / Controller обнаруживает: Leader недоступен
2. Controller выбирает НОВОГО Leader'а из ISR
   • Кандидаты: {Broker 2, Broker 3}
   • Выбирается первый в списке: Broker 2
   
3. НОВОЕ состояние:
   Leader:    Broker 2
   ISR:       {Broker 2, Broker 3}
   
4. Broker 2 начинает принимать запросы на чтение/запись
5. Broker 3 синхронизируется с новым Leader'ом
6. Producer'ы и Consumer'ы автоматически переподключаются к Broker 2

Время failover: обычно < 10 секунд

───────────────────────────────────────────────────────────────────

⚠️  ВАЖНО: Если ВСЕ ISR упали, есть выбор:

  Option 1: Ждать, пока вернётся ISR (безопасно, но долго)
  Option 2: Выбрать Leader из out-of-sync реплик (риск потери данных)
  
  Настройка:
  unclean.leader.election.enable=false  ← НЕ выбирать out-of-sync (по умолчанию)
  unclean.leader.election.enable=true   ← Доступность важнее консистентности

═══════════════════════════════════════════════════════════════════
```
Max

## 4. ZooKeeper — Исчерпывающее описание

### Что такое ZooKeeper?

Apache ZooKeeper — это распределённый **координационный сервис**, изначально разработанный в Yahoo! для управления большими кластерами распределённых систем. Его можно представить как **«центральный реестр»** или **«нотариуса»** для распределённых приложений — он хранит конфигурацию, обеспечивает синхронизацию, отслеживает состояние узлов и помогает выбирать лидеров.

**Аналогия из реальной жизни:**

Представьте **крупную компанию с несколькими офисами** (брокерами). ZooKeeper — это **главный офис / секретариат**, который:

- Знает адреса всех офисов (service discovery)
- Хранит корпоративную политику и правила (configuration management)
- Решает, кто будет руководить каждым проектом, если текущий руководитель уволился (leader election)
- Ведёт реестр всех проектов и документов (metadata storage)
- Отслеживает, какие офисы работают, а какие закрылись (health monitoring)

**Архитектура ZooKeeper:**

text

```
╔══════════════════════════════════════════════════════════════════╗
║                    ZOOKEEPER ENSEMBLE (КЛАСТЕР)                  ║
║                                                                  ║
║  ZooKeeper всегда работает как КЛАСТЕР из нечётного числа нод   ║
║  (обычно 3 или 5). Нечётное число нужно для голосования.        ║
║                                                                  ║
║  ┌───────────────┐   ┌───────────────┐   ┌───────────────┐      ║
║  │  ZK Node 1    │   │  ZK Node 2    │   │  ZK Node 3    │      ║
║  │  (LEADER)     │   │  (FOLLOWER)   │   │  (FOLLOWER)   │      ║
║  │               │   │               │   │               │      ║
║  │  Обрабатывает │   │  Обрабатывает │   │  Обрабатывает │      ║
║  │  ЗАПИСЬ       │   │  ЧТЕНИЕ       │   │  ЧТЕНИЕ       │      ║
║  │  + чтение     │   │               │   │               │      ║
║  └───────┬───────┘   └───────┬───────┘   └───────┬───────┘      ║
║          │                   │                   │              ║
║          └───────────────────┼───────────────────┘              ║
║                              │                                  ║
║                    Репликация данных                             ║
║                    (все ноды имеют                               ║
║                     полную копию данных)                         ║
║                                                                  ║
║  КВОРУМ = majority (большинство):                                ║
║  • 3 ноды → кворум = 2 (переживает падение 1 ноды)              ║
║  • 5 нод  → кворум = 3 (переживает падение 2 нод)              ║
║  • 7 нод  → кворум = 4 (переживает падение 3 нод)              ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

### Что именно ZooKeeper хранит для Kafka?

ZooKeeper является **хранилищем метаданных** и **координатором** для кластера Kafka. Внутри ZooKeeper данные организованы в виде **иерархического дерева** (подобно файловой системе), где каждый узел дерева называется **ZNode** и может хранить данные и иметь дочерние узлы.

text

```
/                                          ← корень
├── /brokers                               ← информация о брокерах
│   ├── /brokers/ids                       ← список живых брокеров
│   │   ├── /brokers/ids/0                 ← данные брокера 0 (хост, порт, версия)
│   │   ├── /brokers/ids/1                 ← данные брокера 1
│   │   └── /brokers/ids/2                 ← данные брокера 2
│   │
│   ├── /brokers/topics                    ← метаданные топиков
│   │   ├── /brokers/topics/orders         ← информация о топике "orders"
│   │   │   └── /brokers/topics/orders/partitions
│   │   │       ├── /0                     ← партиция 0 (leader, ISR, replicas)
│   │   │       ├── /1                     ← партиция 1
│   │   │       └── /2                     ← партиция 2
│   │   │
│   │   └── /brokers/topics/payments       ← топик "payments"
│   │       └── ...
│   │
│   └── /brokers/seqid                     ← счётчик для генерации broker ID
│
├── /controller                            ← какой брокер является Controller
│                                            (координатор кластера)
│
├── /controller_epoch                      ← номер эпохи Controller
│                                            (увеличивается при каждом re-election)
│
├── /admin                                 ← административные операции
│   ├── /admin/delete_topics               ← очередь топиков на удаление
│   └── /admin/preferred_replica_election  ← запланированные выборы preferred replica
│
├── /isr_change_notification               ← уведомления об изменениях ISR
│
├── /config                                ← динамическая конфигурация
│   ├── /config/topics                     ← переопределённые настройки топиков
│   ├── /config/clients                    ← настройки клиентов (quotas)
│   └── /config/brokers                    ← переопределённые настройки брокеров
│
├── /cluster                               ← информация о кластере
│   └── /cluster/id                        ← уникальный ID кластера
│
└── /kafka-acl                             ← Access Control Lists
    ├── /kafka-acl/Topic                   ← ACL для топиков
    ├── /kafka-acl/Group                   ← ACL для consumer groups
    └── /kafka-acl/Cluster                 ← ACL на уровне кластера
```

**Детальное описание каждой функции ZooKeeper в Kafka:**

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║  1. BROKER REGISTRATION (Регистрация брокеров)                       ║
║  ────────────────────────────────────────────                        ║
║                                                                      ║
║  Когда Kafka брокер ЗАПУСКАЕТСЯ:                                     ║
║  • Он создаёт ЭФЕМЕРНЫЙ (ephemeral) ZNode в /brokers/ids/           ║
║  • Эфемерный = исчезнет, когда брокер отключится                    ║
║  • ZNode содержит: хост, порт, JMX-порт, протоколы, версию          ║
║                                                                      ║
║  Пример содержимого /brokers/ids/0:                                  ║
║  {                                                                   ║
║    "host": "broker1.example.com",                                    ║
║    "port": 9092,                                                     ║
║    "jmx_port": 9999,                                                 ║
║    "version": 5,                                                     ║
║    "endpoints": ["PLAINTEXT://broker1.example.com:9092"],            ║
║    "timestamp": "1705318800123"                                      ║
║  }                                                                   ║
║                                                                      ║
║  Когда брокер ПАДАЕТ:                                                ║
║  • Соединение с ZooKeeper разрывается                                ║
║  • ZooKeeper удаляет эфемерный ZNode                                 ║
║  • Другие брокеры получают уведомление (watch)                       ║
║  • Controller инициирует leader election для партиций                ║
║    этого брокера                                                     ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  2. CONTROLLER ELECTION (Выбор координатора кластера)                ║
║  ────────────────────────────────────────────────────                ║
║                                                                      ║
║  Controller — это ОДИН из брокеров, который выполняет                ║
║  дополнительные обязанности по управлению кластером:                 ║
║  • Назначает leader'ов для партиций                                  ║
║  • Обрабатывает падения брокеров                                     ║
║  • Создаёт/удаляет топики                                            ║
║  • Управляет ISR                                                     ║
║                                                                      ║
║  Как выбирается Controller:                                          ║
║  • При старте каждый брокер пытается ПЕРВЫМ создать                  ║
║    ZNode /controller                                                  ║
║  • Первый создавший — становится Controller                          ║
║  • Остальные следят (watch) за этим ZNode                             ║
║  • Если Controller упал → ZNode удаляется →                          ║
║    оставшиеся брокеры СНОВА соревнуются за создание ZNode            ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  3. TOPIC METADATA (Метаданные топиков)                              ║
║  ──────────────────────────────────────                              ║
║                                                                      ║
║  Для каждого топика ZooKeeper хранит:                                ║
║  • Список партиций                                                   ║
║  • Для каждой партиции: leader, replicas, ISR                        ║
║  • Конфигурация топика (retention, compression и т.д.)               ║
║                                                                      ║
║  Пример /brokers/topics/orders:                                      ║
║  {                                                                   ║
║    "version": 2,                                                     ║
║    "partitions": {                                                   ║
║      "0": [1, 2, 3],    ← партиция 0: реплики на брокерах 1,2,3     ║
║      "1": [2, 3, 1],    ← партиция 1: реплики на брокерах 2,3,1     ║
║      "2": [3, 1, 2]     ← партиция 2: реплики на брокерах 3,1,2     ║
║    }                                                                  ║
║  }                                                                   ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  4. CONSUMER GROUP COORDINATION (старые версии Kafka)                ║
║  ────────────────────────────────────────────────────                ║
║                                                                      ║
║  В СТАРЫХ версиях Kafka (< 0.9) consumer group offset'ы             ║
║  хранились в ZooKeeper. В НОВЫХ версиях offset'ы хранятся           ║
║  в СПЕЦИАЛЬНОМ ТОПИКЕ Kafka: __consumer_offsets                      ║
║                                                                      ║
║  Это было перенесено из ZooKeeper по причинам:                       ║
║  • ZooKeeper не справлялся с частыми записями offset                 ║
║  • Kafka сама лучше масштабируется для такой нагрузки                ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  5. ACCESS CONTROL LISTS (Права доступа)                             ║
║  ────────────────────────────────────────                            ║
║                                                                      ║
║  ACL правила хранятся в ZooKeeper под /kafka-acl/                    ║
║  Пример: кто может читать/писать в каждый топик                      ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### Проблемы ZooKeeper с Kafka

ZooKeeper работает надёжно, но у неё есть **существенные ограничения** при использовании с Kafka.

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                  ПРОБЛЕМЫ ZOOKEEPER ДЛЯ KAFKA                       ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  1. ОПЕРАЦИОННАЯ СЛОЖНОСТЬ                                           ║
║     • Нужно развёртывать и поддерживать ДВА кластера:               ║
║       Kafka cluster + ZooKeeper cluster                              ║
║     • Мониторинг двух систем                                         ║
║     • Два набора конфигураций, бэкапов, обновлений                   ║
║     • Разные экспертизы для Kafka и ZooKeeper                        ║
║                                                                      ║
║  2. ОГРАНИЧЕНИЕ МАСШТАБИРУЕМОСТИ                                     ║
║     • ZooKeeper хранит ВСЕ метаданные в памяти                      ║
║     • При сотнях тысяч партиций ZooKeeper начинает тормозить        ║
║     • Практический лимит: ~100,000-200,000 партиций                 ║
║     • Controller загружает ВСЕ метаданные из ZooKeeper при старте    ║
║       → медленный старт кластера                                     ║
║                                                                      ║
║  3. МЕДЛЕННЫЙ FAILOVER                                               ║
║     • При падении Controller нужно:                                  ║
║       а) ZooKeeper обнаруживает, что Controller недоступен           ║
║       б) Новый Controller загружает ВСЕ метаданные из ZooKeeper      ║
║       в) Это может занимать МИНУТЫ для крупных кластеров             ║
║                                                                      ║
║  4. ДОПОЛНИТЕЛЬНАЯ ТОЧКА ОТКАЗА                                      ║
║     • Если ZooKeeper кластер недоступен — Kafka НЕ может:           ║
║       - Создавать/удалять топики                                     ║
║       - Выбирать leader'ов                                           ║
║       - Регистрировать новые брокеры                                 ║
║     • Чтение/запись сообщений ещё работают (на короткое время)       ║
║     • Но любое изменение метаданных невозможно                       ║
║                                                                      ║
║  5. ЛИШНИЙ СЕТЕВОЙ HOP                                               ║
║     • Каждая операция с метаданными → сетевой запрос к ZooKeeper     ║
║     • Это добавляет латентность                                      ║
║     • Особенно заметно при частых изменениях (ISR changes)           ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### Может ли Kafka работать без ZooKeeper?

**ДА!** Начиная с версии 2.8, Kafka может работать в **KRaft режиме** (Kafka Raft), полностью заменяя ZooKeeper встроенным механизмом консенсуса.

**Что такое KRaft?**

KRaft — это реализация **протокола Raft** внутри самой Kafka. Raft — это алгоритм распределённого консенсуса, который позволяет группе серверов согласованно принимать решения даже при наличии сбоев. Вместо хранения метаданных во внешней системе (ZooKeeper), Kafka теперь хранит их в **своём собственном внутреннем топике метаданных**.

**Как работает KRaft:**

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                         KRAFT АРХИТЕКТУРА                            ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  В KRaft режиме есть ДВА типа ролей:                                ║
║                                                                      ║
║  1. CONTROLLER — управляет метаданными кластера                      ║
║     • Выбирается через Raft консенсус                               ║
║     • Хранит и реплицирует метаданные                                ║
║     • Не обрабатывает клиентские данные                               ║
║                                                                      ║
║  2. BROKER — обрабатывает данные (как обычно)                        ║
║     • Обслуживает Producer и Consumer запросы                        ║
║     • Хранит партиции                                                ║
║     • Получает метаданные от Controller'ов                           ║
║                                                                      ║
║  3. COMBINED — совмещает обе роли (для dev/малых кластеров)          ║
║                                                                      ║
║  ════════════════════════════════════════════════════════════════     ║
║                                                                      ║
║  Production Setup (разделённые роли):                                ║
║                                                                      ║
║  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               ║
║  │ Controller 1 │  │ Controller 2 │  │ Controller 3 │               ║
║  │  (ACTIVE)    │  │  (STANDBY)   │  │  (STANDBY)   │               ║
║  │              │  │              │  │              │               ║
║  │ Raft Leader  │  │ Raft Follower│  │ Raft Follower│               ║
║  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘               ║
║         │                 │                 │                       ║
║         └─────── Raft Consensus ────────────┘                       ║
║                         │                                           ║
║                 Metadata Log                                        ║
║                 (внутренний топик                                    ║
║                  @metadata)                                          ║
║                         │                                           ║
║         ┌───────────────┼───────────────┐                           ║
║         │               │               │                           ║
║         ▼               ▼               ▼                           ║
║  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐                ║
║  │  Broker 1    │ │  Broker 2    │ │  Broker 3    │                ║
║  │              │ │              │ │              │                ║
║  │  Получает    │ │  Получает    │ │  Получает    │                ║
║  │  метаданные  │ │  метаданные  │ │  метаданные  │                ║
║  │  от Controller│ │  от Controller│ │  от Controller│              ║
║  └──────────────┘ └──────────────┘ └──────────────┘                ║
║                                                                      ║
║  ════════════════════════════════════════════════════════════════     ║
║                                                                      ║
║  Dev/Small Setup (combined роли):                                    ║
║                                                                      ║
║  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐     ║
║  │  Node 1          │ │  Node 2          │ │  Node 3          │     ║
║  │  Controller +    │ │  Controller +    │ │  Controller +    │     ║
║  │  Broker          │ │  Broker          │ │  Broker          │     ║
║  │  (COMBINED)      │ │  (COMBINED)      │ │  (COMBINED)      │     ║
║  └──────────────────┘ └──────────────────┘ └──────────────────┘     ║
║                                                                      ║
║  Проще деплоить, но меньше изоляция                                  ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Metadata Log — ключевая инновация KRaft:**

Вместо хранения метаданных в ZooKeeper как «снимка состояния», KRaft хранит метаданные как **лог событий** (event log). Это означает, что вся история изменений метаданных сохраняется, как обычный Kafka лог.

text

```
@metadata topic (внутренний топик):

offset 0: TopicRecord { name="orders", id=UUID-1 }
offset 1: PartitionRecord { topicId=UUID-1, partitionId=0, leader=1, isr=[1,2,3] }
offset 2: PartitionRecord { topicId=UUID-1, partitionId=1, leader=2, isr=[1,2,3] }
offset 3: BrokerRegistrationRecord { brokerId=1, host="broker1", port=9092 }
offset 4: BrokerRegistrationRecord { brokerId=2, host="broker2", port=9092 }
offset 5: PartitionChangeRecord { topicId=UUID-1, partitionId=0, newISR=[1,2] }
  ...

Каждое изменение — это новая запись в логе.
Controller'ы реплицируют этот лог через Raft.
Broker'ы подписываются и получают обновления.
```

**Подробное сравнение ZooKeeper vs KRaft:**

text

```
┌──────────────────────┬────────────────────────┬────────────────────────┐
│     Критерий         │    ZooKeeper           │       KRaft            │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Количество           │ Kafka + ZooKeeper      │ Только Kafka           │
│ компонентов          │ (2 кластера)           │ (1 кластер)            │
│                      │                        │                        │
│                      │ Нужно развёртывать,    │ Всё в одном —          │
│                      │ мониторить, обновлять  │ проще управлять        │
│                      │ ДВЕ системы            │                        │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Макс. количество     │ ~100,000 - 200,000     │ Миллионы               │
│ партиций             │                        │                        │
│                      │ ZooKeeper хранит ВСЕ   │ Метаданные             │
│                      │ метаданные в памяти,   │ распределены,          │
│                      │ лимит зависит от RAM   │ нет единого            │
│                      │                        │ bottleneck             │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Время старта         │ Минуты                 │ Секунды                │
│ кластера             │                        │                        │
│                      │ Controller загружает   │ Controller читает      │
│                      │ ВСЕ метаданные из ZK   │ только свой лог        │
│                      │ при старте — это долго  │ — это быстро           │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Время failover       │ Десятки секунд -       │ Миллисекунды -         │
│ Controller           │ минуты                 │ секунды                │
│                      │                        │                        │
│                      │ Новый Controller       │ Raft мгновенно         │
│                      │ заново загружает ВСЕ   │ переключает            │
│                      │ метаданные из ZK       │ leadership             │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Сетевая              │ Kafka ↔ ZooKeeper      │ Нет внешних            │
│ латентность          │ (дополнительный hop)   │ hop'ов                 │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Операционная         │ Высокая                │ Низкая                 │
│ сложность            │ (два кластера)         │ (один кластер)         │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Production           │ ✅ Да (стабильно       │ ✅ Да (с Kafka 3.3)    │
│ ready                │ десятилетиями)         │                        │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Будущее              │ ❌ Deprecated           │ ✅ Стратегическое      │
│                      │ Удалят в Kafka 4.0     │ направление            │
└──────────────────────┴────────────────────────┴────────────────────────┘
```

**Хронология перехода:**

text

```
Kafka 0.x - 2.7:  ZooKeeper ОБЯЗАТЕЛЕН (нет альтернативы)
Kafka 2.8 (2021): KRaft в Early Access (для экспериментов)
Kafka 3.0 (2021): KRaft улучшен, но ещё не для production
Kafka 3.3 (2022): KRaft объявлен PRODUCTION READY ✅
Kafka 3.5 (2023): ZooKeeper объявлен DEPRECATED ⚠️
Kafka 4.0 (2024): ZooKeeper будет ПОЛНОСТЬЮ УДАЛЁН ❌

РЕКОМЕНДАЦИЯ:
  • Новые проекты → ТОЛЬКО KRaft
  • Существующие проекты → планируйте миграцию на KRaft
```

**Миграция с ZooKeeper на KRaft:**

Bash

```
# Kafka предоставляет инструмент миграции:

# Шаг 1: Убедиться, что версия Kafka >= 3.4
bin/kafka-server-start.sh --version

# Шаг 2: Подготовить Controller'ы в KRaft режиме
# (запустить рядом с существующим ZooKeeper кластером)

# Шаг 3: Запустить миграцию
bin/kafka-metadata-migration.sh \
  --bootstrap-controllers controller1:9093,controller2:9093,controller3:9093 \
  --zookeeper-connect zk1:2181,zk2:2181,zk3:2181

# Шаг 4: Метаданные копируются из ZooKeeper в KRaft
# Шаг 5: Переключить брокеры на KRaft
# Шаг 6: Отключить ZooKeeper

# Процесс может быть выполнен БЕЗ даунтайма (rolling migration)
```

---

## 5. Отказоустойчивость кластера Kafka — Глубокое погружение

### Зачем нужна отказоустойчивость?

В production среде **серверы падают**. Это не вопрос «если», а вопрос «когда». Причины могут быть разными: аппаратный сбой, проблемы с сетью, обновление ОС, переполнение диска, out-of-memory ошибки. Kafka спроектирована так, чтобы **пережить** такие сбои **без потери данных** и **с минимальным даунтаймом**.

### Репликация — фундамент отказоустойчивости

Репликация — это процесс **копирования данных** каждой партиции на **несколько брокеров**. Если один брокер упадёт, данные всё ещё доступны на других.

**Роли в репликации:**

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║  LEADER (Лидер)                                                      ║
║  ═══════════════                                                     ║
║  • Единственный брокер, который обслуживает READ и WRITE             ║
║    для конкретной партиции                                           ║
║  • Producer'ы пишут ТОЛЬКО в Leader                                  ║
║  • Consumer'ы читают ТОЛЬКО из Leader (по умолчанию)                 ║
║  • Отвечает за репликацию данных Follower'ам                         ║
║  • Отслеживает, какие Follower'ы в ISR                               ║
║                                                                      ║
║  FOLLOWER (Фолловер)                                                 ║
║  ═══════════════════                                                 ║
║  • Брокеры, которые РЕПЛИЦИРУЮТ данные с Leader'а                    ║
║  • НЕ обслуживают клиентские запросы напрямую                        ║
║    (кроме Kafka 2.4+ с Follower Fetching)                            ║
║  • Периодически отправляют Fetch запросы к Leader'у                  ║
║  • Если Leader упал — один из Follower'ов станет новым Leader'ом     ║
║                                                                      ║
║  ISR (In-Sync Replicas)                                              ║
║  ═══════════════════════                                             ║
║  • Список реплик (Leader + Follower'ы), которые «не отстают»        ║
║  • Follower считается in-sync, если:                                ║
║    - Отстаёт не более чем на replica.lag.time.max.ms (default 30с) ║
║    - Активно отправляет Fetch запросы                                ║
║  • Только реплики из ISR могут стать новым Leader'ом                 ║
║    (если unclean.leader.election.enable=false)                       ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Процесс записи с репликацией (шаг за шагом):**

text

```
Рассмотрим: replication.factor=3, min.insync.replicas=2, acks=all

═══ ШАГ 1: Producer отправляет сообщение ═══

Producer ──── "Hello, Kafka!" ────▶ Leader (Broker 1)

Producer знает, кто Leader, потому что при подключении
получил метаданные о всех партициях и их Leader'ах.


═══ ШАГ 2: Leader записывает сообщение ═══

Leader (Broker 1):
  ┌─────────────────────────────────────────────────┐
  │ Partition 0:                                     │
  │ [msg0, msg1, msg2, msg3, "Hello, Kafka!"]       │
  │                                    ↑             │
  │                            offset 4 (новый)      │
  └─────────────────────────────────────────────────┘

Сообщение записано на диск Leader'а.
НО Producer ещё НЕ получил подтверждение (acks=all).


═══ ШАГ 3: Follower'ы реплицируют ═══

Follower (Broker 2) ──Fetch──▶ Leader: "Дай данные с offset 4"
Leader ──────────────response──▶ Follower: "Hello, Kafka!" (offset 4)

Follower (Broker 2):
  ┌─────────────────────────────────────────────────┐
  │ Partition 0:                                     │
  │ [msg0, msg1, msg2, msg3, "Hello, Kafka!"]  ✅   │
  └─────────────────────────────────────────────────┘

Follower (Broker 3) ──Fetch──▶ Leader: "Дай данные с offset 4"
Leader ──────────────response──▶ Follower: "Hello, Kafka!" (offset 4)

Follower (Broker 3):
  ┌─────────────────────────────────────────────────┐
  │ Partition 0:                                     │
  │ [msg0, msg1, msg2, msg3, "Hello, Kafka!"]  ✅   │
  └─────────────────────────────────────────────────┘


═══ ШАГ 4: Leader обновляет High Water Mark ═══

Все ISR реплики подтвердили запись.
ISR = {Broker 1, Broker 2, Broker 3}

High Water Mark (HWM) сдвигается на offset 4.
Consumer'ы теперь могут прочитать это сообщение.


═══ ШАГ 5: Leader отправляет ACK Producer'у ═══

Leader ──── ACK (success, offset=4) ────▶ Producer

Producer знает: сообщение записано МИНИМУМ на 2 реплики (min.insync.replicas=2).
Даже если Leader упадёт ПРЯМО СЕЙЧАС — сообщение НЕ потеряется.
```

**Настройки Producer ACK — влияние на надёжность и производительность:**

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║  acks=0 — «Выстрелил и забыл»                                       ║
║  ═══════════════════════════════                                     ║
║                                                                      ║
║  Producer отправил сообщение и СРАЗУ продолжает работу.              ║
║  НЕ ждёт никакого подтверждения.                                     ║
║                                                                      ║
║  Producer ──msg──▶ Broker                                            ║
║  Producer продолжает ✅  (даже не знает, дошло ли сообщение)         ║
║                                                                      ║
║  ✅ Максимальная скорость (нет ожидания)                              ║
║  ✅ Минимальная латентность                                           ║
║  ❌ Сообщения МОГУТ потеряться (нет подтверждения)                    ║
║  ❌ Producer не узнает об ошибке                                      ║
║                                                                      ║
║  📌 Применение: логи, метрики, телеметрия                            ║
║     (потеря единичного значения некритична)                           ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  acks=1 — «Подтверждение от Leader'а»                                ║
║  ════════════════════════════════════                                 ║
║                                                                      ║
║  Producer ждёт, пока Leader запишет сообщение на СВОЙ диск.          ║
║  НЕ ждёт репликацию на Follower'ов.                                  ║
║                                                                      ║
║  Producer ──msg──▶ Leader                                            ║
║                    Leader записал ✅                                  ║
║  Producer ◀──ACK── Leader                                            ║
║                    (Follower'ы ещё НЕ реплицировали!)               ║
║                                                                      ║
║  ✅ Хорошая скорость                                                 ║
║  ⚠️  Если Leader УПАДЁТ сразу после ACK, но ДО репликации —         ║
║     сообщение ПОТЕРЯЕТСЯ!                                            ║
║                                                                      ║
║  Временное окно потери данных:                                       ║
║  Leader записал → ACK → Leader упал → новый Leader                   ║
║                                 ↑                                    ║
║                     Follower'ы НЕ реплицировали                      ║
║                     → сообщение потеряно                              ║
║                                                                      ║
║  📌 Применение: большинство задач (баланс скорость/надёжность)       ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  acks=all (или acks=-1) — «Подтверждение от всех ISR»               ║
║  ══════════════════════════════════════════════════                   ║
║                                                                      ║
║  Producer ждёт, пока Leader И ВСЕ реплики из ISR                     ║
║  запишут сообщение.                                                  ║
║                                                                      ║
║  Producer ──msg──▶ Leader                                            ║
║                    Leader записал ✅                                  ║
║                    Follower 1 реплицировал ✅                         ║
║                    Follower 2 реплицировал ✅                         ║
║  Producer ◀──ACK── Leader                                            ║
║                                                                      ║
║  ⚠️  НО если ISR={Leader}, то acks=all == acks=1!                    ║
║  Поэтому ОБЯЗАТЕЛЬНО используйте вместе с min.insync.replicas≥2     ║
║                                                                      ║
║  ✅ Максимальная надёжность                                          ║
║  ✅ Данные НЕ потеряются при падении Leader'а                        ║
║  ❌ Самая медленная запись (ждём все ISR)                             ║
║  ❌ Если ISR < min.insync.replicas → ошибка записи                   ║
║                                                                      ║
║  📌 Применение: финансовые транзакции, критичные данные               ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### Полная production конфигурация кластера

Рассмотрим настройку **высокодоступного** Kafka кластера.

**Требования:**

- Пережить падение **любого одного** брокера без потери данных
- Минимальный даунтайм при failover
- Автоматическое восстановление

**Архитектура:**

text

```
┌──────────────────────────────────────────────────────────────────────┐
│                   PRODUCTION KAFKA CLUSTER                           │
│                                                                      │
│  3+ брокера в РАЗНЫХ Availability Zones (стойках, дата-центрах)     │
│                                                                      │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐   │
│  │   BROKER 1       │  │   BROKER 2       │  │   BROKER 3       │   │
│  │   AZ-A (Rack A)  │  │   AZ-B (Rack B)  │  │   AZ-C (Rack C)  │   │
│  │                  │  │                  │  │                  │   │
│  │  Topic: orders   │  │  Topic: orders   │  │  Topic: orders   │   │
│  │  P0: LEADER      │  │  P0: FOLLOWER    │  │  P0: FOLLOWER    │   │
│  │  P1: FOLLOWER    │  │  P1: LEADER      │  │  P1: FOLLOWER    │   │
│  │  P2: FOLLOWER    │  │  P2: FOLLOWER    │  │  P2: LEADER      │   │
│  │                  │  │                  │  │                  │   │
│  │  + другие топики │  │  + другие топики │  │  + другие топики │   │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘   │
│         │                      │                      │             │
│         └──────────────────────┼──────────────────────┘             │
│                                │                                    │
│                    KRaft Consensus                                   │
│                    (или ZooKeeper)                                   │
│                                                                      │
│  КЛЮЧЕВОЕ: Leader'ы распределены РАВНОМЕРНО между брокерами         │
│  → нагрузка сбалансирована                                          │
│  → при падении одного брокера потеря — максимум 1/3 Leader'ов       │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

**Конфигурация брокеров (server.properties):**

properties

```
# ════════════════════════════════════════════════════════════
# ОСНОВНЫЕ НАСТРОЙКИ ОТКАЗОУСТОЙЧИВОСТИ
# ════════════════════════════════════════════════════════════

# Replication factor по умолчанию для НОВЫХ топиков
# Рекомендация: 3 для production
default.replication.factor=3

# Минимальное кол-во ISR для записи
# С replication.factor=3 и min.insync.replicas=2:
# → Можно пережить падение 1 брокера без потери данных
# → Если 2 из 3 брокеров упали → запись БЛОКИРУЕТСЯ (защита от потери)
min.insync.replicas=2

# Replication для служебных топиков (offset'ы consumer groups)
offsets.topic.replication.factor=3
transaction.state.log.replication.factor=3
transaction.state.log.min.isr=2

# ════════════════════════════════════════════════════════════
# LEADER ELECTION
# ════════════════════════════════════════════════════════════

# ЗАПРЕТИТЬ выбор Leader из out-of-sync реплик
# false = если все ISR упали, партиция НЕДОСТУПНА (но данные в безопасности)
# true  = выбрать Leader из любой реплики (возможна потеря данных!)
unclean.leader.election.enable=false

# Время, после которого Follower считается out-of-sync
# Если Follower не отправлял Fetch запрос дольше этого времени → out-of-sync
replica.lag.time.max.ms=30000

# ════════════════════════════════════════════════════════════
# RACK AWARENESS (распределение реплик по стойкам/AZ)
# ════════════════════════════════════════════════════════════

# Каждый брокер указывает свою "стойку" (или Availability Zone)
broker.rack=az-a  # для Broker 1
# broker.rack=az-b  # для Broker 2
# broker.rack=az-c  # для Broker 3

# Kafka будет ГАРАНТИРОВАТЬ, что реплики одной партиции
# находятся в РАЗНЫХ стойках/AZ
# → даже если вся стойка упадёт — данные сохранятся

# ════════════════════════════════════════════════════════════
# ДИСКОВЫЕ НАСТРОЙКИ
# ════════════════════════════════════════════════════════════

# Принудительная синхронизация на диск каждые N сообщений
# По умолчанию Kafka НЕ делает fsync после каждого сообщения (полагается на ОС)
# Для максимальной надёжности:
# log.flush.interval.messages=1        # fsync после КАЖДОГО сообщения (МЕДЛЕННО!)
# log.flush.interval.ms=1000           # fsync каждую секунду

# РЕКОМЕНДАЦИЯ: НЕ включать fsync, полагаться на репликацию
# Репликация + acks=all надёжнее и быстрее, чем fsync на один диск

# ════════════════════════════════════════════════════════════
# СЕТЕВЫЕ НАСТРОЙКИ
# ════════════════════════════════════════════════════════════

# Размер сетевых буферов
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600

# Количество сетевых потоков
num.network.threads=8
num.io.threads=16

# ════════════════════════════════════════════════════════════
# ХРАНЕНИЕ (Retention)
# ════════════════════════════════════════════════════════════

# Как долго хранить данные
log.retention.hours=168  # 7 дней

# Размер сегмента лога
log.segment.bytes=1073741824  # 1 GB

# Как часто проверять retention
log.retention.check.interval.ms=300000  # 5 минут
```

**Создание топика для production:**

Bash

```
# Создать топик с правильными настройками отказоустойчивости
bin/kafka-topics.sh --create \
  --bootstrap-server broker1:9092,broker2:9092,broker3:9092 \
  --topic payments \
  --partitions 12 \
  --replication-factor 3 \
  --config min.insync.replicas=2 \
  --config retention.ms=2592000000 \
  --config compression.type=lz4 \
  --config cleanup.policy=delete

# Объяснение:
# --partitions 12      → 12 партиций для параллелизма
#                        (кратно числу consumer'ов)
# --replication-factor 3 → 3 копии каждой партиции
# min.insync.replicas=2  → запись только при 2+ живых ISR
# retention.ms           → хранить 30 дней
# compression.type=lz4   → быстрое сжатие
```

### Мониторинг здоровья кластера

Мониторинг — это **глаза и уши** вашего кластера. Без мониторинга вы узнаете о проблеме только когда пользователи начнут жаловаться.

Bash

```
# ════════════════════════════════════════════════════════════
# CLI КОМАНДЫ ДЛЯ МОНИТОРИНГА
# ════════════════════════════════════════════════════════════

# 1. Партиции, где реплики НЕ в синхронизации
# АЛЕРТ: если этот список НЕ пуст — значит есть проблемы с репликацией!
bin/kafka-topics.sh --describe \
  --bootstrap-server localhost:9092 \
  --under-replicated-partitions

# Пример вывода (ПРОБЛЕМА!):
# Topic: payments   Partition: 0   Leader: 1   Replicas: 1,2,3   Isr: 1,2
#                                                                      ↑
#                                                             Broker 3 выпал из ISR!

# 2. Партиции БЕЗ Leader'а (полностью недоступные)
# КРИТИЧЕСКИЙ АЛЕРТ: эти партиции невозможно читать/писать!
bin/kafka-topics.sh --describe \
  --bootstrap-server localhost:9092 \
  --unavailable-partitions

# 3. Состояние consumer groups (consumer lag)
bin/kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group payment-processors \
  --describe

# Вывод:
# GROUP              TOPIC     PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
# payment-processors payments  0          95000           100000          5000
# payment-processors payments  1          98000           100000          2000
# payment-processors payments  2          100000          100000          0
#
# LAG = разница между последним сообщением и позицией consumer'а
# LAG > 0 означает: consumer отстаёт
# Большой и растущий LAG = АЛЕРТ! Consumer не успевает
```

**Ключевые метрики для Prometheus/Grafana:**

text

```
═══════════════════════════════════════════════════════════════
КРИТИЧЕСКИЕ МЕТРИКИ (настройте алерты!)

1. UnderReplicatedPartitions
   Кол-во партиций, где реплики отстают
   НОРМА: 0
   АЛЕРТ: > 0 дольше 5 минут

2. OfflinePartitionsCount
   Кол-во партиций без Leader'а
   НОРМА: 0
   АЛЕРТ: > 0 (НЕМЕДЛЕННО!)

3. ActiveControllerCount
   Количество активных Controller'ов
   НОРМА: ровно 1
   АЛЕРТ: 0 (нет Controller!) или > 1 (split brain!)

4. ISRShrinks / ISRExpands
   Частота изменений ISR
   Частые shrinks = проблемы с сетью или диском

5. ConsumerLag
   Отставание consumer'а от последнего сообщения
   Растущий lag = consumer не успевает обрабатывать
═══════════════════════════════════════════════════════════════
```

### Сценарии отказов и восстановления

text

```
═══════════════════════════════════════════════════════════════════
              СЦЕНАРИЙ 1: Падение одного брокера
═══════════════════════════════════════════════════════════════════

НАЧАЛЬНОЕ СОСТОЯНИЕ:
  Broker 1: Leader(P0), Follower(P1, P2)
  Broker 2: Leader(P1), Follower(P0, P2)
  Broker 3: Leader(P2), Follower(P0, P1)
  
  ISR(P0) = {1, 2, 3}
  ISR(P1) = {1, 2, 3}
  ISR(P2) = {1, 2, 3}

СОБЫТИЕ: Broker 1 УПАЛ (hardware failure)

1. Controller обнаруживает: Broker 1 недоступен
2. Для P0 (Leader был на Broker 1):
   → Выбирается новый Leader из ISR: Broker 2
   → ISR(P0) = {2, 3}
3. Для P1 и P2 (Broker 1 был Follower):
   → ISR(P1) = {2, 3}
   → ISR(P2) = {2, 3}

РЕЗУЛЬТАТ:
  Broker 2: Leader(P0, P1), Follower(P2)   ← нагрузка увеличилась!
  Broker 3: Leader(P2), Follower(P0, P1)
  
  ✅ Все партиции доступны
  ✅ Данные НЕ потеряны
  ⚠️  Broker 2 под повышенной нагрузкой

ВОССТАНОВЛЕНИЕ:
  Broker 1 поднимается:
  1. Регистрируется в кластере
  2. Начинает реплицировать данные с Leader'ов
  3. Догоняет ISR
  4. Preferred Replica Election возвращает Leader'ов на место

═══════════════════════════════════════════════════════════════════
              СЦЕНАРИЙ 2: Падение двух брокеров
═══════════════════════════════════════════════════════════════════

  replication.factor=3, min.insync.replicas=2

СОБЫТИЕ: Broker 1 и Broker 2 упали одновременно

Для каждой партиции:
  ISR = {Broker 3}  ← только ОДИН живой ISR

  min.insync.replicas=2, но живой ISR только 1!
  
  → ЧТЕНИЕ работает ✅ (Leader на Broker 3)
  → ЗАПИСЬ ЗАБЛОКИРОВАНА ❌ (ISR < min.insync.replicas)
  → Producer получает NotEnoughReplicasException

  ⚠️  Данные в безопасности, но новая запись невозможна!
  
  Это ПРАВИЛЬНОЕ поведение — лучше остановить запись,
  чем рисковать потерей данных при падении Broker 3.

═══════════════════════════════════════════════════════════════════
              СЦЕНАРИЙ 3: Падение ВСЕХ брокеров с данной партицией
═══════════════════════════════════════════════════════════════════

СОБЫТИЕ: Все 3 брокера с репликами P0 упали

  → Партиция P0 ПОЛНОСТЬЮ НЕДОСТУПНА ❌
  → Чтение и запись невозможны

  Если unclean.leader.election.enable=false:
    → Ждём, пока вернётся хотя бы один ISR брокер
    → Данные не потеряются, но даунтайм

  Если unclean.leader.election.enable=true:
    → Любой вернувшийся брокер может стать Leader
    → НО может потерять данные, которые были только на других
    → Доступность выше, но надёжность ниже

═══════════════════════════════════════════════════════════════════
```

---

## 6. Разграничение доступа к топикам (ACL) — Подробное руководство

### Зачем нужен контроль доступа?

В production среде разные команды и сервисы работают с разными данными. Без контроля доступа:

- Любой сервис может **читать конфиденциальные данные** (платежи, персональные данные)
- Любой сервис может **писать мусорные данные** в чужие топики
- Любой может **удалять** топики и consumer groups
- Невозможно соблюдать требования безопасности и compliance (GDPR, PCI DSS)

**Kafka ACL (Access Control Lists)** позволяют определить: **кто** может выполнять **какие операции** над **какими ресурсами**.

### Как работает авторизация в Kafka

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                     ПРОЦЕСС АВТОРИЗАЦИИ                              ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  1. Клиент ПОДКЛЮЧАЕТСЯ к Kafka                                      ║
║     → Аутентификация (SASL/SSL): "Кто ты?"                          ║
║     → Результат: Principal (идентичность), например User:alice       ║
║                                                                      ║
║  2. Клиент ОТПРАВЛЯЕТ запрос (читать/писать/создать/...)            ║
║     → Авторизация (ACL): "Имеешь ли ты право?"                      ║
║     → Kafka проверяет ACL для этого Principal + Resource + Operation ║
║                                                                      ║
║  3. РЕШЕНИЕ:                                                         ║
║     → ALLOW: запрос выполняется ✅                                   ║
║     → DENY:  запрос отклоняется ❌ (ошибка авторизации)             ║
║                                                                      ║
║  ┌──────────────────────────────────────────────────────────────┐    ║
║  │                     ACL ЗАПИСЬ                                │    ║
║  │                                                              │    ║
║  │  Principal:  User:alice                                      │    ║
║  │  Permission: ALLOW                                           │    ║
║  │  Operation:  READ                                            │    ║
║  │  Resource:   Topic:orders                                    │    ║
║  │  Host:       192.168.1.100                                   │    ║
║  │                                                              │    ║
║  │  Читается как: "Пользователь alice МОЖЕТ ЧИТАТЬ              │    ║
║  │   топик orders с хоста 192.168.1.100"                        │    ║
║  └──────────────────────────────────────────────────────────────┘    ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### Шаг 1: Настройка аутентификации

Прежде чем настраивать ACL, нужно настроить **аутентификацию** — Kafka должна знать, **кто** подключается. Без аутентификации все клиенты анонимны, и ACL бесполезны.

**SASL/SCRAM — рекомендуемый метод:**

SCRAM (Salted Challenge Response Authentication Mechanism) хранит пароли в виде **солёных хэшей**, а не в открытом виде. Это значительно безопаснее, чем SASL/PLAIN.

properties

```
# ════════════════════════════════════════════════════════════
# server.properties — настройка брокера
# ════════════════════════════════════════════════════════════

# Listener с аутентификацией (SASL) и шифрованием (SSL)
listeners=SASL_SSL://0.0.0.0:9093

# Протокол для inter-broker коммуникации
security.inter.broker.protocol=SASL_SSL

# Механизм аутентификации между брокерами
sasl.mechanism.inter.broker.protocol=SCRAM-SHA-256

# Поддерживаемые механизмы для клиентов
sasl.enabled.mechanisms=SCRAM-SHA-256

# Класс авторизации (включает ACL)
authorizer.class.name=kafka.security.authorizer.AclAuthorizer

# Super users — они ВСЕГДА имеют полный доступ (не проверяются ACL)
# Используйте для административных задач
super.users=User:admin;User:kafka-internal

# Что делать, если для ресурса нет ACL?
# false = по умолчанию доступ ЗАПРЕЩЁН (рекомендуется!)
# true  = по умолчанию доступ РАЗРЕШЁН (опасно в production)
allow.everyone.if.no.acl.found=false
```

**Создание пользователей:**

Bash

```
# Создать пользователя admin
bin/kafka-configs.sh --bootstrap-server localhost:9092 \
  --alter \
  --add-config 'SCRAM-SHA-256=[password=admin-super-secret-password]' \
  --entity-type users \
  --entity-name admin

# Создать пользователя для Order Service
bin/kafka-configs.sh --bootstrap-server localhost:9092 \
  --alter \
  --add-config 'SCRAM-SHA-256=[password=order-svc-p@ssw0rd]' \
  --entity-type users \
  --entity-name order-service

# Создать пользователя для Payment Service
bin/kafka-configs.sh --bootstrap-server localhost:9092 \
  --alter \
  --add-config 'SCRAM-SHA-256=[password=payment-svc-p@ssw0rd]' \
  --entity-type users \
  --entity-name payment-service

# Создать пользователя для Analytics
bin/kafka-configs.sh --bootstrap-server localhost:9092 \
  --alter \
  --add-config 'SCRAM-SHA-256=[password=analytics-p@ssw0rd]' \
  --entity-type users \
  --entity-name analytics-reader

# Список всех пользователей
bin/kafka-configs.sh --bootstrap-server localhost:9092 \
  --describe \
  --entity-type users
```

### Шаг 2: Настройка ACL

**Типы операций в Kafka:**

text

```
╔═══════════════════════════════════════════════════════════════╗
║                    ОПЕРАЦИИ KAFKA                             ║
╠════════════════╦══════════════════════════════════════════════╣
║   Операция     ║   Описание                                   ║
╠════════════════╬══════════════════════════════════════════════╣
║ Read           ║ Читать сообщения из топика                   ║
║ Write          ║ Писать сообщения в топик                     ║
║ Create         ║ Создавать топики                             ║
║ Delete         ║ Удалять топики и записи                      ║
║ Alter          ║ Изменять конфигурацию                        ║
║ Describe       ║ Получать метаданные (список партиций и т.д.) ║
║ ClusterAction  ║ Межброкерные операции                        ║
║ DescribeConfigs║ Просматривать конфигурацию                   ║
║ AlterConfigs   ║ Изменять конфигурацию                        ║
║ IdempotentWrite║ Идемпотентная запись                         ║
║ All            ║ ВСЕ операции                                 ║
╚════════════════╩══════════════════════════════════════════════╝
```

**Типы ресурсов:**

text

```
╔═══════════════════════════════════════════════════════════════╗
║                    РЕСУРСЫ KAFKA                              ║
╠════════════════╦══════════════════════════════════════════════╣
║   Ресурс       ║   Описание                                   ║
╠════════════════╬══════════════════════════════════════════════╣
║ Topic          ║ Конкретный топик                             ║
║ Group          ║ Consumer group                               ║
║ Cluster        ║ Весь кластер                                 ║
║ TransactionalId║ ID транзакции                                ║
║ DelegationToken║ Токен делегации                              ║
╚════════════════╩══════════════════════════════════════════════╝
```

**Практические примеры настройки ACL:**

Bash

```
# ════════════════════════════════════════════════════════════
# ПРИМЕР 1: Order Service — может ПИСАТЬ в "orders"
# ════════════════════════════════════════════════════════════

# Разрешить order-service ПИСАТЬ в топик "orders"
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --add \
  --allow-principal User:order-service \
  --operation Write \
  --topic orders

# Разрешить order-service ОПИСЫВАТЬ топик "orders"
# (нужно для Producer, чтобы получить метаданные о партициях)
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --add \
  --allow-principal User:order-service \
  --operation Describe \
  --topic orders

# Разрешить идемпотентную запись (для exactly-once)
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --add \
  --allow-principal User:order-service \
  --operation IdempotentWrite \
  --cluster


# ════════════════════════════════════════════════════════════
# ПРИМЕР 2: Payment Service — может ЧИТАТЬ "orders" 
#            и ПИСАТЬ в "payments"
# ════════════════════════════════════════════════════════════

# Чтение из "orders"
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --add \
  --allow-principal User:payment-service \
  --operation Read \
  --topic orders

# Consumer Group для Payment Service
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --add \
  --allow-principal User:payment-service \
  --operation Read \
  --group payment-processing-group

# Запись в "payments"
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --add \
  --allow-principal User:payment-service \
  --operation Write \
  --topic payments

# Describe для обоих топиков
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --add \
  --allow-principal User:payment-service \
  --operation Describe \
  --topic orders

bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --add \
  --allow-principal User:payment-service \
  --operation Describe \
  --topic payments


# ════════════════════════════════════════════════════════════
# ПРИМЕР 3: Analytics — может ЧИТАТЬ ВСЕ топики (только чтение!)
# ════════════════════════════════════════════════════════════

# Чтение из всех топиков с префиксом (Kafka 2.0+)
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --add \
  --allow-principal User:analytics-reader \
  --operation Read \
  --operation Describe \
  --topic '*' \
  --resource-pattern-type literal

# Consumer Group для Analytics
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --add \
  --allow-principal User:analytics-reader \
  --operation Read \
  --group analytics-consumer-group

# Analytics НЕ может:
# ❌ Писать в топики
# ❌ Создавать/удалять топики
# ❌ Изменять конфигурацию


# ════════════════════════════════════════════════════════════
# ПРИМЕР 4: Wildcard ACL с префиксом
# ════════════════════════════════════════════════════════════

# Сервис может читать/писать ВСЕ топики, начинающиеся с "billing-"
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --add \
  --allow-principal User:billing-service \
  --operation Read \
  --operation Write \
  --operation Describe \
  --topic billing- \
  --resource-pattern-type prefixed

# Это разрешает доступ к:
# billing-invoices ✅
# billing-payments ✅
# billing-refunds  ✅
# orders           ❌ (не начинается с "billing-")


# ════════════════════════════════════════════════════════════
# ПРИМЕР 5: Ограничение по IP-адресу
# ════════════════════════════════════════════════════════════

# Разрешить доступ ТОЛЬКО с конкретных хостов
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --add \
  --allow-principal User:order-service \
  --allow-host 192.168.1.100 \
  --allow-host 192.168.1.101 \
  --operation Write \
  --topic orders

# Запросы с других IP будут отклонены, даже с правильным паролем


# ════════════════════════════════════════════════════════════
# ПРИМЕР 6: DENY ACL (явный запрет)
# ════════════════════════════════════════════════════════════

# ЗАПРЕТИТЬ пользователю intern доступ к топику "secrets"
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --add \
  --deny-principal User:intern \
  --operation All \
  --topic secrets

# DENY имеет ПРИОРИТЕТ над ALLOW!
# Даже если есть ALLOW для intern → DENY перекроет его
```

**Управление ACL:**

Bash

```
# Посмотреть ВСЕ ACL в кластере
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --list

# Посмотреть ACL для конкретного топика
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --list \
  --topic orders

# Посмотреть ACL для конкретного пользователя
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --list \
  --principal User:order-service

# Удалить конкретный ACL
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --remove \
  --allow-principal User:order-service \
  --operation Write \
  --topic orders

# Удалить ВСЕ ACL для ресурса
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --remove \
  --topic orders
```

**Подключение клиента с аутентификацией:**

Java

```
Properties props = new Properties();
props.put("bootstrap.servers", "kafka-broker:9093");

// Указываем протокол безопасности
props.put("security.protocol", "SASL_SSL");

// Механизм аутентификации
props.put("sasl.mechanism", "SCRAM-SHA-256");

// Учётные данные
props.put("sasl.jaas.config",
    "org.apache.kafka.common.security.scram.ScramLoginModule required " +
    "username=\"order-service\" " +
    "password=\"order-svc-p@ssw0rd\";");

// SSL truststore (для проверки сертификата сервера)
props.put("ssl.truststore.location", "/path/to/client.truststore.jks");
props.put("ssl.truststore.password", "truststore-password");

// Теперь Producer/Consumer будет аутентифицироваться как User:order-service
// и все ACL будут проверяться для этого пользователя
```

---

## 7. Защита данных от перехвата — Шифрование

### Три уровня защиты

Безопасность данных в Kafka обеспечивается на **трёх уровнях**, и каждый из них защищает от **разных угроз**:

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║  1. ENCRYPTION IN TRANSIT (шифрование при передаче)                  ║
║     SSL/TLS между клиентами и брокерами                              ║
║     Защита: от перехвата данных в сети (Man-in-the-Middle)          ║
║                                                                      ║
║  2. ENCRYPTION AT REST (шифрование на диске)                         ║
║     Шифрование файлов данных на уровне ОС или диска                 ║
║     Защита: от физического доступа к серверу/диску                   ║
║                                                                      ║
║  3. END-TO-END ENCRYPTION (сквозное шифрование)                      ║
║     Шифрование данных на уровне приложения                           ║
║     Защита: даже администраторы Kafka не видят данные                ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### Уровень 1: Encryption in Transit (SSL/TLS)

SSL/TLS шифрует **все данные**, передаваемые между клиентами (Producer/Consumer) и брокерами Kafka, а также между самими брокерами. Без шифрования данные передаются **в открытом виде**, и любой, кто имеет доступ к сети, может их **прочитать**.

**Что именно шифруется:**

text

```
БЕЗ SSL/TLS:

Producer ──── {"card": "4111-1111-1111-1111"} ────▶ Broker
                    ↑
       Злоумышленник в сети видит данные в открытом виде!
       Может прочитать номер карты, пароли, персональные данные


С SSL/TLS:

Producer ──── 0x7f3a9b2c...encrypted... ────▶ Broker
                    ↑
       Злоумышленник видит ТОЛЬКО зашифрованный трафик
       Расшифровать без ключа невозможно
```

**Пошаговая настройка SSL/TLS:**

Bash

```
# ════════════════════════════════════════════════════════════
# ШАГ 1: Создание Certificate Authority (CA)
# ════════════════════════════════════════════════════════════

# CA — это «нотариус», который подписывает сертификаты.
# В production обычно используется корпоративный CA или Let's Encrypt.
# Для dev/test можно создать свой self-signed CA.

mkdir -p /var/kafka/ssl
cd /var/kafka/ssl

# Генерация ключа CA и самоподписанного сертификата
openssl req -new -x509 \
  -keyout ca-key.pem \
  -out ca-cert.pem \
  -days 365 \
  -subj "/CN=Kafka-CA/O=MyCompany/C=US" \
  -passout pass:ca-password

# ca-key.pem  — приватный ключ CA (ХРАНИТЕ В СЕКРЕТЕ!)
# ca-cert.pem — публичный сертификат CA (можно раздавать клиентам)


# ════════════════════════════════════════════════════════════
# ШАГ 2: Создание Keystore для каждого брокера
# ════════════════════════════════════════════════════════════

# Keystore содержит ПРИВАТНЫЙ КЛЮЧ и СЕРТИФИКАТ сервера.
# Каждый брокер имеет СВОЙ keystore.

# Создать keystore с парой ключей
keytool -keystore kafka-broker1.keystore.jks \
  -alias broker1 \
  -validity 365 \
  -genkey \
  -keyalg RSA \
  -keysize 2048 \
  -dname "CN=broker1.example.com,O=MyCompany,C=US" \
  -storepass keystore-password \
  -keypass key-password \
  -ext SAN=DNS:broker1.example.com,DNS:localhost,IP:192.168.1.1

# SAN (Subject Alternative Names) — КРИТИЧЕСКИ ВАЖНО!
# Указывает все имена/IP, по которым клиенты могут подключаться.
# Без правильного SAN клиенты получат SSL handshake error.


# ════════════════════════════════════════════════════════════
# ШАГ 3: Создать CSR и подписать сертификат через CA
# ════════════════════════════════════════════════════════════

# Экспорт Certificate Signing Request
keytool -keystore kafka-broker1.keystore.jks \
  -alias broker1 \
  -certreq \
  -file broker1-csr.pem \
  -storepass keystore-password \
  -keypass key-password

# Подписать CSR с помощью CA
openssl x509 -req \
  -CA ca-cert.pem \
  -CAkey ca-key.pem \
  -in broker1-csr.pem \
  -out broker1-cert-signed.pem \
  -days 365 \
  -CAcreateserial \
  -passin pass:ca-password

# Импортировать CA-сертификат в keystore
keytool -keystore kafka-broker1.keystore.jks \
  -alias CARoot \
  -import -file ca-cert.pem \
  -storepass keystore-password \
  -noprompt

# Импортировать подписанный сертификат в keystore
keytool -keystore kafka-broker1.keystore.jks \
  -alias broker1 \
  -import -file broker1-cert-signed.pem \
  -storepass keystore-password \
  -keypass key-password


# ════════════════════════════════════════════════════════════
# ШАГ 4: Создать Truststore
# ════════════════════════════════════════════════════════════

# Truststore содержит CA-сертификат.
# Используется для проверки: «этот сертификат подписан нашим CA?»

keytool -keystore kafka.truststore.jks \
  -alias CARoot \
  -import -file ca-cert.pem \
  -storepass truststore-password \
  -noprompt

# Этот truststore ОДИНАКОВЫЙ для всех брокеров и клиентов
# (потому что все доверяют одному CA)
```

**Конфигурация брокера для SSL:**

properties

```
# ════════════════════════════════════════════════════════════
# server.properties — SSL конфигурация
# ════════════════════════════════════════════════════════════

# Listener с SSL шифрованием
listeners=SSL://0.0.0.0:9093
advertised.listeners=SSL://broker1.example.com:9093

# Для inter-broker коммуникации тоже используем SSL
security.inter.broker.protocol=SSL

# Keystore (содержит приватный ключ и сертификат этого брокера)
ssl.keystore.type=JKS
ssl.keystore.location=/var/kafka/ssl/kafka-broker1.keystore.jks
ssl.keystore.password=keystore-password
ssl.key.password=key-password

# Truststore (содержит CA-сертификат для проверки клиентов)
ssl.truststore.type=JKS
ssl.truststore.location=/var/kafka/ssl/kafka.truststore.jks
ssl.truststore.password=truststore-password

# Требовать сертификат от клиента (mutual TLS / двусторонняя аутентификация)
# required — клиент ОБЯЗАН предъявить свой сертификат
# requested — клиент МОЖЕТ предъявить (если есть)
# none — не требовать
ssl.client.auth=required

# Разрешённые протоколы (ТОЛЬКО современные!)
ssl.enabled.protocols=TLSv1.2,TLSv1.3
ssl.protocol=TLSv1.3

# Разрешённые cipher suites (ТОЛЬКО безопасные!)
ssl.cipher.suites=TLS_AES_256_GCM_SHA384,TLS_AES_128_GCM_SHA256
```

**Конфигурация клиента (Producer/Consumer) для SSL:**

Java

```
Properties props = new Properties();
props.put("bootstrap.servers", "broker1.example.com:9093");

// Протокол безопасности
props.put("security.protocol", "SSL");

// Truststore клиента (для проверки сертификата сервера)
// Клиент проверяет: «сертификат брокера подписан нашим CA?»
props.put("ssl.truststore.location", "/path/to/client.truststore.jks");
props.put("ssl.truststore.password", "truststore-password");

// Keystore клиента (для mutual TLS — если ssl.client.auth=required на сервере)
// Брокер проверяет: «сертификат клиента подписан нашим CA?»
props.put("ssl.keystore.location", "/path/to/client.keystore.jks");
props.put("ssl.keystore.password", "keystore-password");
props.put("ssl.key.password", "key-password");

// Проверка имени хоста в сертификате
// (защита от DNS spoofing)
props.put("ssl.endpoint.identification.algorithm", "https");
```

### Уровень 2: Encryption at Rest (шифрование на диске)

Kafka **НЕ шифрует данные на диске** из коробки. Файлы логов хранятся в открытом виде. Это значит, что любой, кто имеет **физический доступ** к серверу или **root-доступ** к ОС, может прочитать все сообщения напрямую из файлов.

Bash

```
# Без шифрования можно просто прочитать данные из файлов:
strings /var/lib/kafka/data/payments-0/00000000000000000000.log

# Вывод (данные видны в открытом виде!):
# {"card_number": "4111-1111-1111-1111", "amount": 99.99}
# {"card_number": "5500-0000-0000-0004", "amount": 149.50}
```

**Решения для шифрования на диске:**

Bash

```
# ════════════════════════════════════════════════════════════
# РЕШЕНИЕ 1: Шифрование на уровне файловой системы (Linux LUKS)
# ════════════════════════════════════════════════════════════

# LUKS (Linux Unified Key Setup) — стандарт шифрования дисков в Linux
# Шифрует ВЕСЬ раздел, прозрачно для Kafka

# Установить необходимые пакеты
sudo apt install cryptsetup

# Зашифровать раздел
sudo cryptsetup luksFormat /dev/sdb
# Будет запрошен пароль (passphrase)
# ⚠️  ВСЕ ДАННЫЕ НА ДИСКЕ БУДУТ УДАЛЕНЫ!

# Открыть зашифрованный раздел
sudo cryptsetup open /dev/sdb kafka-encrypted
# Введите пароль

# Создать файловую систему
sudo mkfs.ext4 /dev/mapper/kafka-encrypted

# Смонтировать
sudo mount /dev/mapper/kafka-encrypted /data/kafka-logs

# Настроить Kafka для использования этой директории:
# server.properties:
# log.dirs=/data/kafka-logs

# Теперь все данные Kafka шифруются на диске!
# Если диск украдут — данные невозможно прочитать без пароля.

# Для автоматического разблокирования при загрузке:
# используйте ключ-файл или интеграцию с Vault/KMS
```

Bash

```
# ════════════════════════════════════════════════════════════
# РЕШЕНИЕ 2: Облачное шифрование дисков
# ════════════════════════════════════════════════════════════

# AWS: EBS Encryption (включается одним флагом)
aws ec2 create-volume \
  --encrypted \
  --kms-key-id alias/kafka-key \
  --size 500 \
  --volume-type gp3

# Azure: Azure Disk Encryption
az disk create \
  --name kafka-data-disk \
  --encryption-type EncryptionAtRestWithPlatformKey \
  --size-gb 500

# GCP: Persistent Disk Encryption (включено по умолчанию!)
# Google шифрует ВСЕ данные на дисках по умолчанию
# Для дополнительного контроля: Customer-Managed Encryption Keys (CMEK)
gcloud compute disks create kafka-data-disk \
  --kms-key=projects/my-project/locations/us-central1/keyRings/kafka/cryptoKeys/kafka-key
```

### Уровень 3: End-to-End Encryption (сквозное шифрование)

Самый высокий уровень защиты. Данные шифруются **на стороне Producer'а** и расшифровываются **только Consumer'ом**. Kafka брокер хранит и передаёт **зашифрованные** данные, **не имея возможности** их прочитать.

text

```
Producer                      Kafka Broker                    Consumer
   │                              │                              │
   │ 1. Шифрует данные            │                              │
   │    AES-256(plaintext)        │                              │
   │    = encrypted blob          │                              │
   │                              │                              │
   │ ── encrypted blob ──────────▶│                              │
   │                              │ 2. Хранит encrypted blob    │
   │                              │    НЕ может расшифровать!   │
   │                              │                              │
   │                              │── encrypted blob ───────────▶│
   │                              │                              │ 3. Расшифровывает
   │                              │                              │    AES-256(blob)
   │                              │                              │    = plaintext
```

**Реализация на Java:**

Java

```
// ════════════════════════════════════════════════════════════
// Producer: шифрование перед отправкой
// ════════════════════════════════════════════════════════════

import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;
import javax.crypto.spec.GCMParameterSpec;
import java.security.SecureRandom;
import java.util.Base64;

public class EncryptedProducer {
    
    // Ключ шифрования (в production — из Vault, KMS, или HSM!)
    // НИКОГДА не хардкодьте ключи в коде!
    private static final byte[] ENCRYPTION_KEY = 
        getKeyFromVault("kafka-encryption-key"); // 256-bit key
    
    public String encrypt(String plaintext) throws Exception {
        // AES-256-GCM — современный и безопасный алгоритм
        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        SecretKeySpec keySpec = new SecretKeySpec(ENCRYPTION_KEY, "AES");
        
        // Генерация случайного IV (Initialization Vector)
        // IV ДОЛЖЕН быть уникальным для каждого сообщения!
        byte[] iv = new byte[12];
        new SecureRandom().nextBytes(iv);
        
        GCMParameterSpec gcmSpec = new GCMParameterSpec(128, iv);
        cipher.init(Cipher.ENCRYPT_MODE, keySpec, gcmSpec);
        
        byte[] encrypted = cipher.doFinal(plaintext.getBytes("UTF-8"));
        
        // Объединяем IV + зашифрованные данные
        // (IV нужен для расшифровки, его не нужно скрывать)
        byte[] combined = new byte[iv.length + encrypted.length];
        System.arraycopy(iv, 0, combined, 0, iv.length);
        System.arraycopy(encrypted, 0, combined, iv.length, encrypted.length);
        
        return Base64.getEncoder().encodeToString(combined);
    }
    
    public void sendEncrypted(String topic, String key, String message) {
        String encrypted = encrypt(message);
        
        ProducerRecord<String, String> record = 
            new ProducerRecord<>(topic, key, encrypted);
        
        producer.send(record);
        
        // В Kafka хранится: "dGhpcyBpcyBlbmNyeXB0ZWQ=" (зашифровано)
        // Вместо: {"card": "4111-1111-1111-1111"} (открытый текст)
    }
}


// ════════════════════════════════════════════════════════════
// Consumer: расшифровка после получения
// ════════════════════════════════════════════════════════════

public class EncryptedConsumer {
    
    private static final byte[] ENCRYPTION_KEY = 
        getKeyFromVault("kafka-encryption-key");
    
    public String decrypt(String encryptedBase64) throws Exception {
        byte[] combined = Base64.getDecoder().decode(encryptedBase64);
        
        // Извлечь IV (первые 12 байт)
        byte[] iv = new byte[12];
        System.arraycopy(combined, 0, iv, 0, 12);
        
        // Извлечь зашифрованные данные (остальное)
        byte[] encrypted = new byte[combined.length - 12];
        System.arraycopy(combined, 12, encrypted, 0, encrypted.length);
        
        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        SecretKeySpec keySpec = new SecretKeySpec(ENCRYPTION_KEY, "AES");
        GCMParameterSpec gcmSpec = new GCMParameterSpec(128, iv);
        cipher.init(Cipher.DECRYPT_MODE, keySpec, gcmSpec);
        
        byte[] decrypted = cipher.doFinal(encrypted);
        return new String(decrypted, "UTF-8");
    }
    
    public void consumeAndDecrypt() {
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
            
            for (ConsumerRecord<String, String> record : records) {
                // record.value() содержит зашифрованные данные
                String plaintext = decrypt(record.value());
                
                // Теперь plaintext содержит оригинальное сообщение
                processMessage(plaintext);
            }
        }
    }
}
```

### Управление ключами шифрования

Ключи шифрования — это **самая критичная** часть всей системы безопасности. Если ключ скомпрометирован — все данные открыты.

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║  ПРАВИЛА УПРАВЛЕНИЯ КЛЮЧАМИ:                                        ║
║                                                                      ║
║  ❌ НИКОГДА не хардкодьте ключи в исходном коде                      ║
║  ❌ НИКОГДА не храните ключи в Git                                    ║
║  ❌ НИКОГДА не храните ключи в конфиг-файлах без шифрования          ║
║  ❌ НИКОГДА не передавайте ключи по незашифрованному каналу           ║
║                                                                      ║
║  ✅ Используйте HashiCorp Vault, AWS KMS, Azure Key Vault, GCP KMS  ║
║  ✅ Ротируйте ключи регулярно (каждые 90 дней)                      ║
║  ✅ Используйте envelope encryption (ключ шифруется мастер-ключом)  ║
║  ✅ Логируйте все операции с ключами (аудит)                         ║
║  ✅ Используйте разные ключи для разных топиков                      ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### Network Isolation (сетевая изоляция)

Даже с шифрованием и аутентификацией, важно **ограничить сетевой доступ** к Kafka брокерам. Брокеры не должны быть доступны из публичного интернета.

text

```
┌──────────────────────────────────────────────────────────────────┐
│                    СЕТЕВАЯ АРХИТЕКТУРА                            │
│                                                                  │
│  ┌─────────────────────────────────────────────────┐             │
│  │              PRIVATE SUBNET                      │             │
│  │                                                  │             │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐       │             │
│  │  │ Broker 1 │  │ Broker 2 │  │ Broker 3 │       │             │
│  │  │ :9093    │  │ :9093    │  │ :9093    │       │             │
│  │  └──────────┘  └──────────┘  └──────────┘       │             │
│  │         ▲            ▲            ▲              │             │
│  │         │            │            │              │             │
│  │         └─── ТОЛЬКО из Private Subnet ────       │             │
│  │                      │                           │             │
│  │  ┌──────────────────────────────────────────┐    │             │
│  │  │           APPLICATION SUBNET             │    │             │
│  │  │                                          │    │             │
│  │  │  ┌──────────┐  ┌──────────┐              │    │             │
│  │  │  │ Service A│  │ Service B│              │    │             │
│  │  │  │ Producer │  │ Consumer │              │    │             │
│  │  │  └──────────┘  └──────────┘              │    │             │
│  │  └──────────────────────────────────────────┘    │             │
│  │                                                  │             │
│  └─────────────────────────────────────────────────┘             │
│                                                                  │
│  ┌─────────────────────────────────────────────────┐             │
│  │              PUBLIC SUBNET                       │             │
│  │                                                  │             │
│  │  ┌──────────┐                                    │             │
│  │  │ Load     │ ← доступ из интернета              │             │
│  │  │ Balancer │                                    │             │
│  │  └──────────┘                                    │             │
│  │                                                  │             │
│  │  Kafka НЕ доступна из Public Subnet! ❌           │             │
│  └─────────────────────────────────────────────────┘             │
│                                                                  │
│  Firewall Rules:                                                 │
│  ✅ Broker ↔ Broker: разрешить 9093 (inter-broker)               │
│  ✅ App Subnet → Broker: разрешить 9093 (client connections)     │
│  ❌ Public Subnet → Broker: ЗАПРЕТИТЬ ВСЁ                        │
│  ❌ Internet → Broker: ЗАПРЕТИТЬ ВСЁ                             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Audit Logging (журнал аудита)

Аудит-логи позволяют отслеживать **кто, когда и что делал** в кластере. Это критически важно для расследования инцидентов безопасности и compliance.

properties

```
# ════════════════════════════════════════════════════════════
# server.properties — настройка аудит-логирования
# ════════════════════════════════════════════════════════════

# Kafka логирует все авторизационные решения через стандартный logger
# Настройте log4j для вывода в отдельный файл:

# log4j.properties:
log4j.logger.kafka.authorizer.logger=INFO, authorizerAppender
log4j.additivity.kafka.authorizer.logger=false

log4j.appender.authorizerAppender=org.apache.log4j.RollingFileAppender
log4j.appender.authorizerAppender.File=/var/log/kafka/kafka-authorizer.log
log4j.appender.authorizerAppender.MaxFileSize=100MB
log4j.appender.authorizerAppender.MaxBackupIndex=10
log4j.appender.authorizerAppender.layout=org.apache.log4j.PatternLayout
log4j.appender.authorizerAppender.layout.ConversionPattern=[%d{ISO8601}] %p %m (%c)%n
```

**Примеры записей аудит-лога:**

text

```
# Успешная авторизация:
[2024-01-15T10:30:00.123] INFO Principal=User:order-service
  is Allowed Operation=Write from host=192.168.1.100
  on resource=Topic:LITERAL:orders (kafka.authorizer.logger)

# Неудачная попытка (DENY):
[2024-01-15T10:30:01.456] INFO Principal=User:intern
  is Denied Operation=Read from host=192.168.1.200
  on resource=Topic:LITERAL:payments (kafka.authorizer.logger)

# Попытка без ACL:
[2024-01-15T10:30:02.789] INFO Principal=User:unknown-service
  is Denied Operation=Write from host=10.0.0.50
  on resource=Topic:LITERAL:orders
  No matching ACL found (kafka.authorizer.logger)
```

### Итоговый Security Checklist для Production

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║              PRODUCTION SECURITY CHECKLIST                           ║
║                                                                      ║
║  ═══ АУТЕНТИФИКАЦИЯ ═══                                              ║
║  ☐ SASL/SCRAM или mTLS для всех клиентов                            ║
║  ☐ Отдельные учётные записи для каждого сервиса                     ║
║  ☐ Отключён анонимный доступ                                         ║
║  ☐ Super users минимизированы                                        ║
║  ☐ Пароли хранятся в Vault/KMS (не в конфигах!)                      ║
║                                                                      ║
║  ═══ АВТОРИЗАЦИЯ (ACL) ═══                                           ║
║  ☐ ACL включены для всех ресурсов                                    ║
║  ☐ allow.everyone.if.no.acl.found=false                              ║
║  ☐ Принцип наименьших привилегий (minimal access)                    ║
║  ☐ Регулярный аудит ACL                                              ║
║                                                                      ║
║  ═══ ШИФРОВАНИЕ ═══                                                  ║
║  ☐ SSL/TLS для всех соединений (клиент↔брокер, брокер↔брокер)       ║
║  ☐ Только TLS 1.2+ (отключить TLS 1.0, 1.1)                        ║
║  ☐ Шифрование дисков (LUKS / облачное)                               ║
║  ☐ End-to-end шифрование для конфиденциальных данных                 ║
║  ☐ Регулярная ротация сертификатов (каждые 90 дней)                  ║
║                                                                      ║
║  ═══ СЕТЬ ═══                                                        ║
║  ☐ Kafka брокеры в private subnet                                    ║
║  ☐ Firewall: только необходимые порты (9092/9093)                    ║
║  ☐ Firewall: только доверенные IP-адреса                             ║
║  ☐ Нет прямого доступа из интернета                                  ║
║  ☐ VPN/PrivateLink для удалённого доступа                            ║
║                                                                      ║
║  ═══ МОНИТОРИНГ ═══                                                  ║
║  ☐ Audit logging включён и сохраняется                               ║
║  ☐ Алерт на неудачные попытки аутентификации                         ║
║  ☐ Алерт на неавторизованный доступ                                  ║
║  ☐ Мониторинг сертификатов (срок истечения)                          ║
║  ☐ Централизованное хранение логов (ELK, Splunk)                     ║
║                                                                      ║
║  ═══ ОПЕРАЦИОННАЯ БЕЗОПАСНОСТЬ ═══                                   ║
║  ☐ Регулярные обновления Kafka (security patches)                    ║
║  ☐ Бэкапы конфигурации и ACL                                        ║
║  ☐ Документированные процедуры восстановления                        ║
║  ☐ Penetration testing                                               ║
║  ☐ Compliance audit (GDPR, PCI DSS, SOC 2)                          ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```