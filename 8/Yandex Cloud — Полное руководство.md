## 1. Основные сервисы Yandex Cloud

### Введение в Yandex Cloud

**Yandex Cloud** — это российская публичная облачная платформа от компании Yandex, предоставляющая IaaS, PaaS и SaaS сервисы. Это крупнейший российский cloud-провайдер, который позволяет хранить данные на территории РФ и соответствует требованиям российского законодательства (152-ФЗ, GDPR-аналоги).

**Ключевые особенности:**

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                   ПОЧЕМУ YANDEX CLOUD?                               ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  ✅ Данные хранятся в России (соответствие 152-ФЗ)                   ║
║  ✅ Поддержка на русском языке 24/7                                  ║
║  ✅ Оплата в рублях (нет валютных рисков)                            ║
║  ✅ Интеграция с экосистемой Yandex (Метрика, Директ, Tracker)       ║
║  ✅ SLA до 99.95% для критичных сервисов                             ║
║  ✅ Соответствие стандартам: ISO 27001, PCI DSS                      ║
║  ✅ Нет санкционных рисков (российская компания)                     ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### Основные категории сервисов

text

```
┌────────────────────────────────────────────────────────────────────┐
│                    COMPUTE (Вычисления)                            │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Compute Cloud (Виртуальные машины)                                │
│  ══════════════════════════════════════                            │
│  • Виртуальные машины на базе KVM                                  │
│  • Типы: standard, memory-optimized, compute-optimized, GPU        │
│  • ОС: Linux (Ubuntu, CentOS, Debian), Windows Server              │
│  • Прерываемые VM (preemptible) — скидка до 50%                    │
│                                                                    │
│  Пример:                                                           │
│  • s2.micro: 2 vCPU, 8 GB RAM — ~1500 ₽/месяц                      │
│  • s2.medium: 8 vCPU, 32 GB RAM — ~6000 ₽/месяц                    │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  Serverless Containers                                             │
│  ══════════════════════                                            │
│  • Запуск Docker-контейнеров БЕЗ управления инфраструктурой        │
│  • Автомасштабирование 0 → N                                       │
│  • Оплата за фактическое время выполнения                          │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  Cloud Functions (Serverless функции)                              │
│  ═══════════════════════════════════                               │
│  • Функции на Node.js, Python, Go, PHP, Java, .NET, Bash          │
│  • Триггеры: HTTP, таймер, очередь сообщений, Object Storage      │
│  • Оплата за количество вызовов и время выполнения                 │
│                                                                    │
├────────────────────────────────────────────────────────────────────┤
│                    STORAGE (Хранение данных)                       │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Object Storage (S3-совместимое хранилище)                         │
│  ══════════════════════════════════════════                        │
│  • Аналог Amazon S3                                                │
│  • Безлимитное хранилище файлов                                    │
│  • Классы хранения: standard, cold, ice (архивное)                 │
│  • API совместим с AWS S3 (можно использовать AWS SDK)             │
│  • Цена: от 1.20 ₽/GB/месяц (standard), 0.50 ₽/GB (cold)          │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  Compute Cloud Disks                                               │
│  ════════════════════                                              │
│  • SSD (network-ssd): быстрые диски для VM                         │
│  • HDD (network-hdd): дешёвые диски                                │
│  • Non-replicated SSD: максимальная производительность             │
│  • Snapshot'ы для бэкапов                                          │
│                                                                    │
├────────────────────────────────────────────────────────────────────┤
│                 DATABASES (Базы данных)                            │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Managed Service for PostgreSQL                                    │
│  • Полностью управляемая PostgreSQL                                │
│  • Автоматические бэкапы, репликация, failover                     │
│  • Версии: 11, 12, 13, 14, 15, 16                                  │
│                                                                    │
│  Managed Service for MySQL                                         │
│  • Управляемая MySQL 5.7, 8.0                                      │
│                                                                    │
│  Managed Service for ClickHouse                                    │
│  • Аналитическая колоночная СУБД                                   │
│  • Для OLAP (аналитики больших данных)                             │
│                                                                    │
│  Managed Service for MongoDB                                       │
│  • NoSQL документная БД                                            │
│                                                                    │
│  Managed Service for Redis                                         │
│  • In-memory кеш и key-value БД                                    │
│                                                                    │
│  Managed Service for Elasticsearch                                 │
│  • Поиск и аналитика логов                                         │
│                                                                    │
│  Managed Service for Kafka                                         │
│  • Управляемая Apache Kafka                                        │
│  • Для event streaming                                             │
│                                                                    │
├────────────────────────────────────────────────────────────────────┤
│                  NETWORKING (Сети)                                 │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Virtual Private Cloud (VPC)                                       │
│  ════════════════════════════                                      │
│  • Изолированная виртуальная сеть                                  │
│  • Подсети, таблицы маршрутизации                                  │
│  • Security Groups (firewall)                                      │
│  • NAT Gateway, NAT-инстансы                                       │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  Application Load Balancer                                         │
│  ══════════════════════════                                        │
│  • L7 балансировщик нагрузки (HTTP/HTTPS)                          │
│  • Терминация TLS                                                  │
│  • Маршрутизация по хостам, path, headers                          │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  Network Load Balancer                                             │
│  ══════════════════════                                            │
│  • L4 балансировщик (TCP/UDP)                                      │
│  • Для высокой производительности                                  │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  Cloud DNS                                                         │
│  ════════                                                          │
│  • Управление DNS-зонами                                           │
│  • Публичные и внутренние зоны                                     │
│                                                                    │
├────────────────────────────────────────────────────────────────────┤
│              CONTAINERS & ORCHESTRATION                            │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Managed Service for Kubernetes                                    │
│  ═══════════════════════════════════                               │
│  • Полностью управляемый Kubernetes (control plane от Yandex)      │
│  • Kubernetes версии: 1.24, 1.25, 1.26, 1.27                       │
│  • Auto-scaling: Node Groups, Horizontal Pod Autoscaler            │
│  • Интеграция с Container Registry, Load Balancer                  │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  Container Registry                                                │
│  ═══════════════════                                               │
│  • Private Docker-репозиторий                                      │
│  • Vulnerability scanning (сканирование уязвимостей образов)       │
│  • Lifecycle policies (автоудаление старых образов)                │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  Serverless Containers                                             │
│  ══════════════════════                                            │
│  • Запуск контейнеров без K8s                                      │
│  • Автомасштабирование, оплата по факту                            │
│                                                                    │
├────────────────────────────────────────────────────────────────────┤
│                   DATA & ANALYTICS                                 │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  DataProc (Managed Hadoop/Spark)                                   │
│  • Кластеры Apache Hadoop, Spark, Hive                            │
│  • Big Data обработка                                              │
│                                                                    │
│  Data Transfer                                                     │
│  • Миграция данных между БД                                        │
│  • Поддержка PostgreSQL, MySQL, MongoDB, ClickHouse, S3           │
│                                                                    │
│  DataLens (BI & Visualization)                                     │
│  • Визуализация данных (аналог Tableau)                            │
│  • Дашборды, графики, отчёты                                       │
│  • Подключение к PostgreSQL, ClickHouse, MySQL, CSV, Google Sheets │
│                                                                    │
├────────────────────────────────────────────────────────────────────┤
│                   SECURITY & MANAGEMENT                            │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Identity and Access Management (IAM)                              │
│  • Управление доступом                                             │
│  • Роли, пользователи, сервисные аккаунты                          │
│  • Федерация (SAML, Active Directory)                              │
│                                                                    │
│  Key Management Service (KMS)                                      │
│  • Управление криптографическими ключами                           │
│  • Шифрование данных                                               │
│                                                                    │
│  Certificate Manager                                               │
│  • Управление TLS/SSL сертификатами                                │
│  • Let's Encrypt интеграция                                        │
│                                                                    │
│  Audit Trails                                                      │
│  • Логирование всех действий в облаке                              │
│  • Кто, что, когда делал                                           │
│                                                                    │
│  Monitoring                                                        │
│  • Мониторинг метрик ресурсов                                      │
│  • Алерты, дашборды                                                │
│                                                                    │
│  Logging                                                           │
│  • Централизованное хранение логов                                 │
│  • Интеграция с Compute Cloud, Kubernetes, Functions               │
│                                                                    │
├────────────────────────────────────────────────────────────────────┤
│                 AI & MACHINE LEARNING                              │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Yandex SpeechKit                                                  │
│  • Распознавание речи (speech-to-text)                             │
│  • Синтез речи (text-to-speech)                                    │
│                                                                    │
│  Yandex Translate                                                  │
│  • Машинный перевод (90+ языков)                                   │
│                                                                    │
│  Yandex Vision                                                     │
│  • Распознавание текста на изображениях (OCR)                      │
│  • Классификация изображений                                       │
│  • Детекция лиц                                                    │
│                                                                    │
│  DataSphere (Managed Jupyter)                                      │
│  • Jupyter notebooks в облаке                                      │
│  • GPU/TPU для ML                                                  │
│  • Интеграция с Object Storage                                     │
│                                                                    │
├────────────────────────────────────────────────────────────────────┤
│                 DEVELOPER TOOLS                                    │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Container Registry                                                │
│  • Docker-образы                                                   │
│                                                                    │
│  Cloud Functions                                                   │
│  • Serverless функции                                              │
│                                                                    │
│  API Gateway                                                       │
│  • HTTP API для serverless-приложений                              │
│  • OpenAPI спецификация                                            │
│                                                                    │
│  Message Queue                                                     │
│  • Очереди сообщений (совместим с AWS SQS API)                     │
│                                                                    │
│  IoT Core                                                          │
│  • MQTT брокер для IoT-устройств                                   │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## 2. Сервисы для работы с контейнерами

### Обзор контейнерных сервисов

Yandex Cloud предоставляет **три основных сервиса** для работы с контейнерами, каждый для разных сценариев:

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                  КОНТЕЙНЕРНЫЕ СЕРВИСЫ                                ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  1. MANAGED SERVICE FOR KUBERNETES                                   ║
║     ══════════════════════════════════                               ║
║     Полноценный Kubernetes для сложных приложений                    ║
║                                                                      ║
║  2. SERVERLESS CONTAINERS                                            ║
║     ════════════════════════                                         ║
║     Запуск контейнеров БЕЗ управления инфраструктурой               ║
║                                                                      ║
║  3. CONTAINER REGISTRY                                               ║
║     ══════════════════════                                           ║
║     Хранилище Docker-образов                                         ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

### 1. Managed Service for Kubernetes

**Полностью управляемый Kubernetes** — Yandex управляет control plane (master-нодами), вы управляете worker-нодами и приложениями.

text

```
┌────────────────────────────────────────────────────────────────────┐
│          MANAGED SERVICE FOR KUBERNETES                            │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  CONTROL PLANE (Управляется Yandex)                          │  │
│  │  • Kubernetes API Server                                     │  │
│  │  • etcd                                                      │  │
│  │  • Scheduler                                                 │  │
│  │  • Controller Manager                                        │  │
│  │  • Автоматические обновления                                 │  │
│  │  • HA (3 мастера в разных зонах доступности)                 │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                            ▼                                       │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  NODE GROUPS (Управляете вы)                                 │  │
│  │                                                              │  │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐             │  │
│  │  │  Worker 1  │  │  Worker 2  │  │  Worker 3  │             │  │
│  │  │            │  │            │  │            │             │  │
│  │  │  Pods →    │  │  Pods →    │  │  Pods →    │             │  │
│  │  │ [App][App] │  │ [App][App] │  │ [App][App] │             │  │
│  │  └────────────┘  └────────────┘  └────────────┘             │  │
│  │                                                              │  │
│  │  • Autoscaling (CPU/memory-based)                            │  │
│  │  • Rolling updates                                           │  │
│  │  • GPU nodes (для ML)                                        │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Ключевые возможности:**

text

```
✅ ПРЕИМУЩЕСТВА:

• Yandex управляет control plane (не нужно настраивать etcd, API server)
• Автоматические обновления Kubernetes
• High Availability (мастера в 3 зонах доступности)
• Интеграция с другими сервисами:
  - Container Registry (приватный Docker registry)
  - Network Load Balancer (LoadBalancer Service)
  - Persistent Volumes (network-ssd диски)
• Monitoring и Logging (интеграция с Yandex Monitoring)
• Auto-scaling:
  - Horizontal Pod Autoscaler (HPA)
  - Cluster Autoscaler (добавление/удаление нод)
• GPU nodes (для ML/DL задач)
• Версии Kubernetes: 1.24, 1.25, 1.26, 1.27


📌 КОГДА ИСПОЛЬЗОВАТЬ:

✅ Сложные микросервисные приложения
✅ Stateful приложения (БД, очереди)
✅ Нужен полный контроль над Kubernetes
✅ Требуется кастомная конфигурация (CustomResourceDefinitions, Operators)
✅ CI/CD pipeline (GitOps, ArgoCD, Flux)
✅ Команда знает Kubernetes
```

**Создание кластера:**

Bash

```
# Через CLI
yc managed-kubernetes cluster create \
  --name my-k8s-cluster \
  --zone ru-central1-a \
  --network-name default \
  --service-account-name k8s-sa \
  --node-service-account-name k8s-node-sa \
  --release-channel regular \
  --version 1.27

# Создание группы узлов (node group)
yc managed-kubernetes node-group create \
  --name my-node-group \
  --cluster-name my-k8s-cluster \
  --platform standard-v2 \
  --cores 4 \
  --memory 16 \
  --core-fraction 100 \
  --disk-type network-ssd \
  --disk-size 64 \
  --fixed-size 3 \
  --auto-upgrade \
  --auto-repair

# Получить credentials для kubectl
yc managed-kubernetes cluster get-credentials my-k8s-cluster --external

# Проверка
kubectl get nodes
```

---

### 2. Serverless Containers

**Serverless Containers** — запуск Docker-контейнеров **БЕЗ управления** серверами, Kubernetes или любой инфраструктурой. Аналог AWS Fargate + AWS Lambda для контейнеров.

text

```
┌────────────────────────────────────────────────────────────────────┐
│                   SERVERLESS CONTAINERS                            │
│                                                                    │
│  ВЫ:                                                               │
│  1. Собираете Docker-образ                                         │
│  2. Пушите в Container Registry                                    │
│  3. Создаёте контейнер в Serverless Containers                     │
│                                                                    │
│  YANDEX:                                                           │
│  • Автоматически запускает контейнеры по запросам                  │
│  • Автомасштабирование 0 → N                                       │
│  • Балансировка нагрузки                                           │
│  • Мониторинг                                                      │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                                                              │  │
│  │  HTTP запрос → [Serverless Container Instance]               │  │
│  │                                                              │  │
│  │  Нет запросов → 0 инстансов (платите $0)                    │  │
│  │  1000 RPS     → автоматически N инстансов                   │  │
│  │                                                              │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ОПЛАТА:                                                           │
│  • За количество вызовов                                           │
│  • За GB-секунды (память × время выполнения)                       │
│  • За исходящий трафик                                             │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Характеристики:**

text

```
✅ ПРЕИМУЩЕСТВА:

• Нет управления инфраструктурой (ВООБЩЕ)
• Автомасштабирование (0 → тысячи инстансов за секунды)
• Оплата ТОЛЬКО за фактическое использование
• Встроенный HTTPS endpoint
• Интеграция с API Gateway
• Blue/Green deployments (версионирование)
• Холодный старт: ~1-3 секунды


📌 КОГДА ИСПОЛЬЗОВАТЬ:

✅ Веб-приложения с переменной нагрузкой
✅ API backend'ы
✅ Webhook'и (обработка событий от внешних сервисов)
✅ Scheduled tasks (по расписанию)
✅ Прототипы / MVP (быстрый запуск)
✅ Микросервисы без сложной логики оркестрации

❌ НЕ подходит:
  • Stateful приложения (БД внутри контейнера)
  • Длительные процессы (> 10 минут)
  • WebSocket (долгие соединения)
```

**Создание serverless контейнера:**

Bash

```
# 1. Собрать и запушить образ в Container Registry
docker build -t cr.yandex/crp9ftr22d26h0sa3jsj/my-app:v1 .
docker push cr.yandex/crp9ftr22d26h0sa3jsj/my-app:v1

# 2. Создать serverless контейнер
yc serverless container create \
  --name my-serverless-app

# 3. Создать ревизию (версию)
yc serverless container revision deploy \
  --container-name my-serverless-app \
  --image cr.yandex/crp9ftr22d26h0sa3jsj/my-app:v1 \
  --memory 512M \
  --cores 1 \
  --execution-timeout 10s \
  --service-account-id <service-account-id>

# 4. Получить endpoint
yc serverless container get my-serverless-app

# Вывод:
# https://bba3fva6ka5g********.containers.yandexcloud.net/

# Тестирование
curl https://bba3fva6ka5g********.containers.yandexcloud.net/
```

**Пример Dockerfile для Serverless Containers:**

Dockerfile

```
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# ВАЖНО: приложение должно слушать на порту 8080
# и запускаться БЕЗ CMD/ENTRYPOINT (или через Flask/Gunicorn)
CMD ["gunicorn", "app:app", "--bind", "0.0.0.0:8080", "--workers", "1"]
```

---

### 3. Container Registry

**Container Registry** — это приватный **Docker-репозиторий** для хранения и управления Docker-образами в Yandex Cloud.

text

```
┌────────────────────────────────────────────────────────────────────┐
│                    CONTAINER REGISTRY                              │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  REGISTRY: cr.yandex/<registry-id>/                          │  │
│  │                                                              │  │
│  │  ├── my-app:v1.0                                             │  │
│  │  ├── my-app:v1.1                                             │  │
│  │  ├── my-app:latest                                           │  │
│  │  ├── backend:prod-2024-01-15                                 │  │
│  │  └── frontend:dev                                            │  │
│  │                                                              │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ВОЗМОЖНОСТИ:                                                      │
│  • Приватное хранилище (доступ по IAM)                             │
│  • Vulnerability scanning (сканирование на уязвимости)             │
│  • Lifecycle policies (автоудаление старых образов)                │
│  • Docker-совместимость (docker push/pull)                         │
│  • Интеграция с Kubernetes, Serverless Containers                  │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Работа с Container Registry:**

Bash

```
# 1. Создать registry
yc container registry create --name my-registry

# Получить ID registry
yc container registry list
# +----------------------+-------------+
# |          ID          |    NAME     |
# +----------------------+-------------+
# | crp9ftr22d26h0sa3jsj | my-registry |
# +----------------------+-------------+

# 2. Настроить Docker для работы с Yandex Container Registry
yc container registry configure-docker

# Или вручную
docker login \
  --username iam \
  --password $(yc iam create-token) \
  cr.yandex

# 3. Тегировать образ
docker tag my-app:latest cr.yandex/crp9ftr22d26h0sa3jsj/my-app:v1.0

# 4. Запушить образ
docker push cr.yandex/crp9ftr22d26h0sa3jsj/my-app:v1.0

# 5. Список образов в registry
yc container image list --registry-name my-registry

# 6. Сканирование на уязвимости
yc container image scan cr.yandex/crp9ftr22d26h0sa3jsj/my-app:v1.0

# Результат сканирования
yc container image list-scan-results

# 7. Lifecycle policy (автоудаление старых образов)
yc container repository lifecycle-policy create \
  --repository-name my-registry/my-app \
  --name delete-old \
  --description "Keep only 10 latest images" \
  --rules '[{"description":"keep 10","expire_period":"0d","tag_regexp":".*","retained_top":10}]'
```

**Использование в Kubernetes:**

YAML

```
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      # ВАЖНО: Указать imagePullSecrets для приватного registry
      imagePullSecrets:
      - name: yc-registry-secret
      containers:
      - name: app
        image: cr.yandex/crp9ftr22d26h0sa3jsj/my-app:v1.0
        ports:
        - containerPort: 8080
```

Bash

```
# Создание secret для pull образов из приватного registry
kubectl create secret docker-registry yc-registry-secret \
  --docker-server=cr.yandex \
  --docker-username=iam \
  --docker-password=$(yc iam create-token)
```

---

### Container-Optimized Image (COI)

**Container-Optimized Image** — это специальная **легковесная ОС**, оптимизированная для запуска **Docker-контейнеров** на виртуальных машинах Compute Cloud.

text

```
╔══════════════════════════════════════════════════════════════════════╗
║             ЧТО ТАКОЕ CONTAINER-OPTIMIZED IMAGE?                     ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  Container-Optimized Image (COI) — это минималистичный Linux-образ   ║
║  на базе Ubuntu, который содержит ТОЛЬКО:                            ║
║                                                                      ║
║  • Docker Engine (предустановлен)                                    ║
║  • Минимальный набор системных утилит                                ║
║  • Автоматическое обновление системы                                 ║
║  • Оптимизация для быстрого запуска контейнеров                      ║
║                                                                      ║
║  НЕ содержит:                                                        ║
║  • Лишних пакетов (меньше attack surface)                            ║
║  • GUI                                                               ║
║  • Ненужных сервисов                                                 ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Зачем нужен COI:**

text

```
✅ ПРЕИМУЩЕСТВА:

1. Быстрый старт
   • Образ легковесный → быстрая загрузка VM
   • Docker уже установлен → контейнер стартует сразу

2. Безопасность
   • Минимум пакетов → меньше уязвимостей
   • Автоматические обновления безопасности

3. Простота
   • Не нужно настраивать Docker вручную
   • Декларативная спецификация (через cloud-config)

4. Экономия ресурсов
   • Меньше RAM и CPU для ОС
   • Больше ресурсов для контейнера


📌 КОГДА ИСПОЛЬЗОВАТЬ:

✅ Нужно запустить ONE контейнер на VM
✅ Простые сценарии (не нужен Kubernetes)
✅ Быстрый деплой (меньше настройки)
✅ CI/CD агенты (Jenkins, GitLab Runner в Docker)
✅ Batch jobs (запустить контейнер, выполнить задачу, выключить VM)

❌ НЕ подходит:
  • Множество контейнеров с оркестрацией (используйте Kubernetes)
  • Нужны кастомные системные пакеты
```

**Создание VM с COI:**

Bash

```
# Создать VM с Container-Optimized Image
yc compute instance create-with-container \
  --name my-container-vm \
  --zone ru-central1-a \
  --platform standard-v2 \
  --cores 2 \
  --memory 4 \
  --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
  --create-boot-disk size=30 \
  --container-image cr.yandex/crp9ftr22d26h0sa3jsj/my-app:v1.0 \
  --container-name my-app \
  --container-env KEY1=value1,KEY2=value2 \
  --container-restart-policy always \
  --service-account-name my-sa

# Параметры:
# --container-image        — Docker-образ для запуска
# --container-name         — имя контейнера
# --container-env          — переменные окружения
# --container-restart-policy — политика перезапуска (always, on-failure, never)
```

**Спецификация через cloud-config (Docker Compose style):**

YAML

```
#cloud-config

users:
  - name: yc-user
    groups: sudo
    shell: /bin/bash
    sudo: ALL=(ALL) NOPASSWD:ALL

write_files:
  # Docker Compose спецификация
  - path: /var/lib/yandex/docker-compose.yaml
    permissions: '0644'
    content: |
      version: '3.7'
      services:
        web:
          image: cr.yandex/crp9ftr22d26h0sa3jsj/my-app:v1.0
          container_name: my-app
          restart: always
          ports:
            - "80:8080"
          environment:
            - DATABASE_URL=postgres://...
            - REDIS_URL=redis://...
          logging:
            driver: "json-file"
            options:
              max-size: "10m"
              max-file: "3"

runcmd:
  # Запуск Docker Compose при старте VM
  - [ docker-compose, -f, /var/lib/yandex/docker-compose.yaml, up, -d ]
```

Bash

```
# Создание VM с custom cloud-config
yc compute instance create \
  --name my-vm \
  --zone ru-central1-a \
  --platform standard-v2 \
  --cores 2 \
  --memory 4 \
  --create-boot-disk image-family=container-optimized-image,size=30 \
  --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
  --metadata-from-file user-data=cloud-config.yaml
```

---

## 3. IAM (Identity and Access Management)

### Что такое IAM в Yandex Cloud?

**IAM (Identity and Access Management)** — это сервис для управления **доступом** к ресурсам Yandex Cloud. Он определяет **кто** (субъект) может выполнять **какие действия** (роли) над **какими ресурсами** (объекты).

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                         IAM КОНЦЕПЦИИ                                ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  СУБЪЕКТЫ (КТО):                                                     ║
║  • Пользовательский аккаунт (Yandex ID)                              ║
║  • Сервисный аккаунт (для приложений)                                ║
║  • Федеративный пользователь (SAML, Active Directory)                ║
║  • Системная группа (allAuthenticatedUsers, allUsers)                ║
║                                                                      ║
║  РОЛИ (ЧТО МОЖНО ДЕЛАТЬ):                                            ║
║  • Набор разрешений (permissions)                                    ║
║  • Примитивные роли: admin, editor, viewer                           ║
║  • Сервис-специфичные роли: compute.admin, storage.editor            ║
║                                                                      ║
║  РЕСУРСЫ (НАД ЧЕМ):                                                  ║
║  • Облако (cloud)                                                    ║
║  • Каталог (folder)                                                  ║
║  • Конкретный ресурс (VM, bucket, кластер K8s)                       ║
║                                                                      ║
║  ПРИНЦИП:                                                            ║
║  Субъект + Роль + Ресурс = Разрешение на действие                   ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

### Типы ролей в Yandex Cloud

text

```
┌────────────────────────────────────────────────────────────────────┐
│                      ТИПЫ РОЛЕЙ                                    │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  1. ПРИМИТИВНЫЕ РОЛИ (Primitive Roles)                             │
│     ════════════════════════════════                               │
│                                                                    │
│     viewer (Наблюдатель)                                           │
│     • Только ЧТЕНИЕ ресурсов                                       │
│     • Просмотр списков, метаданных                                 │
│     • НЕ может создавать/изменять/удалять                          │
│                                                                    │
│     editor (Редактор)                                              │
│     • Всё из viewer +                                              │
│     • СОЗДАВАТЬ, ИЗМЕНЯТЬ, УДАЛЯТЬ ресурсы                         │
│     • НЕ может управлять доступом (IAM)                            │
│                                                                    │
│     admin (Администратор)                                          │
│     • Всё из editor +                                              │
│     • Управление правами доступа (назначать роли)                  │
│     • Полный контроль над ресурсом                                 │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  2. СЕРВИС-СПЕЦИФИЧНЫЕ РОЛИ                                        │
│     ════════════════════════════                                   │
│                                                                    │
│     Compute Cloud:                                                 │
│     • compute.admin      — полное управление VM                    │
│     • compute.editor     — создание/изменение VM                   │
│     • compute.viewer     — просмотр VM                             │
│     • compute.images.user — использование образов                  │
│                                                                    │
│     Object Storage:                                                │
│     • storage.admin      — управление бакетами                     │
│     • storage.editor     — запись объектов                         │
│     • storage.viewer     — чтение объектов                         │
│     • storage.uploader   — только загрузка файлов                  │
│                                                                    │
│     Managed Kubernetes:                                            │
│     • k8s.admin          — управление кластером                    │
│     • k8s.editor         — создание/изменение                      │
│     • k8s.viewer         — просмотр                                │
│     • k8s.cluster-api.cluster-admin — kubectl доступ               │
│                                                                    │
│     Container Registry:                                            │
│     • container-registry.admin   — управление registry             │
│     • container-registry.images.puller — pull образов              │
│     • container-registry.images.pusher — push образов              │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  3. ПУБЛИЧНЫЕ РОЛИ                                                 │
│     ══════════════                                                 │
│                                                                    │
│     allAuthenticatedUsers                                          │
│     • Все аутентифицированные пользователи Yandex                  │
│     • Использовать ОСТОРОЖНО (потенциально опасно)                 │
│                                                                    │
│     allUsers                                                       │
│     • ВСЕ пользователи (включая анонимных)                         │
│     • Только для публичных ресурсов (статика в S3)                 │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Назначение ролей:**

Bash

```
# Назначить роль пользователю на каталог
yc resource-manager folder add-access-binding my-folder \
  --role editor \
  --subject userAccount:aje6o61dvog2h6g9a33s

# Назначить роль сервисному аккаунту
yc resource-manager folder add-access-binding my-folder \
  --role compute.admin \
  --subject serviceAccount:aje1234567890abcdef

# Назначить роль на конкретный ресурс (VM)
yc compute instance add-access-binding my-vm \
  --role compute.viewer \
  --subject userAccount:aje6o61dvog2h6g9a33s

# Список ролей на каталоге
yc resource-manager folder list-access-bindings my-folder

# Удалить роль
yc resource-manager folder remove-access-binding my-folder \
  --role editor \
  --subject userAccount:aje6o61dvog2h6g9a33s
```

---

### Сервисные аккаунты

**Сервисный аккаунт** — это специальный тип аккаунта, предназначенный для **приложений** и **автоматизации**, а не для людей. Это аналог AWS IAM Roles или GCP Service Accounts.

text

```
╔══════════════════════════════════════════════════════════════════════╗
║               ЗАЧЕМ НУЖНЫ СЕРВИСНЫЕ АККАУНТЫ?                        ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  ПРОБЛЕМА:                                                           ║
║  Приложение на VM нужно получить доступ к Object Storage.           ║
║                                                                      ║
║  ❌ ПЛОХОЕ РЕШЕНИЕ:                                                  ║
║     Хардкодить access key в коде:                                    ║
║     aws_access_key = "AKIA..."                                       ║
║     aws_secret_key = "wJal..."                                       ║
║                                                                      ║
║     Проблемы:                                                        ║
║     • Ключи в коде → попадут в Git → утечка                          ║
║     • Сложно ротировать                                              ║
║     • Если скомпрометированы → нужно менять везде                    ║
║                                                                      ║
║  ✅ ХОРОШЕЕ РЕШЕНИЕ:                                                 ║
║     Привязать СЕРВИСНЫЙ АККАУНТ к VM:                                ║
║     1. Создать сервисный аккаунт                                     ║
║     2. Назначить ему роль storage.editor                             ║
║     3. Привязать к VM                                                ║
║     4. Приложение на VM автоматически получает credentials           ║
║        через metadata service (БЕЗ хардкода!)                        ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Работа с сервисными аккаунтами:**

Bash

```
# ════════════════════════════════════════════════════════════
# СОЗДАНИЕ СЕРВИСНОГО АККАУНТА
# ════════════════════════════════════════════════════════════

# Создать сервисный аккаунт
yc iam service-account create \
  --name my-sa \
  --description "Service account for my application"

# Получить ID
yc iam service-account get my-sa

# Вывод:
# id: aje1234567890abcdef
# folder_id: b1g12345678901234567
# name: my-sa
# description: Service account for my application


# ════════════════════════════════════════════════════════════
# НАЗНАЧЕНИЕ РОЛЕЙ
# ════════════════════════════════════════════════════════════

# Назначить роль на каталог
yc resource-manager folder add-access-binding my-folder \
  --role storage.editor \
  --subject serviceAccount:aje1234567890abcdef

# Несколько ролей
yc resource-manager folder add-access-binding my-folder \
  --role compute.viewer \
  --subject serviceAccount:aje1234567890abcdef


# ════════════════════════════════════════════════════════════
# ИСПОЛЬЗОВАНИЕ С VM
# ════════════════════════════════════════════════════════════

# Создать VM с привязанным сервисным аккаунтом
yc compute instance create \
  --name my-vm \
  --zone ru-central1-a \
  --platform standard-v2 \
  --cores 2 \
  --memory 4 \
  --create-boot-disk image-family=ubuntu-2004-lts,size=20 \
  --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
  --service-account-name my-sa  # ← Привязка сервисного аккаунта

# Теперь приложение на VM может получить IAM-токен:
# curl -H Metadata-Flavor:Google http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token
```

**Использование в коде (Python):**

Python

```
# app.py на VM с привязанным сервисным аккаунтом
import boto3
import requests

# ════════════════════════════════════════════════════════════
# ПОЛУЧЕНИЕ IAM-ТОКЕНА из metadata service
# ════════════════════════════════════════════════════════════

def get_iam_token():
    """Получить IAM-токен сервисного аккаунта из metadata."""
    metadata_url = "http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token"
    headers = {"Metadata-Flavor": "Google"}
    
    response = requests.get(metadata_url, headers=headers)
    response.raise_for_status()
    
    return response.json()["access_token"]

# ════════════════════════════════════════════════════════════
# ИСПОЛЬЗОВАНИЕ С YANDEX OBJECT STORAGE (S3)
# ════════════════════════════════════════════════════════════

# БЕЗ хардкода credentials!
# Yandex Cloud SDK автоматически получит credentials из metadata
s3 = boto3.client(
    's3',
    endpoint_url='https://storage.yandexcloud.net',
    region_name='ru-central1'
)

# Загрузить файл
s3.upload_file('local_file.txt', 'my-bucket', 'remote_file.txt')

# Скачать файл
s3.download_file('my-bucket', 'remote_file.txt', 'downloaded_file.txt')
```

**Авторизованные ключи (для локальной разработки):**

Bash

```
# ════════════════════════════════════════════════════════════
# СОЗДАНИЕ АВТОРИЗОВАННОГО КЛЮЧА
# ════════════════════════════════════════════════════════════

# Создать ключ для сервисного аккаунта (для использования вне Yandex Cloud)
yc iam key create \
  --service-account-name my-sa \
  --output key.json

# key.json содержит:
# {
#   "id": "ajelj1234567890",
#   "service_account_id": "aje1234567890abcdef",
#   "created_at": "2024-01-15T10:30:00Z",
#   "key_algorithm": "RSA_2048",
#   "public_key": "-----BEGIN PUBLIC KEY-----\n...",
#   "private_key": "-----BEGIN PRIVATE KEY-----\n..."
# }

# Использование в коде (Python)
from yandexcloud import SDK

sdk = SDK(service_account_key=open('key.json').read())
s3 = sdk.client('storage')
```

---

## 4. YC CLI — Command Line Interface

### Что такое yc CLI?

**yc CLI** — это официальный инструмент командной строки для управления ресурсами Yandex Cloud. Аналог `aws cli` для AWS или `gcloud` для GCP.

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                        YC CLI                                        ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  • Управление ВСЕМИ сервисами Yandex Cloud из терминала              ║
║  • Автодополнение команд (Tab)                                       ║
║  • Работа с несколькими профилями (prod, staging, dev)               ║
║  • Вывод в JSON/YAML для автоматизации                               ║
║  • Интерактивный режим для выбора ресурсов                           ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### Установка yc CLI

Bash

```
# ════════════════════════════════════════════════════════════
# LINUX / macOS
# ════════════════════════════════════════════════════════════

# Скачать и установить
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash

# Перезагрузить shell
exec -l $SHELL

# Проверка
yc version


# ════════════════════════════════════════════════════════════
# WINDOWS (PowerShell)
# ════════════════════════════════════════════════════════════

iex (New-Object System.Net.WebClient).DownloadString('https://storage.yandexcloud.net/yandexcloud-yc/install.ps1')


# ════════════════════════════════════════════════════════════
# ОБНОВЛЕНИЕ
# ════════════════════════════════════════════════════════════

yc components update


# ════════════════════════════════════════════════════════════
# АВТОДОПОЛНЕНИЕ (Bash)
# ════════════════════════════════════════════════════════════

echo 'source <(yc completion bash)' >> ~/.bashrc
source ~/.bashrc

# Теперь можно нажимать Tab для автодополнения
yc compute instance create <Tab><Tab>
```

### Инициализация и авторизация

Bash

```
# ════════════════════════════════════════════════════════════
# ПЕРВОНАЧАЛЬНАЯ НАСТРОЙКА
# ════════════════════════════════════════════════════════════

yc init

# Интерактивный процесс:
# 1. Предложит получить OAuth-токен (откроет браузер)
# 2. Выбрать облако (cloud)
# 3. Выбрать каталог (folder)
# 4. Выбрать зону доступности по умолчанию (ru-central1-a)

# После этого создаётся профиль "default" с настройками


# ════════════════════════════════════════════════════════════
# ПОЛУЧЕНИЕ IAM-ТОКЕНА
# ════════════════════════════════════════════════════════════

# Получить временный IAM-токен (действителен 12 часов)
yc iam create-token

# Использование в скриптах
export YC_TOKEN=$(yc iam create-token)


# ════════════════════════════════════════════════════════════
# ПРОСМОТР КОНФИГУРАЦИИ
# ════════════════════════════════════════════════════════════

yc config list

# Вывод:
# token: AQA...
# cloud-id: b1g12345678901234567
# folder-id: b1g98765432109876543
# compute-default-zone: ru-central1-a


# ════════════════════════════════════════════════════════════
# ИЗМЕНЕНИЕ НАСТРОЕК
# ════════════════════════════════════════════════════════════

# Сменить каталог по умолчанию
yc config set folder-id b1g12345678901234567

# Сменить зону по умолчанию
yc config set compute-default-zone ru-central1-b
```

### Базовые команды yc CLI

Bash

```
# ════════════════════════════════════════════════════════════
# СТРУКТУРА КОМАНД
# ════════════════════════════════════════════════════════════

yc <сервис> <ресурс> <действие> [флаги]

# Примеры:
yc compute instance create ...    # Создать VM
yc storage bucket create ...      # Создать S3 bucket
yc managed-kubernetes cluster get ... # Получить инфо о K8s кластере


# ════════════════════════════════════════════════════════════
# ПОЛУЧЕНИЕ ИНФОРМАЦИИ
# ════════════════════════════════════════════════════════════

# Список облаков
yc resource-manager cloud list

# Список каталогов
yc resource-manager folder list

# Список VM
yc compute instance list

# Детальная информация о VM
yc compute instance get my-vm

# Вывод в JSON (для парсинга)
yc compute instance get my-vm --format json

# Вывод в YAML
yc compute instance get my-vm --format yaml


# ════════════════════════════════════════════════════════════
# СОЗДАНИЕ РЕСУРСОВ
# ════════════════════════════════════════════════════════════

# Создать VM
yc compute instance create \
  --name my-vm \
  --zone ru-central1-a \
  --platform standard-v2 \
  --cores 2 \
  --memory 4 \
  --create-boot-disk image-family=ubuntu-2004-lts,size=20 \
  --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
  --ssh-key ~/.ssh/id_rsa.pub

# Создать S3 bucket
yc storage bucket create my-bucket

# Создать сервисный аккаунт
yc iam service-account create --name my-sa


# ════════════════════════════════════════════════════════════
# УПРАВЛЕНИЕ РЕСУРСАМИ
# ════════════════════════════════════════════════════════════

# Запустить VM
yc compute instance start my-vm

# Остановить VM
yc compute instance stop my-vm

# Перезагрузить VM
yc compute instance restart my-vm

# Удалить VM
yc compute instance delete my-vm

# Snapshot диска
yc compute snapshot create \
  --name my-snapshot \
  --disk-name my-disk


# ════════════════════════════════════════════════════════════
# IAM ОПЕРАЦИИ
# ════════════════════════════════════════════════════════════

# Список сервисных аккаунтов
yc iam service-account list

# Создать ключ доступа (для S3 API)
yc iam access-key create --service-account-name my-sa

# Создать авторизованный ключ (для SDK)
yc iam key create --service-account-name my-sa --output key.json

# Назначить роль
yc resource-manager folder add-access-binding my-folder \
  --role editor \
  --subject serviceAccount:aje1234567890


# ════════════════════════════════════════════════════════════
# ФИЛЬТРАЦИЯ И ФОРМАТИРОВАНИЕ ВЫВОДА
# ════════════════════════════════════════════════════════════

# Список VM с фильтром
yc compute instance list --filter "name='test-*'"

# Только определённые поля
yc compute instance list --format "table(name,zone_id,status)"

# С помощью jq (для JSON)
yc compute instance list --format json | jq '.[] | {name: .name, ip: .network_interfaces[0].primary_v4_address.one_to_one_nat.address}'
```

### Профили (работа с несколькими окружениями)

Bash

```
# ════════════════════════════════════════════════════════════
# СОЗДАНИЕ ПРОФИЛЕЙ
# ════════════════════════════════════════════════════════════

# Создать профиль для production
yc config profile create production
yc config set cloud-id b1gprod123456789
yc config set folder-id b1gprodfolder

# Создать профиль для staging
yc config profile create staging
yc config set cloud-id b1gstag123456789
yc config set folder-id b1gstagfolder

# Создать профиль для dev
yc config profile create dev
yc config set cloud-id b1gdev123456789
yc config set folder-id b1gdevfolder


# ════════════════════════════════════════════════════════════
# ПЕРЕКЛЮЧЕНИЕ МЕЖДУ ПРОФИЛЯМИ
# ════════════════════════════════════════════════════════════

# Список профилей
yc config profile list

# Активировать профиль
yc config profile activate production

# Теперь все команды работают с production

# Переключиться на staging
yc config profile activate staging


# ════════════════════════════════════════════════════════════
# ИСПОЛЬЗОВАНИЕ ПРОФИЛЯ ДЛЯ ОДНОЙ КОМАНДЫ
# ════════════════════════════════════════════════════════════

# Выполнить команду с конкретным профилем
yc compute instance list --profile production

# Или через переменную окружения
YC_PROFILE=staging yc compute instance list
```

---

### Авторизация через сервисный аккаунт

Для **автоматизации** (CI/CD, скрипты) нужно авторизоваться **БЕЗ интерактивного получения OAuth-токена**. Используется **авторизованный ключ** сервисного аккаунта.

Bash

```
# ════════════════════════════════════════════════════════════
# ШАГ 1: Создать сервисный аккаунт
# ════════════════════════════════════════════════════════════

yc iam service-account create --name ci-cd-sa

# Назначить роли
yc resource-manager folder add-access-binding my-folder \
  --role admin \
  --subject serviceAccount:$(yc iam service-account get ci-cd-sa --format json | jq -r .id)


# ════════════════════════════════════════════════════════════
# ШАГ 2: Создать авторизованный ключ
# ════════════════════════════════════════════════════════════

yc iam key create \
  --service-account-name ci-cd-sa \
  --output key.json \
  --description "Key for CI/CD automation"

# key.json содержит приватный ключ
# ВАЖНО: Хранить в секретном месте! НЕ коммитить в Git!


# ════════════════════════════════════════════════════════════
# ШАГ 3: Авторизация в yc CLI
# ════════════════════════════════════════════════════════════

# Создать профиль для сервисного аккаунта
yc config profile create sa-profile

# Установить авторизованный ключ
yc config set service-account-key key.json

# Установить cloud и folder
yc config set cloud-id b1g12345678901234567
yc config set folder-id b1g98765432109876543

# Активировать профиль
yc config profile activate sa-profile

# Теперь можно работать от имени сервисного аккаунта!
yc compute instance list


# ════════════════════════════════════════════════════════════
# ИСПОЛЬЗОВАНИЕ В CI/CD (GitLab CI)
# ════════════════════════════════════════════════════════════

# .gitlab-ci.yml
deploy:
  image: yandexcloud/cli:latest
  script:
    # Создать key.json из переменной окружения (GitLab CI variable)
    - echo "$YC_SA_KEY" > key.json
    
    # Авторизоваться
    - yc config profile create sa-profile
    - yc config set service-account-key key.json
    - yc config set cloud-id $YC_CLOUD_ID
    - yc config set folder-id $YC_FOLDER_ID
    
    # Деплой приложения
    - yc compute instance create-with-container ...
    
    # Очистить ключ
    - rm key.json


# ════════════════════════════════════════════════════════════
# АЛЬТЕРНАТИВА: IAM-токен из ключа (для API)
# ════════════════════════════════════════════════════════════

# Получить IAM-токен из авторизованного ключа
yc iam create-token --profile sa-profile

# Использование в скриптах
export YC_TOKEN=$(yc iam create-token --profile sa-profile)

curl -H "Authorization: Bearer $YC_TOKEN" \
  https://compute.api.cloud.yandex.net/compute/v1/instances?folderId=b1g...
```

---

## 5. Организация и Каталог в Yandex Cloud

### Иерархия ресурсов

text

```
┌────────────────────────────────────────────────────────────────────┐
│                 ИЕРАРХИЯ YANDEX CLOUD                              │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  ОРГАНИЗАЦИЯ (Organization)                                  │  │
│  │  • Корневой уровень для компании                             │  │
│  │  • Управление пользователями                                 │  │
│  │  • Федерация (SAML, Active Directory)                        │  │
│  │  • Биллинг аккаунт                                           │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                            │                                       │
│                            ▼                                       │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  ОБЛАКО (Cloud)                                              │  │
│  │  • Изолированное окружение                                   │  │
│  │  • Может быть несколько облаков в организации                │  │
│  │  • Разделение по проектам / командам                         │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                            │                                       │
│                            ▼                                       │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  КАТАЛОГ (Folder)                                            │  │
│  │  • Логическая группа ресурсов                                │  │
│  │  • Изоляция по окружениям (prod, staging, dev)               │  │
│  │  • Разделение по сервисам / микросервисам                    │  │
│  │  • Назначение квот и ролей                                   │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                            │                                       │
│                            ▼                                       │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  РЕСУРСЫ                                                     │  │
│  │  • VM, диски, сети                                           │  │
│  │  • S3 buckets                                                │  │
│  │  • Kubernetes кластеры                                       │  │
│  │  • Базы данных                                               │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

### Организация (Organization)

**Организация** — это **корневая сущность** в Yandex Cloud, которая объединяет **все ресурсы компании**, управляет **пользователями** и **биллингом**.

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                      ЧТО ТАКОЕ ОРГАНИЗАЦИЯ?                          ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  • Представляет ВАШУ КОМПАНИЮ в Yandex Cloud                         ║
║  • Все облака, каталоги, ресурсы принадлежат организации             ║
║  • Единое управление пользователями и правами                        ║
║  • Централизованный биллинг                                          ║
║                                                                      ║
║  Создаётся автоматически при первой регистрации.                     ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Возможности организации:**

text

```
1. УПРАВЛЕНИЕ ПОЛЬЗОВАТЕЛЯМИ
   • Приглашение пользователей по email
   • Назначение ролей на уровне организации
   • Федерация (SAML 2.0, Active Directory)
   • SSO (Single Sign-On)

2. ФЕДЕРАЦИЯ (для корпораций)
   • Интеграция с корпоративным Active Directory
   • Пользователи логинятся через корпоративные credentials
   • Централизованное управление доступом

3. БИЛЛИНГ
   • Один платёжный аккаунт для всей организации
   • Отчёты по расходам
   • Бюджеты и алерты

4. AUDIT
   • Audit Trails для всей организации
   • Кто, что, когда делал
```

**Работа с организацией:**

Bash

```
# Список организаций
yc organization-manager organization list

# Информация об организации
yc organization-manager organization get <organization-id>

# Список пользователей организации
yc organization-manager user list --organization-id <organization-id>

# Назначить роль пользователю на уровне организации
yc organization-manager organization add-access-binding <organization-id> \
  --role organization-manager.admin \
  --subject userAccount:aje6o61dvog2h6g9a33s
```

---

### Каталог (Folder)

**Каталог** — это **логическая группа ресурсов** внутри облака. Это основная единица для **изоляции окружений** и **управления доступом**.

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                    ЗАЧЕМ НУЖНЫ КАТАЛОГИ?                             ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  1. ИЗОЛЯЦИЯ ОКРУЖЕНИЙ                                               ║
║     ═══════════════════════                                          ║
║                                                                      ║
║     Cloud: "my-company"                                              ║
║       ├── Folder: "production"                                       ║
║       │    ├── VM: prod-web-1, prod-web-2                            ║
║       │    ├── Database: prod-postgres                               ║
║       │    └── S3: prod-static-bucket                                ║
║       │                                                              ║
║       ├── Folder: "staging"                                          ║
║       │    ├── VM: stage-web-1                                       ║
║       │    └── Database: stage-postgres                              ║
║       │                                                              ║
║       └── Folder: "development"                                      ║
║            ├── VM: dev-web-1                                         ║
║            └── Database: dev-postgres                                ║
║                                                                      ║
║     Преимущества:                                                    ║
║     ✅ Полная изоляция ресурсов                                      ║
║     ✅ Раздельные права доступа (devs → только dev folder)           ║
║     ✅ Отдельные квоты для каждого окружения                         ║
║     ✅ Легко посчитать стоимость каждого окружения                   ║
║                                                                      ║
║  ────────────────────────────────────────────────────────            ║
║                                                                      ║
║  2. ИЗОЛЯЦИЯ ПО ПРОЕКТАМ / КОМАНДАМ                                  ║
║     ═══════════════════════════════════                              ║
║                                                                      ║
║     Cloud: "enterprise"                                              ║
║       ├── Folder: "team-billing"                                     ║
║       ├── Folder: "team-analytics"                                   ║
║       ├── Folder: "team-ml"                                          ║
║       └── Folder: "team-infrastructure"                              ║
║                                                                      ║
║     Каждая команда управляет СВОИМИ ресурсами.                       ║
║                                                                      ║
║  ────────────────────────────────────────────────────────            ║
║                                                                      ║
║  3. ИЗОЛЯЦИЯ ПО МИКРОСЕРВИСАМ                                        ║
║     ════════════════════════════════                                 ║
║                                                                      ║
║     Cloud: "microservices"                                           ║
║       ├── Folder: "user-service"                                     ║
║       ├── Folder: "order-service"                                    ║
║       ├── Folder: "payment-service"                                  ║
║       └── Folder: "notification-service"                             ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Работа с каталогами:**

Bash

```
# ════════════════════════════════════════════════════════════
# СОЗДАНИЕ КАТАЛОГА
# ════════════════════════════════════════════════════════════

# Создать каталог
yc resource-manager folder create \
  --name production \
  --description "Production environment"

# Получить ID каталога
yc resource-manager folder get production


# ════════════════════════════════════════════════════════════
# СПИСОК КАТАЛОГОВ
# ════════════════════════════════════════════════════════════

yc resource-manager folder list

# Вывод:
# +----------------------+-------------+--------+--------+
# |          ID          |    NAME     | LABELS | STATUS |
# +----------------------+-------------+--------+--------+
# | b1gprod123456789     | production  |        | ACTIVE |
# | b1gstag987654321     | staging     |        | ACTIVE |
# | b1gdev456789012      | development |        | ACTIVE |
# +----------------------+-------------+--------+--------+


# ════════════════════════════════════════════════════════════
# УПРАВЛЕНИЕ ДОСТУПОМ К КАТАЛОГУ
# ════════════════════════════════════════════════════════════

# Назначить роль пользователю на каталог
yc resource-manager folder add-access-binding production \
  --role editor \
  --subject userAccount:aje6o61dvog2h6g9a33s

# Теперь пользователь может создавать/изменять ресурсы
# ТОЛЬКО в каталоге "production"

# Список ролей на каталоге
yc resource-manager folder list-access-bindings production


# ════════════════════════════════════════════────────────────
# КВОТЫ НА УРОВНЕ КАТАЛОГА
# ════════════════════════════════════════════════════════────

# Просмотр квот
yc resource-manager quota list --folder-id b1gprod123456789

# Пример вывода:
# cpu.cores: 20 / 100        (используется 20 из 100 vCPU)
# memory.size: 80GB / 256GB  (используется 80GB из 256GB RAM)
# compute.disks.count: 10 / 50


# ════════════════════════════════════════════════════════════
# ПЕРЕКЛЮЧЕНИЕ МЕЖДУ КАТАЛОГАМИ В CLI
# ════════════════════════════════════════════════════════════

# Установить каталог по умолчанию
yc config set folder-id b1gprod123456789

# Или для одной команды
yc compute instance list --folder-id b1gstag987654321


# ════════════════════════════════════════════════════════════
# УДАЛЕНИЕ КАТАЛОГА
# ════════════════════════════════════════════════════════════

# ОСТОРОЖНО: удаляются ВСЕ ресурсы внутри каталога!
yc resource-manager folder delete production
```

**Best Practices для организации каталогов:**

text

```
┌────────────────────────────────────────────────────────────────────┐
│                  РЕКОМЕНДАЦИИ                                      │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  1. ОТДЕЛЬНЫЕ КАТАЛОГИ ДЛЯ ОКРУЖЕНИЙ                               │
│     • production, staging, development                             │
│     • Раздельные права доступа                                     │
│     • Разные квоты                                                 │
│                                                                    │
│  2. NAMING CONVENTION                                              │
│     • <project>-<environment>                                      │
│     • Пример: myapp-prod, myapp-stage, myapp-dev                   │
│                                                                    │
│  3. ROLE-BASED ACCESS CONTROL (RBAC)                               │
│     • Developers → editor на dev, viewer на staging/prod           │
│     • DevOps     → admin на все каталоги                           │
│     • Auditors   → viewer на все каталоги                          │
│                                                                    │
│  4. TAGGING (метки ресурсов)                                       │
│     • project=myapp, environment=prod, team=backend                │
│     • Помогает отслеживать расходы                                 │
│                                                                    │
│  5. ЦЕНТРАЛИЗОВАННЫЙ МОНИТОРИНГ                                    │
│     • Один Yandex Monitoring дашборд для всех каталогов            │
│     • Централизованные логи (Yandex Logging)                       │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```