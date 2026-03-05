
## 1. Что такое Helm и какую проблему он решает?

### Введение в проблему

Когда вы начинаете работать с Kubernetes, первое с чем сталкиваетесь — это множество YAML-файлов. Даже простое приложение требует создания нескольких ресурсов:

- **Deployment** — описывает как запускать ваше приложение (какой образ, сколько реплик, какие ресурсы)
- **Service** — обеспечивает сетевой доступ к подам
- **ConfigMap** — хранит конфигурацию приложения
- **Secret** — хранит чувствительные данные (пароли, токены)
- **Ingress** — настраивает внешний доступ через HTTP/HTTPS
- **HorizontalPodAutoscaler** — автоматически масштабирует приложение
- **PersistentVolumeClaim** — запрашивает постоянное хранилище
- **ServiceAccount** — определяет права доступа для подов
- **NetworkPolicy** — контролирует сетевой трафик

Для одного приложения это может быть 10-15 файлов. А если у вас несколько окружений (development, staging, production), то файлов становится в три раза больше.

### Проблемы ручного управления манифестами

**Проблема №1: Копирование и изменение файлов**

Представьте, что вам нужно задеплоить одно и то же приложение в три окружения. Вы копируете все YAML-файлы три раза:

text

```
my-app/
├── dev/
│   ├── deployment.yaml     # replicaCount: 1, image: myapp:dev
│   ├── service.yaml
│   ├── ingress.yaml        # host: myapp-dev.example.com
│   └── ...
├── staging/
│   ├── deployment.yaml     # replicaCount: 2, image: myapp:staging
│   ├── service.yaml
│   ├── ingress.yaml        # host: myapp-staging.example.com
│   └── ...
└── production/
    ├── deployment.yaml     # replicaCount: 10, image: myapp:v1.2.3
    ├── service.yaml
    ├── ingress.yaml        # host: myapp.example.com
    └── ...
```

Проблемы такого подхода:

- **Дублирование кода**: 90% содержимого файлов идентично
- **Риск ошибок**: при изменении нужно помнить обновить все три версии
- **Сложность поддержки**: если нужно добавить новый ресурс (например, PodDisruptionBudget), придётся создавать его в трёх местах
- **Нет единого источника правды**: непонятно, какая версия актуальная

**Проблема №2: Отсутствие версионирования**

Когда вы применяете манифесты через `kubectl apply -f`, Kubernetes не сохраняет историю изменений:

Bash

```
# Вы применили манифесты
kubectl apply -f production/

# Что-то пошло не так
# Как откатиться на предыдущую версию?
# Нужно вручную помнить что было раньше
```

Kubernetes знает только текущее состояние кластера, но не знает:

- Кто и когда применил изменения
- Какие именно параметры были изменены
- Как быстро откатиться на стабильную версию
- Сколько было релизов и что в них менялось

**Проблема №3: Сложность применения изменений**

Манифесты нужно применять в правильном порядке:

Bash

```
# Сначала создаём namespace
kubectl apply -f namespace.yaml

# Потом ConfigMap и Secret (они нужны Deployment)
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml

# Потом ServiceAccount и RBAC
kubectl apply -f serviceaccount.yaml
kubectl apply -f rbac.yaml

# Только потом Deployment
kubectl apply -f deployment.yaml

# И наконец Service и Ingress
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

Если нарушить порядок, деплой может упасть или работать некорректно. Нужно постоянно помнить о зависимостях между ресурсами.

**Проблема №4: Управление зависимостями**

Ваше приложение часто зависит от других компонентов:

- База данных (PostgreSQL, MySQL)
- Кеш (Redis, Memcached)
- Очередь сообщений (RabbitMQ, Kafka)
- Мониторинг (Prometheus, Grafana)

Каждый из этих компонентов — это ещё один набор YAML-файлов. Приходится:

- Искать готовые манифесты для каждого компонента
- Адаптировать их под свою инфраструктуру
- Обновлять вручную при выходе новых версий
- Следить за совместимостью версий

### Что такое Helm — концептуально

**Helm — это менеджер пакетов для Kubernetes.** Он решает те же задачи, что и менеджеры пакетов в других экосистемах:

**Аналогия с другими системами:**

|Система|Менеджер пакетов|Что управляет|Репозиторий|
|---|---|---|---|
|Ubuntu/Debian|apt|.deb пакеты|apt repositories|
|RedHat/CentOS|yum/dnf|.rpm пакеты|yum repositories|
|macOS|Homebrew|формулы|brew taps|
|Node.js|npm/yarn|node модули|npmjs.com|
|Python|pip|Python пакеты|PyPI|
|**Kubernetes**|**Helm**|**Kubernetes приложения**|**Helm repositories**|

Когда вы используете `apt install nginx`, вы не создаёте вручную конфигурационные файлы, не копируете бинарники, не настраиваете systemd — всё это делает пакет. Аналогично работает Helm.

### Архитектура Helm

**Helm 3 (текущая версия)** состоит только из клиентской части:

text

```
┌─────────────────────────────────────────────────────────────┐
│                         Пользователь                         │
│                  (запускает helm команды)                    │
└────────────────────────────┬────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                      Helm Client (CLI)                       │
│  • Читает Chart (пакет с шаблонами)                         │
│  • Читает values.yaml (конфигурацию)                        │
│  • Генерирует Kubernetes манифесты из шаблонов              │
│  • Применяет манифесты через Kubernetes API                 │
│  • Сохраняет метаданные релиза в кластер (как Secret)       │
└────────────────────────────┬────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                   Kubernetes API Server                      │
│  • Получает манифесты от Helm                               │
│  • Создаёт/обновляет ресурсы (Pods, Services, etc.)         │
│  • Хранит Secrets с метаданными Helm-релизов                │
└─────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                        │
│  • Запущенные Pods                                          │
│  • Services, Ingresses                                       │
│  • ConfigMaps, Secrets                                       │
│  • И другие ресурсы                                          │
└─────────────────────────────────────────────────────────────┘
```

**Важное отличие от Helm 2:**

В Helm 2 был серверный компонент **Tiller**, который работал внутри кластера и имел широкие права доступа. Это создавало проблемы с безопасностью. В Helm 3 Tiller убрали — теперь Helm работает напрямую через kubectl/kubeconfig, используя те же права доступа, что и у пользователя.

### Что такое Chart (чарт)

**Chart** — это пакет для Helm. Это директория (или архив .tgz) со специальной структурой, содержащая:

1. **Шаблоны манифестов** — YAML-файлы с переменными вместо жёстко заданных значений
2. **Значения по умолчанию** — файл с параметрами, которые можно переопределить
3. **Метаданные** — информация о чарте (название, версия, зависимости)
4. **Документацию** — README, NOTES (сообщения после установки)

Аналогия: Chart = .deb пакет для Ubuntu или .rpm для RedHat.

### Основные концепции Helm

**1. Chart (чарт)** — пакет, шаблон приложения

text

```
mychart/
├── Chart.yaml          # Метаданные: имя, версия, описание
├── values.yaml         # Значения по умолчанию
├── templates/          # Шаблоны Kubernetes-манифестов
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── charts/             # Зависимости (другие чарты)
```

**2. Release (релиз)** — конкретная установка чарта в кластер

Один и тот же чарт можно установить много раз с разными именами и параметрами:

Bash

```
# Установить один чарт дважды
helm install myapp-dev ./mychart --set env=dev
helm install myapp-prod ./mychart --set env=prod
```

Каждая установка создаёт отдельный **релиз** с уникальным именем (`myapp-dev`, `myapp-prod`).

**3. Revision (ревизия)** — версия релиза

Каждый раз когда вы обновляете релиз, создаётся новая ревизия:

Bash

```
# Установка — создаёт ревизию 1
helm install myapp ./mychart

# Обновление — создаёт ревизию 2
helm upgrade myapp ./mychart --set replicaCount=5

# Ещё обновление — ревизия 3
helm upgrade myapp ./mychart --set image.tag=v2.0

# Откат на ревизию 2 — создаёт ревизию 4 (с содержимым ревизии 2)
helm rollback myapp 2
```

История ревизий позволяет откатываться на любую предыдущую версию.

**4. Repository (репозиторий)** — место хранения чартов

Как npm registry для Node.js или PyPI для Python, существуют репозитории чартов:

Bash

```
# Добавить репозиторий Bitnami
helm repo add bitnami https://charts.bitnami.com/bitnami

# Установить PostgreSQL из репозитория
helm install my-postgres bitnami/postgresql
```

**5. Values (значения)** — конфигурация чарта

Вместо того чтобы менять шаблоны, вы меняете значения:

YAML

```
# values.yaml
replicaCount: 3
image:
  repository: nginx
  tag: "1.25"
```

Эти значения подставляются в шаблоны при генерации манифестов.

### Жизненный цикл приложения с Helm

**Традиционный подход без Helm:**

Bash

```
# 1. Создать манифесты вручную
vim deployment.yaml
vim service.yaml
vim ingress.yaml

# 2. Применить
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml

# 3. Изменить параметр (например, увеличить реплики)
vim deployment.yaml  # Изменить replicas: 1 → replicas: 5
kubectl apply -f deployment.yaml

# 4. Откатиться? Нужно вручную помнить предыдущее состояние
# или использовать git history
```

**С Helm:**

Bash

```
# 1. Установить приложение
helm install myapp ./mychart

# 2. Обновить (изменить параметр)
helm upgrade myapp ./mychart --set replicaCount=5

# 3. Посмотреть историю
helm history myapp

# 4. Откатиться одной командой
helm rollback myapp 1

# 5. Удалить
helm uninstall myapp
```

### Ключевые преимущества Helm

**1. Параметризация через шаблоны**

Вместо дублирования файлов для каждого окружения, используете один шаблон + разные values:

text

```
Без Helm:                     С Helm:
├── dev/                      ├── mychart/
│   ├── deployment.yaml       │   └── templates/
│   └── service.yaml          │       ├── deployment.yaml (шаблон)
├── staging/                  │       └── service.yaml (шаблон)
│   ├── deployment.yaml       └── environments/
│   └── service.yaml              ├── values-dev.yaml
└── production/                   ├── values-staging.yaml
    ├── deployment.yaml           └── values-prod.yaml
    └── service.yaml
```

**2. Управление версиями и откат**

Helm сохраняет историю каждого деплоя:

Bash

```
helm history myapp
# REVISION  STATUS      CHART         DESCRIPTION
# 1         superseded  myapp-1.0.0   Install complete
# 2         superseded  myapp-1.0.1   Upgrade complete
# 3         deployed    myapp-1.1.0   Upgrade complete

# Откат на любую ревизию
helm rollback myapp 1
```

**3. Управление зависимостями**

Ваш чарт может автоматически устанавливать зависимости:

YAML

```
# Chart.yaml
dependencies:
  - name: postgresql
    version: "12.1.2"
    repository: "https://charts.bitnami.com/bitnami"
  - name: redis
    version: "17.3.7"
    repository: "https://charts.bitnami.com/bitnami"
```

При установке вашего чарта автоматически установятся PostgreSQL и Redis.

**4. Переиспользование**

Один раз создали чарт — можете использовать его для всех проектов и окружений, меняя только values.

**5. Сообщество**

Тысячи готовых чартов для популярных приложений:

- Базы данных: PostgreSQL, MySQL, MongoDB
- Мониторинг: Prometheus, Grafana, Loki
- CI/CD: Jenkins, GitLab Runner, ArgoCD
- Ingress: nginx-ingress, traefik
- И многое другое

Вместо написания манифестов с нуля, вы используете готовые решения.

### Когда использовать Helm

**Helm подходит когда:**

✅ У вас несколько окружений (dev/stage/prod) с похожей конфигурацией  
✅ Нужна история деплоев и возможность отката  
✅ Приложение зависит от других компонентов (БД, кеш, очередь)  
✅ Нужно деплоить одно приложение много раз (multi-tenancy)  
✅ Хотите переиспользовать конфигурацию между проектами  
✅ Работаете в команде и нужен единый стандарт деплоя

**Helm может быть избыточен когда:**

❌ Очень простое приложение (1-2 манифеста)  
❌ Деплоите только один раз в один кластер  
❌ Нет необходимости в параметризации  
❌ Используете GitOps подход с ArgoCD/Flux (хотя они умеют работать с Helm)

---

## 2. Helm Chart — структура и компоненты

### Что такое Chart подробно

**Chart (чарт)** — это архив или директория, содержащая полное описание Kubernetes-приложения. Это не просто набор YAML-файлов, а структурированный пакет с метаданными, шаблонами, значениями по умолчанию и зависимостями.

**Анатомия чарта:**

text

```
mychart/                           # Корневая директория чарта
│
├── Chart.yaml                     # ОБЯЗАТЕЛЬНЫЙ: метаданные чарта
│                                  # (имя, версия, описание, зависимости)
│
├── Chart.lock                     # Автогенерируемый: зафиксированные версии зависимостей
│                                  # (аналог package-lock.json в npm)
│
├── values.yaml                    # Значения по умолчанию для всех переменных
│                                  # Пользователь может переопределить их
│
├── values.schema.json             # JSON Schema для валидации values
│                                  # Опционально, но рекомендуется
│
├── README.md                      # Документация для пользователей чарта
│                                  # Как устанавливать, какие параметры есть
│
├── LICENSE                        # Лицензия чарта
│
├── .helmignore                    # Файлы для игнорирования при упаковке
│                                  # (аналог .gitignore или .dockerignore)
│
├── charts/                        # Директория с зависимостями
│   ├── postgresql-12.1.2.tgz      # Зависимость в виде архива
│   └── redis-17.3.7.tgz           # Ещё одна зависимость
│
├── templates/                     # Шаблоны Kubernetes-манифестов
│   │                              # Helm генерирует из них финальные YAML
│   │
│   ├── NOTES.txt                  # Сообщение, показываемое после установки
│   │                              # "Ваше приложение доступно по адресу..."
│   │
│   ├── _helpers.tpl               # Вспомогательные функции (не генерирует манифесты)
│   │                              # Переиспользуемые куски кода
│   │
│   ├── deployment.yaml            # Шаблон Deployment
│   ├── service.yaml               # Шаблон Service
│   ├── ingress.yaml               # Шаблон Ingress
│   ├── configmap.yaml             # Шаблон ConfigMap
│   ├── secret.yaml                # Шаблон Secret
│   ├── serviceaccount.yaml        # Шаблон ServiceAccount
│   ├── hpa.yaml                   # Шаблон HorizontalPodAutoscaler
│   ├── pvc.yaml                   # Шаблон PersistentVolumeClaim
│   ├── rbac.yaml                  # RBAC манифесты (Role, RoleBinding)
│   ├── networkpolicy.yaml         # Шаблон NetworkPolicy
│   ├── poddisruptionbudget.yaml   # Шаблон PodDisruptionBudget
│   │
│   └── tests/                     # Тесты для проверки работоспособности
│       └── test-connection.yaml   # Тест подключения к сервису
│
└── crds/                          # Custom Resource Definitions
    └── mycustomresource.yaml      # CRD устанавливаются ДО основных ресурсов
```

### Chart.yaml — метаданные чарта

Это обязательный файл, содержащий информацию о чарте. Без него директория не считается валидным чартом.

YAML

```
# Chart.yaml

# ===== ОБЯЗАТЕЛЬНЫЕ ПОЛЯ =====

# Версия API Helm Chart
# v1 — для Helm 2 (устаревший)
# v2 — для Helm 3 (текущий стандарт)
apiVersion: v2

# Имя чарта
# Должно совпадать с именем директории
# Используется только строчные буквы, цифры и дефисы
# Не может начинаться с цифры или дефиса
name: my-web-application

# Краткое описание чарта
# Отображается в helm search и Artifact Hub
description: A production-ready Helm chart for deploying my web application to Kubernetes

# Тип чарта:
# - application: обычный чарт для деплоя приложения (по умолчанию)
# - library: чарт-библиотека, не деплоится сам, используется другими чартами
type: application

# ===== ВЕРСИОНИРОВАНИЕ =====

# Версия ЧАРТА (не приложения!)
# Следует Semantic Versioning (SemVer): MAJOR.MINOR.PATCH
# Увеличивается при изменении шаблонов, зависимостей, структуры
# Примеры изменений:
#   - Исправили баг в шаблоне → 1.0.0 → 1.0.1 (PATCH)
#   - Добавили новый параметр (обратно совместимо) → 1.0.1 → 1.1.0 (MINOR)
#   - Изменили структуру values (breaking change) → 1.1.0 → 2.0.0 (MAJOR)
version: 1.2.3

# Версия ПРИЛОЖЕНИЯ (информационное поле)
# Какую версию приложения деплоит этот чарт по умолчанию
# Не влияет на логику Helm, только для информации
# Обычно совпадает с тегом Docker-образа
# Должно быть строкой (в кавычках), чтобы избежать проблем с версиями типа "1.0"
appVersion: "2.4.1"

# ===== МЕТАДАННЫЕ =====

# Ключевые слова для поиска
# Используются в helm search и Artifact Hub
keywords:
  - web
  - nginx
  - frontend
  - proxy
  - kubernetes

# URL домашней страницы проекта
# Ссылка на сайт приложения или его документацию
home: https://github.com/myorganization/my-web-app

# Ссылки на исходный код
# Может быть несколько репозиториев
sources:
  - https://github.com/myorganization/my-web-app
  - https://github.com/myorganization/my-web-app-charts

# Информация о сопровождающих чарта
# Контакты людей, которые отвечают за чарт
maintainers:
  - name: John Doe                    # Имя сопровождающего
    email: john.doe@example.com       # Email для связи
    url: https://github.com/johndoe   # Профиль GitHub/сайт
  - name: Jane Smith
    email: jane.smith@example.com

# URL иконки чарта
# Отображается в Artifact Hub и UI инструментах
# Должна быть квадратной, рекомендуется SVG или PNG
icon: https://example.com/assets/my-app-icon.svg

# ===== ЗАВИСИМОСТИ =====

# Список чартов, от которых зависит этот чарт
# Helm автоматически скачает и установит их
dependencies:
  
  # Первая зависимость: PostgreSQL
  - name: postgresql                           # Имя чарта-зависимости
    version: "12.1.2"                          # Версия (может быть диапазон: ">=12.0.0 <13.0.0")
    repository: "https://charts.bitnami.com/bitnami"  # URL репозитория
    
    # Условие включения зависимости
    # Зависимость установится только если postgresql.enabled=true в values
    condition: postgresql.enabled
    
    # Теги для группировки зависимостей
    # Можно включить/выключить группу зависимостей через --set tags.database=false
    tags:
      - database
      - backend
    
    # Импорт значений из зависимости в родительский чарт
    # Позволяет переименовать ключи values
    import-values:
      - child: auth              # Ключ в зависимости
        parent: postgresql.auth  # Ключ в родительском чарте
  
  # Вторая зависимость: Redis
  - name: redis
    version: "17.3.7"
    repository: "https://charts.bitnami.com/bitnami"
    condition: redis.enabled
    
    # Алиас позволяет переименовать зависимость
    # В values будет доступен как cache.xxx вместо redis.xxx
    # Полезно если нужно несколько экземпляров одной зависимости
    alias: cache
    tags:
      - cache
      - backend
  
  # Зависимость из локальной директории
  - name: common
    version: "1.0.0"
    repository: "file://../common"  # Относительный путь
  
  # Зависимость из OCI registry (Helm 3.8+)
  - name: myapp-library
    version: "2.0.0"
    repository: "oci://registry.example.com/helm-charts"

# ===== ДОПОЛНИТЕЛЬНЫЕ ПОЛЯ =====

# Пометить чарт как устаревший
# При установке Helm выдаст предупреждение
# Должно содержать сообщение о том, что использовать вместо этого
deprecated: false
# Если true:
# deprecated: true
# deprecationMessage: "This chart is deprecated. Use my-new-app chart instead."

# Требования к версии Kubernetes
# Чарт откажется устанавливаться, если версия кластера не подходит
# Формат: SemVer constraints (поддерживает >=, <=, >, <, =, !=, диапазоны)
kubeVersion: ">=1.20.0-0 <1.28.0-0"
# Примеры:
# ">=1.19.0"        — 1.19 и выше
# "1.20.x"          — любая 1.20.x
# ">=1.19.0 <1.25.0" — от 1.19 до 1.25 (не включая)

# Произвольные аннотации
# Могут использоваться инструментами или для метаданных
annotations:
  # Категория в Artifact Hub
  category: WebApplications
  
  # Лицензия (стандарт SPDX)
  licenses: Apache-2.0
  
  # Ссылки на дополнительные ресурсы
  artifacthub.io/links: |
    - name: Documentation
      url: https://docs.example.com
    - name: Support
      url: https://support.example.com
  
  # Изменения в этой версии
  artifacthub.io/changes: |
    - kind: added
      description: Added support for horizontal pod autoscaling
    - kind: fixed
      description: Fixed ingress TLS configuration
    - kind: changed
      description: Updated default resource limits
  
  # Скриншоты для Artifact Hub
  artifacthub.io/images: |
    - name: dashboard
      image: https://example.com/screenshots/dashboard.png
  
  # Является ли чарт подписанным
  artifacthub.io/signKey: |
    fingerprint: 0123456789ABCDEF
    url: https://keybase.io/myorg
  
  # Оператор чарта (если это Kubernetes Operator)
  artifacthub.io/operator: "true"
  artifacthub.io/operatorCapabilities: Full Lifecycle
  
  # Рекомендуемая версия Kubernetes
  artifacthub.io/recommendations: |
    - url: https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-intro/
  
  # Метаданные для CI/CD
  ci.example.com/last-tested: "2024-01-15"
  ci.example.com/stability: "stable"
```

**Разница между `version` и `appVersion`:**

- **`version`** — версия самого чарта (шаблонов, структуры, зависимостей)
    
    - Меняется когда вы изменяете Chart.yaml, templates/, values.yaml
    - Пример: добавили новый параметр в values → увеличиваем MINOR версию
- **`appVersion`** — версия приложения, которое деплоит чарт
    
    - Информационное поле
    - Обычно используется как значение по умолчанию для image.tag
    - Пример: вышла новая версия вашего приложения 2.5.0, но шаблоны чарта не менялись

YAML

```
# Пример сценария:
# 1. Создали чарт версии 1.0.0 для приложения версии 2.0.0
version: 1.0.0
appVersion: "2.0.0"

# 2. Исправили баг в шаблоне Ingress, приложение то же
version: 1.0.1      # Увеличили PATCH версию чарта
appVersion: "2.0.0" # Версия приложения не изменилась

# 3. Вышла новая версия приложения 2.1.0, шаблоны не меняли
version: 1.0.1      # Версия чарта не изменилась
appVersion: "2.1.0" # Обновили версию приложения

# 4. Добавили новый параметр в values (обратно совместимо)
version: 1.1.0      # Увеличили MINOR (новая функциональность)
appVersion: "2.1.0" # Версия приложения та же

# 5. Полностью изменили структуру values (breaking change)
version: 2.0.0      # Увеличили MAJOR (несовместимые изменения)
appVersion: "2.1.0" # Версия приложения может быть той же или новой
```

### Команды для работы с зависимостями

Bash

```
# После добавления зависимостей в Chart.yaml нужно их скачать

# Скачать зависимости и создать Chart.lock
# Использует точные версии из Chart.yaml
# Создаёт файл Chart.lock с зафиксированными версиями
helm dependency update ./mychart

# Собрать зависимости используя существующий Chart.lock
# Если Chart.lock есть, использует версии из него
# Если нет — создаёт новый (аналог update)
helm dependency build ./mychart

# Посмотреть список зависимостей и их статус
# Показывает какие зависимости есть и загружены ли они
helm dependency list ./mychart

# Пример вывода:
# NAME        VERSION   REPOSITORY                             STATUS
# postgresql  12.1.2    https://charts.bitnami.com/bitnami    ok
# redis       17.3.7    https://charts.bitnami.com/bitnami    missing
```

**Chart.lock** (автогенерируемый файл):

YAML

```
# Chart.lock
# Фиксирует точные версии зависимостей, которые были скачаны
# Аналог package-lock.json в npm или Pipfile.lock в Python
# Рекомендуется коммитить в git для воспроизводимых сборок

dependencies:
- name: postgresql
  repository: https://charts.bitnami.com/bitnami
  version: 12.1.2              # Точная версия, которая была скачана
  
- name: redis
  repository: https://charts.bitnami.com/bitnami
  version: 17.3.7

# Дайджест для проверки целостности (опционально)
digest: sha256:a1b2c3d4e5f6...

# Дата генерации файла
generated: "2024-01-15T10:30:00.123456789Z"
```

### values.yaml — конфигурация по умолчанию

Это файл с значениями по умолчанию для всех переменных, используемых в шаблонах. Пользователь может переопределить любое значение через `-f` или `--set`.

YAML

```
# values.yaml
# Это конфигурация по умолчанию для чарта
# Все значения могут быть переопределены при установке

# ===== ОСНОВНЫЕ ПАРАМЕТРЫ =====

# Количество реплик Pod'ов
# Используется в Deployment если autoscaling.enabled=false
# Может быть переопределено: --set replicaCount=5
replicaCount: 1

# ===== КОНФИГУРАЦИЯ ОБРАЗА =====

# Настройки Docker-образа приложения
image:
  # Registry где хранится образ
  # Примеры: docker.io, gcr.io, registry.example.com
  registry: docker.io
  
  # Репозиторий образа (без тега)
  # Формат: [organization/]repository
  repository: mycompany/myapp
  
  # Тег образа (версия)
  # Если не указан, используется .Chart.AppVersion из Chart.yaml
  # ВАЖНО: всегда используйте кавычки для строковых версий типа "1.0"
  # Иначе YAML может интерпретировать как число и потерять trailing zero
  tag: "1.16.0"
  
  # Политика загрузки образа:
  # - Always: всегда скачивать (даже если есть локально)
  # - IfNotPresent: скачивать только если нет локально (рекомендуется)
  # - Never: никогда не скачивать, использовать только локальный
  pullPolicy: IfNotPresent
  
  # Secrets для доступа к приватному registry
  # Если образ в приватном registry, нужно создать Secret типа docker-registry
  # kubectl create secret docker-registry regcred --docker-server=registry.example.com --docker-username=user --docker-password=pass
  pullSecrets: []
  # pullSecrets:
  #   - name: regcred
  #   - name: another-secret

# ===== ПЕРЕОПРЕДЕЛЕНИЕ ИМЁН =====

# Переопределить имя чарта (используется в labels и именах ресурсов)
# По умолчанию используется .Chart.Name
# Обычно не нужно менять
nameOverride: ""

# Полное переопределение имени ресурсов
# По умолчанию имя генерируется как: {{ .Release.Name }}-{{ .Chart.Name }}
# Если задано, будет использовано это значение
fullnameOverride: ""

# ===== SERVICE ACCOUNT =====

# ServiceAccount используется для назначения прав Pod'ам
serviceAccount:
  # Создавать ли ServiceAccount
  # Если false, используется default ServiceAccount
  create: true
  
  # Аннотации для ServiceAccount
  # Например, для AWS EKS IRSA (IAM Roles for Service Accounts):
  # annotations:
  #   eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/my-role
  annotations: {}
  
  # Имя ServiceAccount
  # Если не указано, генерируется из {{ include "mychart.fullname" . }}
  # Если create=false, должен существовать в namespace
  name: ""
  
  # Автоматически монтировать токен ServiceAccount
  # Если false, токен не будет доступен в Pod
  automountServiceAccountToken: true

# ===== АННОТАЦИИ И МЕТКИ POD'ОВ =====

# Аннотации, добавляемые к Pod'ам
# Используются для интеграции с другими системами
podAnnotations:
  # Prometheus scraping
  prometheus.io/scrape: "true"   # Prometheus будет скрапить метрики
  prometheus.io/port: "9090"     # Порт для метрик
  prometheus.io/path: "/metrics" # Путь к метрикам
  
  # Datadog
  # ad.datadoghq.com/myapp.check_names: '["http_check"]'
  
  # Другие аннотации
  # example.com/some-annotation: "value"

# Дополнительные метки для Pod'ов
# Метки используются для селекторов и организации ресурсов
podLabels:
  # Стандартные Kubernetes labels (recommended)
  app.kubernetes.io/component: backend
  app.kubernetes.io/part-of: my-platform
  
  # Кастомные метки
  environment: production
  team: platform

# ===== SECURITY CONTEXT =====

# Security Context на уровне Pod
# Применяется ко всем контейнерам в Pod
podSecurityContext:
  # Запускать от непривилегированного пользователя
  runAsNonRoot: true
  
  # UID пользователя для запуска процессов
  runAsUser: 1000
  
  # GID группы для запуска процессов
  runAsGroup: 1000
  
  # GID для владения томами (volumes)
  # Все файлы в томах будут принадлежать этой группе
  fsGroup: 1000
  
  # Профиль seccomp (ограничение системных вызовов)
  # RuntimeDefault — использовать профиль по умолчанию из runtime
  # Unconfined — без ограничений (не рекомендуется)
  seccompProfile:
    type: RuntimeDefault
  
  # SELinux контекст (для систем с SELinux)
  # seLinuxOptions:
  #   level: "s0:c123,c456"

# Security Context на уровне контейнера
# Переопределяет podSecurityContext для конкретного контейнера
securityContext:
  # Запретить повышение привилегий
  # Предотвращает использование setuid/setgid бинарников
  allowPrivilegeEscalation: false
  
  # Capabilities — права на уровне ядра Linux
  capabilities:
    # Убрать все capabilities
    drop:
      - ALL
    # Добавить только необходимые (если нужно)
    # add:
    #   - NET_BIND_SERVICE  # Для привязки к портам < 1024
  
  # Сделать корневую файловую систему read-only
  # Предотвращает запись в контейнер (только в volumes можно писать)
  readOnlyRootFilesystem: true
  
  # Запускать в privileged режиме (НЕ РЕКОМЕНДУЕТСЯ)
  # Даёт контейнеру почти все права хоста
  # privileged: false
  
  # Запускать от конкретного пользователя (переопределяет podSecurityContext)
  # runAsUser: 1000
  # runAsGroup: 1000

# ===== SERVICE =====

# Конфигурация Kubernetes Service
service:
  # Тип Service:
  # - ClusterIP: доступен только внутри кластера (по умолчанию)
  # - NodePort: доступен на порту каждой ноды (30000-32767)
  # - LoadBalancer: создаёт внешний LoadBalancer (в облаке)
  # - ExternalName: CNAME запись на внешний сервис
  type: ClusterIP
  
  # Порт Service (на который обращаются клиенты)
  port: 80
  
  # Целевой порт в Pod (куда Service перенаправляет трафик)
  # Может быть числом или именем порта из containers.ports
  targetPort: 8080
  
  # Аннотации для Service
  # Используются для настройки LoadBalancer, MetalLB и т.д.
  annotations: {}
  # Примеры:
  # annotations:
  #   service.beta.kubernetes.io/aws-load-balancer-type: nlb  # AWS NLB вместо CLB
  #   metallb.universe.tf/address-pool: production           # MetalLB pool
  #   external-dns.alpha.kubernetes.io/hostname: api.example.com  # ExternalDNS
  
  # IP адрес LoadBalancer (только для type: LoadBalancer)
  # Обычно выделяется автоматически, но можно задать конкретный
  loadBalancerIP: ""
  
  # Диапазоны IP, которым разрешён доступ к LoadBalancer
  # loadBalancerSourceRanges:
  #   - 10.0.0.0/8
  #   - 192.168.0.0/16
  
  # NodePort (только для type: NodePort)
  # Если не указан, выбирается автоматически из диапазона 30000-32767
  nodePort: ""
  
  # Session Affinity — привязка сессии к Pod
  # None — без привязки (по умолчанию)
  # ClientIP — запросы с одного IP идут на один Pod
  sessionAffinity: ""
  
  # Дополнительные порты (если нужно expose несколько портов)
  extraPorts: []
  # extraPorts:
  #   - name: metrics
  #     port: 9090
  #     targetPort: 9090
  #     protocol: TCP

# ===== INGRESS =====

# Конфигурация Ingress (HTTP/HTTPS маршрутизация)
ingress:
  # Включить создание Ingress
  enabled: false
  
  # Класс Ingress Controller
  # Примеры: nginx, traefik, alb (AWS), gce (GCP)
  # Определяет какой Ingress Controller обработает этот Ingress
  className: "nginx"
  
  # Аннотации для Ingress
  # Зависят от используемого Ingress Controller
  annotations:
    # Cert-manager для автоматических TLS сертификатов
    cert-manager.io/cluster-issuer: letsencrypt-prod
    
    # Nginx Ingress Controller
    nginx.ingress.kubernetes.io/ssl-redirect: "true"           # Редирект HTTP → HTTPS
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"         # Макс размер тела запроса
    nginx.ingress.kubernetes.io/rate-limit: "100"              # Rate limiting
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"       # Протокол к бэкенду
    
    # CORS
    # nginx.ingress.kubernetes.io/enable-cors: "true"
    # nginx.ingress.kubernetes.io/cors-allow-origin: "https://example.com"
    
    # Whitelist IP
    # nginx.ingress.kubernetes.io/whitelist-source-range: "10.0.0.0/8,192.168.0.0/16"
  
  # Список хостов и путей
  hosts:
    - host: app.example.com      # Доменное имя
      paths:
        - path: /                 # Путь (можно несколько)
          pathType: Prefix        # Prefix | Exact | ImplementationSpecific
          # Prefix: /api → /api, /api/v1, /api/users
          # Exact: /api → только /api
  
  # TLS конфигурация
  tls:
    - secretName: app-tls         # Имя Secret с сертификатом
      hosts:
        - app.example.com         # Домены для этого сертификата
  # Несколько TLS секретов:
  # tls:
  #   - secretName: app-tls
  #     hosts:
  #       - app.example.com
  #   - secretName: api-tls
  #     hosts:
  #       - api.example.com

# ===== РЕСУРСЫ =====

# Запросы и лимиты ресурсов для контейнеров
# ВАЖНО: всегда задавайте ресурсы в production
resources:
  # Лимиты — максимум, что может использовать контейнер
  # Если превысит memory — будет убит (OOMKilled)
  # Если превысит CPU — будет throttled (замедлен)
  limits:
    cpu: 500m        # 500 миллиядер = 0.5 CPU core
    memory: 512Mi    # 512 мебибайт
  
  # Запросы — гарантированные ресурсы
  # Kubernetes планирует Pod только на ноду, где есть столько ресурсов
  # Должны быть <= limits
  requests:
    cpu: 250m        # 0.25 CPU core
    memory: 256Mi    # 256 МБ

# Пустые ресурсы (не рекомендуется для production)
# resources: {}

# ===== AUTOSCALING =====

# Horizontal Pod Autoscaler — автоматическое масштабирование
autoscaling:
  # Включить HPA
  # Если enabled=true, replicaCount игнорируется
  enabled: false
  
  # Минимальное количество реплик
  minReplicas: 2
  
  # Максимальное количество реплик
  maxReplicas: 10
  
  # Целевая утилизация CPU (в процентах от requests)
  # Если средняя утилизация > 80%, добавляются реплики
  # Если < 80%, удаляются реплики
  targetCPUUtilizationPercentage: 80
  
  # Целевая утилизация Memory (опционально)
  targetMemoryUtilizationPercentage: 80
  
  # Кастомные метрики (для продвинутого использования)
  # metrics:
  #   - type: Pods
  #     pods:
  #       metric:
  #         name: http_requests_per_second
  #       target:
  #         type: AverageValue
  #         averageValue: "1000"

# ===== HEALTH CHECKS (PROBES) =====

# Liveness Probe — проверка что приложение живо
# Если проверка падает несколько раз подряд, Pod перезапускается
livenessProbe:
  httpGet:
    path: /health              # Endpoint для проверки
    port: http                 # Порт (имя из containers.ports или число)
    # scheme: HTTP             # HTTP или HTTPS
  initialDelaySeconds: 30      # Задержка перед первой проверкой (дать приложению запуститься)
  periodSeconds: 10            # Интервал между проверками
  timeoutSeconds: 5            # Таймаут одной проверки
  successThreshold: 1          # Сколько успешных проверок нужно для success
  failureThreshold: 3          # Сколько неудачных проверок до рестарта Pod

# Readiness Probe — проверка что приложение готово принимать трафик
# Если проверка падает, Pod убирается из Service (не получает трафик)
# Но Pod НЕ перезапускается (в отличие от liveness)
readinessProbe:
  httpGet:
    path: /ready
    port: http
  initialDelaySeconds: 10      # Обычно меньше чем в liveness
  periodSeconds: 5             # Можно проверять чаще
  timeoutSeconds: 3
  successThreshold: 1
  failureThreshold: 3

# Startup Probe — для медленно стартующих приложений (Kubernetes 1.16+)
# Отключает liveness и readiness проверки пока приложение стартует
# Полезно для приложений которые долго инициализируются
startupProbe:
  httpGet:
    path: /startup
    port: http
  failureThreshold: 30         # 30 попыток * 10 секунд = 5 минут на старт
  periodSeconds: 10
  timeoutSeconds: 3

# Альтернативные виды проверок:
# TCP socket probe:
# livenessProbe:
#   tcpSocket:
#     port: 8080
#
# Exec probe (выполнение команды):
# livenessProbe:
#   exec:
#     command:
#       - cat
#       - /tmp/healthy
#
# gRPC probe (Kubernetes 1.24+):
# livenessProbe:
#   grpc:
#     port: 9090

# ===== PERSISTENCE =====

# Постоянное хранилище (PersistentVolumeClaim)
persistence:
  # Включить PVC
  enabled: false
  
  # StorageClass для динамического provisioning
  # Если пусто (""), используется default StorageClass кластера
  # Если "-", отключается dynamic provisioning (нужен предсозданный PV)
  storageClass: ""
  # Примеры StorageClass:
  # - standard (GCP)
  # - gp2, gp3 (AWS EBS)
  # - azure-disk (Azure)
  # - local-path (локальное хранилище)
  
  # Режим доступа:
  # - ReadWriteOnce (RWO): монтируется read-write только на одной ноде
  # - ReadOnlyMany (ROX): монтируется read-only на многих нодах
  # - ReadWriteMany (RWX): монтируется read-write на многих нодах (NFS, CephFS)
  accessMode: ReadWriteOnce
  
  # Размер хранилища
  size: 8Gi
  
  # Аннотации для PVC
  annotations: {}
  
  # Использовать существующий PVC вместо создания нового
  # Если указано, новый PVC не создаётся
  existingClaim: ""
  
  # Точка монтирования в контейнере
  # mountPath: /data
  
  # Subpath внутри тома (опционально)
  # subPath: ""

# Дополнительные тома (кроме persistence)
# Монтируются ко всем контейнерам
extraVolumes: []
# extraVolumes:
#   - name: config
#     configMap:
#       name: my-config
#   - name: secrets
#     secret:
#       secretName: my-secrets
#   - name: empty
#     emptyDir: {}

# Дополнительные точки монтирования
extraVolumeMounts: []
# extraVolumeMounts:
#   - name: config
#     mountPath: /etc/config
#     readOnly: true
#   - name: secrets
#     mountPath: /etc/secrets
#     readOnly: true

# ===== NODE ASSIGNMENT =====

# NodeSelector — простой способ выбрать ноды
# Pod будет запланирован только на нодах с этими метками
nodeSelector: {}
# nodeSelector:
#   disktype: ssd              # Только ноды с SSD
#   kubernetes.io/arch: amd64  # Только x86_64 архитектура
#   node-role.kubernetes.io/worker: ""  # Только worker ноды

# Tolerations — позволяет планировать на нодах с taints
# Taints "отталкивают" Pod'ы, tolerations "разрешают" планирование
tolerations: []
# tolerations:
#   # Разрешить планирование на master нодах
#   - key: "node-role.kubernetes.io/master"
#     operator: "Exists"
#     effect: "NoSchedule"
#   
#   # Разрешить планирование на нодах с конкретным taint
#   - key: "dedicated"
#     operator: "Equal"
#     value: "experimental"
#     effect: "NoSchedule"
#   
#   # Разрешить планирование на spot instances
#   - key: "cloud.google.com/gke-preemptible"
#     operator: "Equal"
#     value: "true"
#     effect: "NoSchedule"

# Affinity — продвинутые правила планирования
affinity: {}
# affinity:
#   # Pod Affinity — планировать рядом с определёнными Pod'ами
#   podAffinity:
#     requiredDuringSchedulingIgnoredDuringExecution:
#       - labelSelector:
#           matchExpressions:
#             - key: app
#               operator: In
#               values:
#                 - cache
#         topologyKey: kubernetes.io/hostname  # На той же ноде
#   
#   # Pod Anti-Affinity — НЕ планировать рядом с определёнными Pod'ами
#   # Полезно для распределения реплик по разным нодам (HA)
#   podAntiAffinity:
#     preferredDuringSchedulingIgnoredDuringExecution:
#       - weight: 100
#         podAffinityTerm:
#           labelSelector:
#             matchExpressions:
#               - key: app.kubernetes.io/name
#                 operator: In
#                 values:
#                   - myapp
#           topologyKey: kubernetes.io/hostname  # Разные ноды
#   
#   # Node Affinity — планировать на нодах с определёнными характеристиками
#   nodeAffinity:
#     requiredDuringSchedulingIgnoredDuringExecution:
#       nodeSelectorTerms:
#         - matchExpressions:
#             - key: kubernetes.io/e2e-az-name
#               operator: In
#               values:
#                 - e2e-az1
#                 - e2e-az2

# ===== ENVIRONMENT VARIABLES =====

# Переменные окружения для контейнера
env: []
# env:
#   # Обычная переменная
#   - name: LOG_LEVEL
#     value: "info"
#   
#   # Из ConfigMap
#   - name: CONFIG_VALUE
#     valueFrom:
#       configMapKeyRef:
#         name: my-config
#         key: config.key
#   
#   # Из Secret
#   - name: API_KEY
#     valueFrom:
#       secretKeyRef:
#         name: my-secrets
#         key: api-key
#   
#   # Метаданные Pod (Downward API)
#   - name: POD_NAME
#     valueFrom:
#       fieldRef:
#         fieldPath: metadata.name
#   - name: POD_NAMESPACE
#     valueFrom:
#       fieldRef:
#         fieldPath: metadata.namespace
#   - name: POD_IP
#     valueFrom:
#       fieldRef:
#         fieldPath: status.podIP

# Загрузить все ключи из ConfigMap/Secret как переменные
envFrom: []
# envFrom:
#   # Все ключи из ConfigMap станут переменными
#   - configMapRef:
#       name: app-config
#   
#   # Все ключи из Secret станут переменными
#   - secretRef:
#       name: app-secrets
#   
#   # С префиксом
#   - prefix: DB_
#     configMapRef:
#       name: database-config

# ===== CONFIGMAP И SECRETS =====

# Создание ConfigMap из values
configMap:
  # Текстовые данные
  data:
    # YAML конфигурация
    config.yaml: |
      server:
        port: 8080
        timeout: 30
      logging:
        level: info
    
    # JSON конфигурация
    config.json: |
      {
        "server": {
          "port": 8080
        }
      }
    
    # Простые key-value
    app.properties: |
      app.name=MyApp
      app.version=1.0.0

# Создание Secret из values
# ВНИМАНИЕ: НЕ храните реальные секреты в values.yaml в git!
# Используйте:
# - sealed-secrets (Bitnami)
# - external-secrets
# - SOPS (Mozilla)
# - Vault
secrets:
  # Строковые данные (автоматически кодируются в base64)
  stringData:
    api-key: "super-secret-key-DO-NOT-COMMIT"
    db-password: "password123-DO-NOT-COMMIT"
  
  # Бинарные данные (уже в base64)
  # data:
  #   tls.crt: LS0tLS1CRUdJTi...
  #   tls.key: LS0tLS1CRUdJTi...

# ===== ЗАВИСИМОСТИ =====

# Настройки для зависимостей из Chart.yaml

# PostgreSQL (если dependencies содержит postgresql)
postgresql:
  # Включить установку PostgreSQL
  enabled: true
  
  # Аутентификация
  auth:
    username: myapp
    password: changeme         # В production использовать existingSecret
    database: myappdb
    # existingSecret: postgres-credentials  # Использовать существующий Secret
  
  # Настройки primary сервера
  primary:
    persistence:
      size: 10Gi
      storageClass: ""
    resources:
      requests:
        cpu: 250m
        memory: 256Mi

# Redis (если dependencies содержит redis с alias: cache)
cache:
  enabled: false
  auth:
    enabled: true
    password: changeme
  master:
    persistence:
      size: 5Gi

# ===== МОНИТОРИНГ =====

# Метрики Prometheus
metrics:
  enabled: false
  
  # ServiceMonitor для Prometheus Operator
  serviceMonitor:
    enabled: false
    interval: 30s              # Интервал scraping
    scrapeTimeout: 10s
    namespace: monitoring      # Namespace где Prometheus ищет ServiceMonitor
    labels: {}                 # Дополнительные метки

# ===== RBAC =====

# Role-Based Access Control
rbac:
  # Создавать Role и RoleBinding
  create: true
  
  # Правила доступа для ServiceAccount
  rules:
    - apiGroups: [""]          # Core API group
      resources: ["configmaps", "secrets"]
      verbs: ["get", "list", "watch"]
    
    - apiGroups: ["apps"]
      resources: ["deployments"]
      verbs: ["get", "list"]

# ===== POD DISRUPTION BUDGET =====

# Минимальное количество доступных Pod'ов при disruptions (обновления, evictions)
podDisruptionBudget:
  enabled: false
  
  # Минимум доступных Pod'ов
  minAvailable: 1
  
  # ИЛИ максимум недоступных
  # maxUnavailable: 1
  
  # Используйте либо minAvailable, либо maxUnavailable, не оба

# ===== NETWORK POLICY =====

# Контроль сетевого трафика на уровне Pod'ов
networkPolicy:
  enabled: false
  
  # Типы политик
  policyTypes:
    - Ingress      # Входящий трафик
    - Egress       # Исходящий трафик
  
  # Правила входящего трафика
  ingress:
    # Разрешить трафик только от Pod'ов с меткой app: frontend
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      # Только на порт 8080
      ports:
        - protocol: TCP
          port: 8080
  
  # Правила исходящего трафика
  egress: []
  # egress:
  #   # Разрешить DNS
  #   - to:
  #       - namespaceSelector:
  #           matchLabels:
  #             name: kube-system
  #       - podSelector:
  #           matchLabels:
  #             k8s-app: kube-dns
  #     ports:
  #       - protocol: UDP
  #         port: 53

# ===== ДОПОЛНИТЕЛЬНО =====

# Приоритет Pod (влияет на планирование и eviction)
# Требуется создание PriorityClass заранее
priorityClassName: ""

# Время ожидания graceful shutdown (в секундах)
# Kubernetes отправляет SIGTERM, ждёт это время, потом SIGKILL
terminationGracePeriodSeconds: 30

# Политика DNS для Pod
# ClusterFirst — использовать DNS кластера (по умолчанию)
# Default — использовать DNS ноды
# None — кастомная конфигурация через dnsConfig
dnsPolicy: ClusterFirst

# Политика перезапуска контейнеров
# Always — всегда перезапускать (для Deployment)
# OnFailure — только при ошибке (для Job)
# Never — никогда не перезапускать
restartPolicy: Always

# Init Containers — запускаются перед основными контейнерами
# Полезны для инициализации, миграций, ожидания зависимостей
initContainers: []
# initContainers:
#   - name: wait-for-db
#     image: busybox:1.36
#     command:
#       - sh
#       - -c
#       - |
#         until nc -z postgres 5432; do
#           echo "Waiting for PostgreSQL..."
#           sleep 2
#         done
#   - name: run-migrations
#     image: myapp:latest
#     command: ["/app/migrate"]

# Sidecar контейнеры (в том же Pod)
sidecars: []
# sidecars:
#   - name: log-shipper
#     image: fluent/fluent-bit:2.0
#     volumeMounts:
#       - name: varlog
#         mountPath: /var/log

# Стратегия обновления Deployment
updateStrategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0          # Сколько Pod'ов могут быть недоступны
    maxSurge: 1                # Сколько дополнительных Pod'ов создавать

# Аннотации для Deployment (не Pod)
deploymentAnnotations: {}

# Lifecycle hooks
lifecycle: {}
# lifecycle:
#   # Выполнить перед остановкой контейнера
#   preStop:
#     exec:
#       command: ["/bin/sh", "-c", "sleep 15"]
#   # Выполнить после старта контейнера
#   postStart:
#     exec:
#       command: ["/bin/sh", "-c", "echo Container started"]
```

## 3. Шаблоны (templates/) — детальный разбор

### Основы шаблонизации в Helm

Helm использует **Go templates** — движок шаблонизации из стандартной библиотеки Go. Шаблоны позволяют генерировать Kubernetes-манифесты динамически, подставляя значения из values.yaml.

**Базовый синтаксис:**

YAML

```
# Двойные фигурные скобки — это директивы шаблона
# Всё между {{ и }} обрабатывается Helm

# Вывести значение переменной
{{ .Values.replicaCount }}

# Вызвать функцию
{{ quote .Values.image.tag }}

# Использовать пайплайн (передать результат в следующую функцию)
{{ .Values.image.tag | quote }}

# Управление пробелами:
# {{- убирает пробелы и переносы СЛЕВА от директивы
# -}} убирает пробелы и переносы СПРАВА от директивы
{{- if .Values.enabled }}
value: true
{{- end }}
```

### Встроенные объекты Helm

Helm предоставляет несколько встроенных объектов, доступных во всех шаблонах:

YAML

```
# ===== .Values =====
# Значения из values.yaml и переопределений (--set, -f)
# Это главный источник конфигурации

{{ .Values.replicaCount }}           # Простое значение: 3
{{ .Values.image.repository }}       # Вложенное значение: nginx
{{ .Values.service.port }}           # Ещё вложенное: 80


# ===== .Release =====
# Информация о текущем релизе (установке чарта)

{{ .Release.Name }}        # Имя релиза, указанное при helm install
                           # Пример: "myapp-production"

{{ .Release.Namespace }}   # Namespace куда устанавливается релиз
                           # Пример: "production"

{{ .Release.Service }}     # Сервис, выполняющий установку
                           # Всегда "Helm" для Helm 3

{{ .Release.Revision }}    # Номер ревизии релиза
                           # При install = 1, при каждом upgrade увеличивается

{{ .Release.IsUpgrade }}   # Boolean: true если это upgrade, false если install

{{ .Release.IsInstall }}   # Boolean: true если это install, false если upgrade


# ===== .Chart =====
# Содержимое файла Chart.yaml

{{ .Chart.Name }}          # Имя чарта из Chart.yaml
                           # Пример: "my-web-app"

{{ .Chart.Version }}       # Версия чарта
                           # Пример: "1.2.3"

{{ .Chart.AppVersion }}    # Версия приложения
                           # Пример: "2.4.1"

{{ .Chart.Description }}   # Описание чарта

{{ .Chart.Type }}          # Тип: "application" или "library"

{{ .Chart.Keywords }}      # Список ключевых слов

{{ .Chart.Home }}          # URL домашней страницы

{{ .Chart.Maintainers }}   # Список сопровождающих


# ===== .Capabilities =====
# Информация о возможностях кластера Kubernetes

{{ .Capabilities.KubeVersion }}              # Версия Kubernetes
                                             # Пример: v1.27.3

{{ .Capabilities.KubeVersion.Major }}        # Мажорная версия: "1"

{{ .Capabilities.KubeVersion.Minor }}        # Минорная версия: "27"

{{ .Capabilities.KubeVersion.GitVersion }}   # Полная версия: "v1.27.3"

{{ .Capabilities.APIVersions }}              # Список доступных API версий

# Проверка поддержки конкретного API
{{ .Capabilities.APIVersions.Has "networking.k8s.io/v1" }}  # true/false


# ===== .Template =====
# Информация о текущем шаблоне

{{ .Template.Name }}       # Путь к текущему файлу шаблона
                           # Пример: "mychart/templates/deployment.yaml"

{{ .Template.BasePath }}   # Базовый путь к templates
                           # Пример: "mychart/templates"


# ===== .Files =====
# Доступ к файлам внутри чарта (кроме templates/)

{{ .Files.Get "config/app.conf" }}           # Содержимое файла как строка

{{ .Files.GetBytes "binary/data.bin" }}      # Содержимое как байты

{{ .Files.Glob "config/*.yaml" }}            # Все файлы по паттерну

{{ .Files.Lines "config/list.txt" }}         # Файл как список строк

{{ .Files.AsConfig }}                        # Как данные для ConfigMap

{{ .Files.AsSecrets }}                       # Как данные для Secret (base64)
```

### Функции шаблонизации

Helm включает более 60 функций из библиотеки **Sprig** плюс свои собственные:

YAML

```
# ===== РАБОТА СО СТРОКАМИ =====

# quote — обернуть в двойные кавычки
# Важно для строк которые могут быть интерпретированы как числа/boolean
image: {{ .Values.image.tag | quote }}
# Результат: image: "1.16.0"

# squote — обернуть в одинарные кавычки
annotation: {{ .Values.annotation | squote }}
# Результат: annotation: 'some value'

# upper — в верхний регистр
env: {{ .Values.environment | upper }}
# Результат: env: PRODUCTION

# lower — в нижний регистр
name: {{ .Values.name | lower }}
# Результат: name: myapp

# title — каждое слово с большой буквы
title: {{ "hello world" | title }}
# Результат: title: Hello World

# trim — убрать пробелы по краям
name: {{ .Values.name | trim }}

# trimPrefix — убрать префикс
name: {{ "prefix-myapp" | trimPrefix "prefix-" }}
# Результат: name: myapp

# trimSuffix — убрать суффикс
name: {{ "myapp-suffix" | trimSuffix "-suffix" }}
# Результат: name: myapp

# trunc — обрезать до N символов
# Kubernetes имеет ограничения на длину имён (63 символа)
name: {{ .Values.longName | trunc 63 }}

# replace — замена подстроки
name: {{ .Values.name | replace "_" "-" }}
# Результат: my_app → my-app

# contains — проверка вхождения подстроки
{{- if contains "nginx" .Values.image.repository }}
# ...
{{- end }}

# hasPrefix / hasSuffix — проверка префикса/суффикса
{{- if hasPrefix "v" .Values.image.tag }}
# Тег начинается с "v"
{{- end }}

# indent — добавить отступ (без начального переноса строки)
data:
{{ .Values.config | indent 2 }}

# nindent — добавить отступ С начальным переносом строки
# Более удобен в большинстве случаев
data:
  {{- .Values.config | nindent 2 }}

# printf — форматированный вывод (как в C/Go)
image: {{ printf "%s:%s" .Values.image.repository .Values.image.tag }}
# Результат: image: nginx:1.16.0

# Конкатенация строк через print
name: {{ print .Release.Name "-" .Chart.Name }}
# Результат: name: myrelease-mychart


# ===== ЗНАЧЕНИЯ ПО УМОЛЧАНИЮ =====

# default — значение по умолчанию если пусто/nil
replicas: {{ .Values.replicaCount | default 1 }}
# Если replicaCount не задан или пуст, будет 1

# Цепочка defaults
port: {{ .Values.customPort | default .Values.service.port | default 8080 }}

# coalesce — первое непустое значение из списка
value: {{ coalesce .Values.first .Values.second .Values.third "default" }}

# required — обязательное значение, ошибка если не задано
apiKey: {{ required "API key is required!" .Values.apiKey }}
# Если .Values.apiKey пуст, helm install/upgrade упадёт с ошибкой


# ===== ПРОВЕРКИ ТИПОВ И ЗНАЧЕНИЙ =====

# empty — проверка на пустоту (nil, "", 0, false, пустой список/map)
{{- if not (empty .Values.annotations) }}
annotations:
  {{- toYaml .Values.annotations | nindent 2 }}
{{- end }}

# kindOf — получить тип значения
{{ kindOf .Values.data }}  # "map", "slice", "string", "int", etc.

# kindIs — проверить тип
{{- if kindIs "string" .Values.value }}
value: {{ .Values.value | quote }}
{{- else }}
value: {{ .Values.value }}
{{- end }}

# typeOf — более точный тип (Go type)
{{ typeOf .Values.data }}  # "map[string]interface {}"


# ===== СПИСКИ (SLICES) =====

# list — создать список
{{ list "a" "b" "c" }}  # ["a", "b", "c"]

# first — первый элемент
{{ first .Values.hosts }}

# last — последний элемент
{{ last .Values.hosts }}

# rest — всё кроме первого
{{ rest .Values.hosts }}

# initial — всё кроме последнего
{{ initial .Values.hosts }}

# append — добавить элемент
{{ append .Values.hosts "new-host" }}

# prepend — добавить в начало
{{ prepend .Values.hosts "first-host" }}

# concat — объединить списки
{{ concat .Values.hosts1 .Values.hosts2 }}

# has — проверить наличие элемента
{{- if has "admin" .Values.roles }}
# Пользователь — администратор
{{- end }}

# without — убрать элементы из списка
{{ without .Values.features "deprecated-feature" }}

# uniq — уникальные элементы
{{ uniq .Values.tags }}

# sortAlpha — сортировка строк
{{ sortAlpha .Values.names }}

# reverse — обратный порядок
{{ reverse .Values.items }}

# Доступ по индексу
{{ index .Values.hosts 0 }}  # Первый элемент


# ===== СЛОВАРИ (MAPS) =====

# dict — создать словарь
{{ dict "key1" "value1" "key2" "value2" }}

# get — получить значение по ключу (безопасно, не падает если нет)
{{ get .Values.config "timeout" }}

# set — установить значение (изменяет оригинал!)
{{ set .Values.config "newKey" "newValue" }}

# unset — удалить ключ
{{ unset .Values.config "deprecatedKey" }}

# hasKey — проверить наличие ключа
{{- if hasKey .Values.config "optionalSetting" }}
# ...
{{- end }}

# keys — получить список ключей
{{ keys .Values.config }}

# values — получить список значений
{{ values .Values.config }}

# merge — объединить словари (первый имеет приоритет)
{{ merge .Values.overrides .Values.defaults }}

# mergeOverwrite — объединить (последний имеет приоритет)
{{ mergeOverwrite .Values.defaults .Values.overrides }}

# pick — оставить только указанные ключи
{{ pick .Values.config "key1" "key2" }}

# omit — убрать указанные ключи
{{ omit .Values.config "sensitiveKey" }}

# pluck — извлечь значения по ключу из списка словарей
{{ pluck "name" .Values.users }}  # ["user1", "user2", ...]


# ===== ПРЕОБРАЗОВАНИЕ В YAML/JSON =====

# toYaml — преобразовать в YAML строку
# Самая часто используемая функция для вложенных структур
resources:
  {{- toYaml .Values.resources | nindent 2 }}
# Результат:
# resources:
#   limits:
#     cpu: 500m
#     memory: 512Mi
#   requests:
#     cpu: 250m
#     memory: 256Mi

# toJson — преобразовать в JSON
config: {{ .Values.config | toJson | quote }}

# toPrettyJson — JSON с форматированием
config: |
  {{ .Values.config | toPrettyJson | nindent 2 }}

# fromYaml — парсить YAML строку в объект
{{- $config := .Files.Get "config.yaml" | fromYaml }}
{{ $config.key }}

# fromJson — парсить JSON строку в объект
{{- $data := .Files.Get "data.json" | fromJson }}


# ===== МАТЕМАТИКА =====

# add — сложение
{{ add 1 2 3 }}  # 6

# sub — вычитание
{{ sub 10 3 }}  # 7

# mul — умножение
{{ mul 2 3 4 }}  # 24

# div — целочисленное деление
{{ div 10 3 }}  # 3

# mod — остаток от деления
{{ mod 10 3 }}  # 1

# max / min — максимум / минимум
{{ max 1 5 3 }}  # 5
{{ min 1 5 3 }}  # 1

# floor / ceil / round — округление
{{ floor 3.7 }}  # 3
{{ ceil 3.2 }}   # 4
{{ round 3.5 }}  # 4


# ===== ЛОГИКА =====

# and — логическое И (возвращает последний true или первый false)
{{- if and .Values.ingress.enabled .Values.ingress.tls }}

# or — логическое ИЛИ (возвращает первый true или последний false)
{{- if or .Values.service.enabled .Values.ingress.enabled }}

# not — логическое НЕ
{{- if not .Values.disabled }}

# eq — равенство
{{- if eq .Values.environment "production" }}

# ne — не равно
{{- if ne .Values.environment "development" }}

# lt — меньше (less than)
{{- if lt .Values.replicaCount 3 }}

# le — меньше или равно
{{- if le .Values.replicaCount 3 }}

# gt — больше (greater than)
{{- if gt .Values.replicaCount 1 }}

# ge — больше или равно
{{- if ge .Values.replicaCount 1 }}

# Тернарный оператор через ternary
{{ ternary "yes" "no" .Values.enabled }}
# Если enabled=true → "yes", иначе → "no"


# ===== РЕГУЛЯРНЫЕ ВЫРАЖЕНИЯ =====

# regexMatch — проверка соответствия
{{- if regexMatch "^v[0-9]+" .Values.image.tag }}
# Тег начинается с v и цифр
{{- end }}

# regexFind — найти первое совпадение
{{ regexFind "[0-9]+" "version-123-abc" }}  # "123"

# regexFindAll — найти все совпадения
{{ regexFindAll "[0-9]+" "a1b2c3" -1 }}  # ["1", "2", "3"]

# regexReplaceAll — замена по регулярке
{{ regexReplaceAll "\\s+" .Values.name "-" }}
# Заменить все пробелы на дефисы

# regexSplit — разбить по регулярке
{{ regexSplit "\\s+" "a b  c" -1 }}  # ["a", "b", "c"]


# ===== КРИПТОГРАФИЯ И КОДИРОВАНИЕ =====

# b64enc — кодировать в base64
# Необходимо для Secrets
data:
  password: {{ .Values.password | b64enc }}

# b64dec — декодировать из base64
{{ .Values.encodedData | b64dec }}

# sha1sum / sha256sum — хэш-суммы
# Полезно для аннотаций, чтобы перезапустить Pod при изменении ConfigMap
annotations:
  checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}

# randAlphaNum — случайная строка
secretKey: {{ randAlphaNum 32 | b64enc }}

# randAlpha — только буквы
# randNumeric — только цифры
# randAscii — любые ASCII


# ===== ДАТА И ВРЕМЯ =====

# now — текущее время
{{ now }}  # 2024-01-15 10:30:00.123456789 +0000 UTC

# date — форматирование даты
# Формат Go: Mon Jan 2 15:04:05 MST 2006
{{ now | date "2006-01-02" }}  # 2024-01-15

# dateModify — изменить дату
{{ now | dateModify "-24h" | date "2006-01-02" }}  # Вчера

# toDate — парсить дату из строки
{{ toDate "2006-01-02" "2024-01-15" }}

# unixEpoch — Unix timestamp
{{ now | unixEpoch }}  # 1705315800


# ===== РАБОТА С ПУТЯМИ =====

# base — имя файла из пути
{{ base "/path/to/file.txt" }}  # file.txt

# dir — директория из пути
{{ dir "/path/to/file.txt" }}  # /path/to

# ext — расширение файла
{{ ext "/path/to/file.txt" }}  # .txt

# clean — нормализовать путь
{{ clean "/path//to/../to/file" }}  # /path/to/file

# isAbs — абсолютный ли путь
{{ isAbs "/path/to/file" }}  # true


# ===== РАБОТА С URL =====

# urlParse — разобрать URL
{{- $url := urlParse "https://user:pass@example.com:8080/path?query=1" }}
{{ $url.scheme }}  # https
{{ $url.host }}    # example.com:8080
{{ $url.path }}    # /path

# urlJoin — собрать URL из частей
{{ urlJoin (dict "scheme" "https" "host" "example.com" "path" "/api") }}


# ===== СПЕЦИФИЧНЫЕ ДЛЯ HELM =====

# include — вызвать именованный шаблон и вернуть результат как строку
# Основной способ переиспользования кода
name: {{ include "mychart.fullname" . }}

# template — аналог include, но результат напрямую в вывод
# Не рекомендуется, include более гибкий
{{ template "mychart.labels" . }}

# tpl — обработать строку как шаблон
# Полезно когда шаблон приходит из values
{{ tpl .Values.customTemplate . }}

# lookup — получить ресурс из кластера (только при install/upgrade, не template)
# apiVersion, kind, namespace, name
{{- $secret := lookup "v1" "Secret" .Release.Namespace "my-secret" }}
{{- if $secret }}
# Secret существует
{{- end }}

# Получить список ресурсов
{{- $configmaps := lookup "v1" "ConfigMap" .Release.Namespace "" }}
{{- range $configmaps.items }}
# Итерация по ConfigMaps
{{- end }}
```

### Управляющие конструкции

YAML

```
# ===== УСЛОВИЯ (IF/ELSE) =====

# Простое условие
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
# ...
{{- end }}

# С else
{{- if .Values.autoscaling.enabled }}
# HPA управляет репликами
{{- else }}
replicas: {{ .Values.replicaCount }}
{{- end }}

# С else if
{{- if eq .Values.service.type "LoadBalancer" }}
type: LoadBalancer
{{- else if eq .Values.service.type "NodePort" }}
type: NodePort
{{- else }}
type: ClusterIP
{{- end }}

# Сложные условия
{{- if and .Values.ingress.enabled (gt (len .Values.ingress.hosts) 0) }}
# Ingress включён И есть хосты
{{- end }}

{{- if or .Values.persistence.enabled .Values.persistence.existingClaim }}
# Либо создаём PVC, либо используем существующий
{{- end }}

# Проверка на существование ключа
{{- if hasKey .Values "optionalFeature" }}
# Ключ существует в values
{{- end }}

# Проверка на непустоту
{{- if .Values.annotations }}
# .Values.annotations не nil и не пустой map
annotations:
  {{- toYaml .Values.annotations | nindent 2 }}
{{- end }}


# ===== WITH — ИЗМЕНЕНИЕ КОНТЕКСТА =====

# with изменяет текущий контекст (.) на указанный объект
# Если объект пустой/nil, блок пропускается (как if)

# Без with:
{{- if .Values.nodeSelector }}
nodeSelector:
  {{- toYaml .Values.nodeSelector | nindent 2 }}
{{- end }}

# С with (короче и понятнее):
{{- with .Values.nodeSelector }}
nodeSelector:
  {{- toYaml . | nindent 2 }}
{{- end }}

# ВАЖНО: внутри with текущий контекст (.) изменён!
# Чтобы обратиться к корневому контексту, используйте $

{{- with .Values.ingress }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  # $.Release — обращение к корневому контексту
  name: {{ $.Release.Name }}-ingress
  # . — теперь это .Values.ingress
  {{- with .annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
{{- end }}


# ===== RANGE — ЦИКЛЫ =====

# Итерация по списку
{{- range .Values.hosts }}
- host: {{ . }}
{{- end }}
# .Values.hosts = ["host1.com", "host2.com"]
# Результат:
# - host: host1.com
# - host: host2.com

# С индексом
{{- range $index, $host := .Values.hosts }}
- index: {{ $index }}
  host: {{ $host }}
{{- end }}

# Итерация по словарю (map)
{{- range $key, $value := .Values.env }}
- name: {{ $key }}
  value: {{ $value | quote }}
{{- end }}
# .Values.env = {LOG_LEVEL: info, DEBUG: "true"}
# Результат:
# - name: DEBUG
#   value: "true"
# - name: LOG_LEVEL
#   value: "info"

# Вложенные циклы
{{- range .Values.ingress.hosts }}
- host: {{ .host }}
  paths:
    {{- range .paths }}
    - path: {{ .path }}
      pathType: {{ .pathType }}
    {{- end }}
{{- end }}

# Доступ к корневому контексту из цикла
{{- range .Values.containers }}
- name: {{ .name }}
  image: {{ $.Values.image.repository }}:{{ $.Values.image.tag }}
{{- end }}

# Генерация чисел через untilStep
{{- range untilStep 0 5 1 }}
# Числа от 0 до 4 с шагом 1: {{ . }}
{{- end }}


# ===== DEFINE — ОПРЕДЕЛЕНИЕ ИМЕНОВАННЫХ ШАБЛОНОВ =====

# Определение шаблона (обычно в _helpers.tpl)
{{- define "mychart.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

# Использование шаблона через include
name: {{ include "mychart.name" . }}

# include возвращает строку, которую можно обработать
labels:
  {{- include "mychart.labels" . | nindent 2 }}

# template напрямую выводит результат (менее гибко)
{{ template "mychart.name" . }}


# ===== ПЕРЕМЕННЫЕ =====

# Определение переменной
{{- $fullname := include "mychart.fullname" . }}
{{- $servicePort := .Values.service.port }}

# Использование
name: {{ $fullname }}
port: {{ $servicePort }}

# Переменные полезны для:
# 1. Сокращения длинных выражений
# 2. Сохранения контекста перед with/range
# 3. Промежуточных вычислений

{{- $root := . }}
{{- range .Values.items }}
  # $ тоже ссылается на root, но $root более явно
  release: {{ $root.Release.Name }}
{{- end }}

# Изменение переменной (не рекомендуется, усложняет код)
{{- $value := "initial" }}
{{- $value = "modified" }}


# ===== УПРАВЛЕНИЕ ПРОБЕЛАМИ =====

# Пробелы в шаблонах часто вызывают проблемы
# Сравните:

# Без управления пробелами:
labels:
  {{ if .Values.customLabel }}
  custom: {{ .Values.customLabel }}
  {{ end }}
  standard: value

# Результат (лишние пустые строки):
# labels:
#   
#   custom: myvalue
#   
#   standard: value

# С управлением пробелами:
labels:
  {{- if .Values.customLabel }}
  custom: {{ .Values.customLabel }}
  {{- end }}
  standard: value

# Результат (чисто):
# labels:
#   custom: myvalue
#   standard: value

# Правила:
# {{-  — убрать все пробелы/переносы СЛЕВА от директивы
# -}}  — убрать все пробелы/переносы СПРАВА от директивы
# {{- -}} — убрать с обеих сторон
```

### templates/_helpers.tpl — переиспользуемый код

Файл `_helpers.tpl` содержит именованные шаблоны (define), которые можно вызывать из других шаблонов. Имя начинается с `_`, поэтому Helm не обрабатывает его как манифест.

YAML

```
{{/*
==========================================================================
_helpers.tpl — вспомогательные шаблоны для чарта
==========================================================================

Этот файл содержит переиспользуемые именованные шаблоны.
Они вызываются через {{ include "имя" . }} из других шаблонов.

Файлы начинающиеся с _ не рендерятся как отдельные манифесты.
*/}}


{{/*
--------------------------------------------------------------------------
Expand the name of the chart.
Возвращает имя чарта, которое используется в метках и именах ресурсов.

Логика:
1. Если задан .Values.nameOverride — использовать его
2. Иначе — использовать .Chart.Name из Chart.yaml
3. Обрезать до 63 символов (ограничение Kubernetes)
4. Убрать trailing дефис (если обрезка попала на него)

Пример:
  .Chart.Name = "my-web-application"
  .Values.nameOverride = ""
  Результат: "my-web-application"

  .Values.nameOverride = "myapp"
  Результат: "myapp"
--------------------------------------------------------------------------
*/}}
{{- define "mychart.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}


{{/*
--------------------------------------------------------------------------
Create a default fully qualified app name.
Создаёт полное имя приложения для использования в именах ресурсов.

Логика:
1. Если задан .Values.fullnameOverride — использовать его напрямую
2. Иначе:
   a. Взять name (из nameOverride или Chart.Name)
   b. Если имя релиза уже содержит name — использовать только имя релиза
      (избегаем дублирования: "myapp-myapp")
   c. Иначе — комбинировать: {{ .Release.Name }}-{{ name }}
3. Обрезать до 63 символов

Пример:
  .Release.Name = "production"
  .Chart.Name = "myapp"
  Результат: "production-myapp"

  .Release.Name = "myapp"
  .Chart.Name = "myapp"
  Результат: "myapp" (не "myapp-myapp")

  .Values.fullnameOverride = "custom-name"
  Результат: "custom-name"
--------------------------------------------------------------------------
*/}}
{{- define "mychart.fullname" -}}
{{- if .Values.fullnameOverride }}
  {{- /* Если задан fullnameOverride — используем его */ -}}
  {{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
  {{- /* Иначе комбинируем Release.Name и Chart.Name */ -}}
  {{- $name := default .Chart.Name .Values.nameOverride }}
  {{- if contains $name .Release.Name }}
    {{- /* Если имя релиза уже содержит имя чарта — не дублируем */ -}}
    {{- .Release.Name | trunc 63 | trimSuffix "-" }}
  {{- else }}
    {{- /* Стандартный случай: release-name + chart-name */ -}}
    {{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
  {{- end }}
{{- end }}
{{- end }}


{{/*
--------------------------------------------------------------------------
Create chart name and version as used by the chart label.
Создаёт строку "name-version" для метки helm.sh/chart.

Используется для идентификации какой версией чарта был создан ресурс.

Логика:
1. Форматировать как {{ .Chart.Name }}-{{ .Chart.Version }}
2. Заменить "+" на "_" (плюсы недопустимы в метках Kubernetes)
3. Обрезать до 63 символов

Пример:
  .Chart.Name = "myapp"
  .Chart.Version = "1.2.3+build.456"
  Результат: "myapp-1.2.3_build.456"
--------------------------------------------------------------------------
*/}}
{{- define "mychart.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}


{{/*
--------------------------------------------------------------------------
Common labels
Общие метки, добавляемые ко ВСЕМ ресурсам чарта.

Следует рекомендациям Kubernetes по меткам:
https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/

Включает:
- helm.sh/chart: идентификация чарта и версии
- app.kubernetes.io/name: имя приложения
- app.kubernetes.io/instance: имя инстанса (релиза)
- app.kubernetes.io/version: версия приложения
- app.kubernetes.io/managed-by: чем управляется (Helm)

Использование в шаблонах:
  metadata:
    labels:
      {{- include "mychart.labels" . | nindent 6 }}
--------------------------------------------------------------------------
*/}}
{{- define "mychart.labels" -}}
helm.sh/chart: {{ include "mychart.chart" . }}
{{ include "mychart.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}


{{/*
--------------------------------------------------------------------------
Selector labels
Метки для селекторов (selector.matchLabels и template.metadata.labels).

ВАЖНО: Эти метки используются для связи Service → Pods и Deployment → Pods.
Они НЕ должны меняться после первоначальной установки, иначе:
- Service потеряет связь с Pod'ами
- Deployment не сможет обновить существующие Pod'ы

Поэтому здесь используются только стабильные значения:
- app.kubernetes.io/name — имя приложения (из чарта)
- app.kubernetes.io/instance — имя релиза

НЕ включаются:
- version (меняется при обновлении приложения)
- chart version (меняется при обновлении чарта)
--------------------------------------------------------------------------
*/}}
{{- define "mychart.selectorLabels" -}}
app.kubernetes.io/name: {{ include "mychart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}


{{/*
--------------------------------------------------------------------------
Create the name of the service account to use.
Определяет имя ServiceAccount для использования в Pod'ах.

Логика:
1. Если ServiceAccount создаётся чартом (create=true):
   - Использовать указанное имя (.Values.serviceAccount.name)
   - Или сгенерировать из fullname
2. Если НЕ создаётся (используем существующий):
   - Использовать указанное имя
   - Или "default" ServiceAccount namespace

Примеры:
  serviceAccount.create=true, name=""
  Результат: "release-mychart" (fullname)

  serviceAccount.create=true, name="custom-sa"
  Результат: "custom-sa"

  serviceAccount.create=false, name="existing-sa"
  Результат: "existing-sa"

  serviceAccount.create=false, name=""
  Результат: "default"
--------------------------------------------------------------------------
*/}}
{{- define "mychart.serviceAccountName" -}}
{{- if .Values.serviceAccount.create }}
  {{- /* Создаём SA: используем имя или генерируем */ -}}
  {{- default (include "mychart.fullname" .) .Values.serviceAccount.name }}
{{- else }}
  {{- /* Не создаём: используем указанное или default */ -}}
  {{- default "default" .Values.serviceAccount.name }}
{{- end }}
{{- end }}


{{/*
--------------------------------------------------------------------------
Return the appropriate apiVersion for HPA.
Возвращает правильную apiVersion для HorizontalPodAutoscaler.

Kubernetes меняет API версии со временем:
- autoscaling/v1 — старый, ограниченный (только CPU)
- autoscaling/v2beta2 — расширенный (memory, custom metrics)
- autoscaling/v2 — стабильный с Kubernetes 1.23+

Этот шаблон автоматически выбирает подходящую версию.
--------------------------------------------------------------------------
*/}}
{{- define "mychart.hpa.apiVersion" -}}
{{- if .Capabilities.APIVersions.Has "autoscaling/v2" }}
  {{- /* Kubernetes 1.23+ */ -}}
  {{- print "autoscaling/v2" }}
{{- else if .Capabilities.APIVersions.Has "autoscaling/v2beta2" }}
  {{- /* Kubernetes 1.18-1.22 */ -}}
  {{- print "autoscaling/v2beta2" }}
{{- else }}
  {{- /* Очень старый Kubernetes */ -}}
  {{- print "autoscaling/v2beta1" }}
{{- end }}
{{- end }}


{{/*
--------------------------------------------------------------------------
Return the appropriate apiVersion for Ingress.
Аналогично для Ingress, который тоже менял API.
--------------------------------------------------------------------------
*/}}
{{- define "mychart.ingress.apiVersion" -}}
{{- if .Capabilities.APIVersions.Has "networking.k8s.io/v1" }}
  {{- /* Kubernetes 1.19+ */ -}}
  {{- print "networking.k8s.io/v1" }}
{{- else if .Capabilities.APIVersions.Has "networking.k8s.io/v1beta1" }}
  {{- /* Kubernetes 1.14-1.18 */ -}}
  {{- print "networking.k8s.io/v1beta1" }}
{{- else }}
  {{- /* Очень старый Kubernetes */ -}}
  {{- print "extensions/v1beta1" }}
{{- end }}
{{- end }}


{{/*
--------------------------------------------------------------------------
Create image pull secret reference.
Формирует список imagePullSecrets для Pod'а.

Поддерживает два формата в values:
1. Список строк: ["secret1", "secret2"]
2. Список объектов: [{name: "secret1"}, {name: "secret2"}]
--------------------------------------------------------------------------
*/}}
{{- define "mychart.imagePullSecrets" -}}
{{- with .Values.image.pullSecrets }}
imagePullSecrets:
  {{- range . }}
    {{- if typeIs "string" . }}
  - name: {{ . }}
    {{- else }}
  - name: {{ .name }}
    {{- end }}
  {{- end }}
{{- end }}
{{- end }}


{{/*
--------------------------------------------------------------------------
Create container image string.
Формирует полный путь к образу: registry/repository:tag

Параметры из .Values.image:
- registry: реестр (docker.io, gcr.io, etc.)
- repository: репозиторий образа
- tag: тег (если не указан, используется Chart.AppVersion)
--------------------------------------------------------------------------
*/}}
{{- define "mychart.image" -}}
{{- $registry := .Values.image.registry | default "" }}
{{- $repository := .Values.image.repository }}
{{- $tag := .Values.image.tag | default .Chart.AppVersion }}
{{- if $registry }}
  {{- /* С registry */ -}}
  {{- printf "%s/%s:%s" $registry $repository $tag }}
{{- else }}
  {{- /* Без registry (Docker Hub) */ -}}
  {{- printf "%s:%s" $repository $tag }}
{{- end }}
{{- end }}


{{/*
--------------------------------------------------------------------------
Checksum helper.
Вычисляет checksum содержимого файла для аннотаций.

Использование:
annotations:
  checksum/config: {{ include "mychart.checksum" (dict "file" "configmap.yaml" "context" $) }}

Это заставляет Pod перезапуститься при изменении ConfigMap,
так как меняется аннотация → меняется Pod spec → rollout.
--------------------------------------------------------------------------
*/}}
{{- define "mychart.checksum" -}}
{{- $file := .file }}
{{- $context := .context }}
{{ include (print $context.Template.BasePath "/" $file) $context | sha256sum }}
{{- end }}


{{/*
--------------------------------------------------------------------------
Environment variables from secrets helper.
Генерирует env записи для переменных из Secret.
--------------------------------------------------------------------------
*/}}
{{- define "mychart.secretEnv" -}}
{{- range $key, $value := . }}
- name: {{ $key }}
  valueFrom:
    secretKeyRef:
      name: {{ $.secretName }}
      key: {{ $key }}
{{- end }}
{{- end }}


{{/*
--------------------------------------------------------------------------
Validate required values.
Проверяет что обязательные значения заданы.

Использование в начале deployment.yaml:
{{- include "mychart.validateValues" . }}
--------------------------------------------------------------------------
*/}}
{{- define "mychart.validateValues" -}}
{{- if not .Values.image.repository }}
  {{- fail "ERROR: image.repository is required!" }}
{{- end }}
{{- if and .Values.ingress.enabled (empty .Values.ingress.hosts) }}
  {{- fail "ERROR: ingress.hosts is required when ingress is enabled!" }}
{{- end }}
{{- end }}
```

### templates/deployment.yaml — полный пример

YAML

```
{{/*
==========================================================================
deployment.yaml — шаблон Kubernetes Deployment

Deployment управляет созданием и обновлением ReplicaSet → Pod'ов.
Это основной контроллер для stateless приложений.
==========================================================================
*/}}

{{- /* Вызываем валидацию обязательных значений */ -}}
{{- include "mychart.validateValues" . }}

apiVersion: apps/v1                                    # API версия для Deployment
kind: Deployment                                       # Тип ресурса
metadata:
  # Имя Deployment — генерируется из имени релиза и чарта
  # Пример: "production-myapp"
  name: {{ include "mychart.fullname" . }}
  
  # Namespace берётся из контекста релиза
  # Задаётся через helm install --namespace XXX
  namespace: {{ .Release.Namespace }}
  
  # Метки для Deployment (не для Pod'ов!)
  labels:
    {{- /* Включаем общие метки из _helpers.tpl */ -}}
    {{- include "mychart.labels" . | nindent 4 }}
  
  # Аннотации для Deployment
  # Могут использоваться для:
  # - Интеграции с ArgoCD, Flux
  # - Документирования
  # - Других инструментов
  {{- with .Values.deploymentAnnotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}

spec:
  # ===== КОЛИЧЕСТВО РЕПЛИК =====
  # Если включён autoscaling (HPA), replicas не указываем —
  # HPA будет управлять количеством реплик
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  
  # ===== ИСТОРИЯ РЕВИЗИЙ =====
  # Сколько старых ReplicaSet сохранять для возможности отката
  # По умолчанию Kubernetes хранит 10
  revisionHistoryLimit: {{ .Values.revisionHistoryLimit | default 10 }}
  
  # ===== СТРАТЕГИЯ ОБНОВЛЕНИЯ =====
  # Определяет как происходит rolling update
  {{- with .Values.updateStrategy }}
  strategy:
    {{- toYaml . | nindent 4 }}
  {{- end }}
  # Если не задано, используется default:
  # strategy:
  #   type: RollingUpdate
  #   rollingUpdate:
  #     maxUnavailable: 25%    # Максимум недоступных Pod'ов при обновлении
  #     maxSurge: 25%          # Максимум дополнительных Pod'ов при обновлении
  
  # ===== СЕЛЕКТОР =====
  # Определяет какие Pod'ы принадлежат этому Deployment
  # ВАЖНО: должен совпадать с labels в template.metadata.labels
  # НЕЛЬЗЯ менять после создания!
  selector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
  
  # ===== ШАБЛОН POD'А =====
  template:
    metadata:
      # Метки Pod'а — должны включать selectorLabels
      labels:
        {{- /* Обязательные метки для селектора */ -}}
        {{- include "mychart.selectorLabels" . | nindent 8 }}
        
        {{- /* Дополнительные метки из values */ -}}
        {{- with .Values.podLabels }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
      
      # Аннотации Pod'а
      annotations:
        # ===== CHECKSUM для автоматического перезапуска =====
        # Когда ConfigMap или Secret меняется, Pod не перезапускается автоматически.
        # Добавляя checksum содержимого в аннотацию, мы заставляем Pod перезапуститься
        # при изменении конфигурации, так как меняется Pod spec.
        
        {{- /* Checksum для ConfigMap — если он создаётся чартом */ -}}
        {{- if .Values.configMap }}
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
        {{- end }}
        
        {{- /* Checksum для Secret — если он создаётся чартом */ -}}
        {{- if .Values.secrets }}
        checksum/secret: {{ include (print $.Template.BasePath "/secret.yaml") . | sha256sum }}
        {{- end }}
        
        {{- /* Дополнительные аннотации из values */ -}}
        {{- with .Values.podAnnotations }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
    
    spec:
      # ===== IMAGE PULL SECRETS =====
      # Secrets для аутентификации в приватных container registries
      {{- with .Values.image.pullSecrets }}
      imagePullSecrets:
        {{- /* Поддерживаем оба формата: строки и объекты */ -}}
        {{- range . }}
          {{- if typeIs "string" . }}
        - name: {{ . }}
          {{- else }}
        - name: {{ .name }}
          {{- end }}
        {{- end }}
      {{- end }}
      
      # ===== SERVICE ACCOUNT =====
      # ServiceAccount определяет права Pod'а в кластере
      serviceAccountName: {{ include "mychart.serviceAccountName" . }}
      
      # Автоматически монтировать токен ServiceAccount
      # Если не нужен доступ к Kubernetes API — лучше отключить (false)
      automountServiceAccountToken: {{ .Values.serviceAccount.automountServiceAccountToken | default true }}
      
      # ===== SECURITY CONTEXT (POD LEVEL) =====
      # Настройки безопасности на уровне всего Pod'а
      {{- with .Values.podSecurityContext }}
      securityContext:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      
      # ===== INIT CONTAINERS =====
      # Контейнеры, которые запускаются ПЕРЕД основными
      # Используются для:
      # - Ожидания зависимостей (БД, другие сервисы)
      # - Инициализации данных
      # - Миграций БД
      # - Клонирования репозиториев
      {{- with .Values.initContainers }}
      initContainers:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      
      # ===== ОСНОВНЫЕ КОНТЕЙНЕРЫ =====
      containers:
        # ----- Главный контейнер приложения -----
        - name: {{ .Chart.Name }}
          
          # ===== ОБРАЗ =====
          # Формируем полный путь: registry/repository:tag
          {{- if .Values.image.registry }}
          image: "{{ .Values.image.registry }}/{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          {{- else }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          {{- end }}
          
          # Политика загрузки образа
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          
          # ===== SECURITY CONTEXT (CONTAINER LEVEL) =====
          # Настройки безопасности для этого контейнера
          {{- with .Values.securityContext }}
          securityContext:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          
          # ===== ПОРТЫ =====
          # Определяем какие порты слушает контейнер
          # ВАЖНО: Это только документация! Не открывает порты наружу.
          # Для доступа нужен Service.
          ports:
            # Основной HTTP порт
            - name: http                                    # Имя порта для ссылок
              containerPort: {{ .Values.service.targetPort | default 8080 }}  # Порт в контейнере
              protocol: TCP                                 # Протокол (TCP/UDP/SCTP)
            
            # Дополнительные порты (метрики, health, etc.)
            {{- range .Values.extraPorts }}
            - name: {{ .name }}
              containerPort: {{ .containerPort }}
              protocol: {{ .protocol | default "TCP" }}
            {{- end }}
          
          # ===== LIVENESS PROBE =====
          # Проверка что приложение живо
          # Если проверка падает N раз подряд — Pod перезапускается
          {{- with .Values.livenessProbe }}
          livenessProbe:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          
          # ===== READINESS PROBE =====
          # Проверка что приложение готово принимать трафик
          # Если падает — Pod убирается из Service endpoints (не получает трафик)
          # Pod НЕ перезапускается (в отличие от liveness)
          {{- with .Values.readinessProbe }}
          readinessProbe:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          
          # ===== STARTUP PROBE =====
          # Проверка начального запуска (Kubernetes 1.16+)
          # Пока не пройдёт — liveness и readiness не проверяются
          # Полезно для приложений с долгой инициализацией
          {{- with .Values.startupProbe }}
          startupProbe:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          
          # ===== ПЕРЕМЕННЫЕ ОКРУЖЕНИЯ =====
          # Способ передать конфигурацию в приложение
          {{- if or .Values.env .Values.extraEnv }}
          env:
            # Переменные из values.env
            {{- with .Values.env }}
            {{- toYaml . | nindent 12 }}
            {{- end }}
            
            # Дополнительные переменные
            {{- with .Values.extraEnv }}
            {{- toYaml . | nindent 12 }}
            {{- end }}
          {{- end }}
          
          # Загрузка переменных из ConfigMap/Secret целиком
          {{- with .Values.envFrom }}
          envFrom:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          
          # ===== РЕСУРСЫ =====
          # Лимиты и запросы CPU/Memory
          # ВСЕГДА задавайте в production!
          {{- with .Values.resources }}
          resources:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          
          # ===== VOLUME MOUNTS =====
          # Точки монтирования томов в контейнере
          volumeMounts:
            # Монтируем ConfigMap как файл конфигурации
            {{- if .Values.configMap }}
            - name: config
              mountPath: /etc/config
              readOnly: true
            {{- end }}
            
            # Монтируем PVC для постоянного хранилища
            {{- if .Values.persistence.enabled }}
            - name: data
              mountPath: {{ .Values.persistence.mountPath | default "/data" }}
              {{- with .Values.persistence.subPath }}
              subPath: {{ . }}
              {{- end }}
            {{- end }}
            
            # Дополнительные volume mounts из values
            {{- with .Values.extraVolumeMounts }}
            {{- toYaml . | nindent 12 }}
            {{- end }}
          
          # ===== LIFECYCLE HOOKS =====
          # Хуки жизненного цикла контейнера
          {{- with .Values.lifecycle }}
          lifecycle:
            {{- toYaml . | nindent 12 }}
          {{- end }}
        
        # ----- Sidecar контейнеры -----
        # Дополнительные контейнеры в том же Pod'е
        # Например: log shipper, proxy, monitoring agent
        {{- with .Values.sidecars }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
      
      # ===== VOLUMES =====
      # Определение томов для Pod'а
      volumes:
        # ConfigMap как том
        {{- if .Values.configMap }}
        - name: config
          configMap:
            name: {{ include "mychart.fullname" . }}-config
            # Можно указать конкретные ключи и права
            # items:
            #   - key: config.yaml
            #     path: config.yaml
            #     mode: 0644
        {{- end }}
        
        # PersistentVolumeClaim для постоянного хранилища
        {{- if .Values.persistence.enabled }}
        - name: data
          persistentVolumeClaim:
            # Если указан existingClaim — используем его
            # Иначе — используем PVC созданный этим чартом
            claimName: {{ .Values.persistence.existingClaim | default (include "mychart.fullname" .) }}
        {{- end }}
        
        # Дополнительные тома из values
        {{- with .Values.extraVolumes }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
      
      # ===== NODE SELECTOR =====
      # Простой способ выбрать ноды для Pod'а по меткам
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      
      # ===== AFFINITY =====
      # Продвинутые правила размещения Pod'ов
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      
      # ===== TOLERATIONS =====
      # Разрешения для размещения на нодах с taints
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      
      # ===== PRIORITY =====
      # Приоритет Pod'а при планировании и eviction
      {{- with .Values.priorityClassName }}
      priorityClassName: {{ . }}
      {{- end }}
      
      # ===== DNS =====
      # Политика DNS для Pod'а
      dnsPolicy: {{ .Values.dnsPolicy | default "ClusterFirst" }}
      
      # Кастомная DNS конфигурация
      {{- with .Values.dnsConfig }}
      dnsConfig:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      
      # ===== TERMINATION =====
      # Время ожидания graceful shutdown
      terminationGracePeriodSeconds: {{ .Values.terminationGracePeriodSeconds | default 30 }}
      
      # Политика перезапуска
      restartPolicy: {{ .Values.restartPolicy | default "Always" }}
      
      # ===== SCHEDULING =====
      # Имя Scheduler (для кастомных scheduler'ов)
      {{- with .Values.schedulerName }}
      schedulerName: {{ . }}
      {{- end }}
      
      # Имя ноды (жёсткая привязка — НЕ рекомендуется)
      {{- with .Values.nodeName }}
      nodeName: {{ . }}
      {{- end }}
      
      # ===== TOPOLOGY =====
      # Topology Spread Constraints — распределение по зонам/нодам
      {{- with .Values.topologySpreadConstraints }}
      topologySpreadConstraints:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

### templates/service.yaml — полный пример

YAML

```
{{/*
==========================================================================
service.yaml — шаблон Kubernetes Service

Service обеспечивает стабильный сетевой endpoint для доступа к Pod'ам.
Pod'ы могут появляться и исчезать, но Service остаётся.
==========================================================================
*/}}

{{- /* Создаём Service только если он включён */ -}}
{{- if .Values.service.enabled | default true }}

apiVersion: v1                                         # Core API
kind: Service                                          # Тип ресурса
metadata:
  # Имя Service — обычно совпадает с именем приложения
  name: {{ include "mychart.fullname" . }}
  
  namespace: {{ .Release.Namespace }}
  
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
  
  # Аннотации Service
  # Часто используются для:
  # - Настройки LoadBalancer (AWS, GCP, Azure)
  # - External DNS
  # - Service Mesh (Istio, Linkerd)
  {{- with .Values.service.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}

spec:
  # ===== ТИП SERVICE =====
  # ClusterIP — доступен только внутри кластера (по умолчанию)
  # NodePort — доступен на порту каждой ноды
  # LoadBalancer — создаёт внешний LoadBalancer (в облаке)
  # ExternalName — CNAME на внешний сервис
  type: {{ .Values.service.type | default "ClusterIP" }}
  
  # ===== ПОРТЫ =====
  ports:
    # Основной порт
    - port: {{ .Values.service.port | default 80 }}     # Порт Service (на который обращаются клиенты)
      targetPort: {{ .Values.service.targetPort | default "http" }}  # Порт в Pod (имя или число)
      protocol: TCP                                      # Протокол
      name: http                                         # Имя порта (для ссылок)
      
      # NodePort — только для type: NodePort или LoadBalancer
      # Порт на каждой ноде кластера (диапазон 30000-32767)
      {{- if and (eq .Values.service.type "NodePort") .Values.service.nodePort }}
      nodePort: {{ .Values.service.nodePort }}
      {{- end }}
    
    # Дополнительные порты
    {{- range .Values.service.extraPorts }}
    - port: {{ .port }}
      targetPort: {{ .targetPort }}
      protocol: {{ .protocol | default "TCP" }}
      name: {{ .name }}
      {{- if and (eq $.Values.service.type "NodePort") .nodePort }}
      nodePort: {{ .nodePort }}
      {{- end }}
    {{- end }}
  
  # ===== СЕЛЕКТОР =====
  # Определяет к каким Pod'ам направляется трафик
  # Должен совпадать с labels Pod'ов в Deployment
  selector:
    {{- include "mychart.selectorLabels" . | nindent 4 }}
  
  # ===== LOADBALANCER СПЕЦИФИЧНЫЕ НАСТРОЙКИ =====
  
  # Статический IP для LoadBalancer
  {{- if and (eq .Values.service.type "LoadBalancer") .Values.service.loadBalancerIP }}
  loadBalancerIP: {{ .Values.service.loadBalancerIP }}
  {{- end }}
  
  # Whitelist IP для LoadBalancer
  {{- with .Values.service.loadBalancerSourceRanges }}
  loadBalancerSourceRanges:
    {{- toYaml . | nindent 4 }}
  {{- end }}
  
  # Класс LoadBalancer (для кластеров с несколькими LB контроллерами)
  {{- with .Values.service.loadBalancerClass }}
  loadBalancerClass: {{ . }}
  {{- end }}
  
  # Сохранять исходный IP клиента
  # Local — трафик идёт только на ноду с Pod'ом, сохраняет IP
  # Cluster — трафик распределяется, IP может потеряться
  {{- if .Values.service.externalTrafficPolicy }}
  externalTrafficPolicy: {{ .Values.service.externalTrafficPolicy }}
  {{- end }}
  
  # ===== SESSION AFFINITY =====
  # Привязка сессии к конкретному Pod'у
  # None — без привязки (по умолчанию)
  # ClientIP — запросы с одного IP идут на один Pod
  {{- if .Values.service.sessionAffinity }}
  sessionAffinity: {{ .Values.service.sessionAffinity }}
  
  {{- with .Values.service.sessionAffinityConfig }}
  sessionAffinityConfig:
    {{- toYaml . | nindent 4 }}
  {{- end }}
  {{- end }}
  
  # ===== CLUSTER IP =====
  # Можно задать конкретный ClusterIP (обычно не нужно)
  {{- with .Values.service.clusterIP }}
  clusterIP: {{ . }}
  {{- end }}
  
  # Для headless service (StatefulSet)
  # clusterIP: None
  
  # ===== EXTERNAL IPs =====
  # Внешние IP адреса для доступа к Service
  {{- with .Values.service.externalIPs }}
  externalIPs:
    {{- toYaml . | nindent 4 }}
  {{- end }}
  
  # ===== IP FAMILIES =====
  # Поддержка IPv4/IPv6 dual-stack
  {{- with .Values.service.ipFamilies }}
  ipFamilies:
    {{- toYaml . | nindent 4 }}
  {{- end }}
  
  {{- with .Values.service.ipFamilyPolicy }}
  ipFamilyPolicy: {{ . }}
  {{- end }}

{{- end }}
```

### templates/NOTES.txt — сообщение после установки

YAML

```
{{/*
==========================================================================
NOTES.txt — сообщение, отображаемое после helm install/upgrade

Здесь обычно показывают:
- Как получить доступ к приложению
- Полезные команды
- Важные предупреждения
- Ссылки на документацию
==========================================================================
*/}}

==========================================================
  {{ .Chart.Name }} {{ .Chart.Version }} успешно установлен!
==========================================================

{{ if .Values.ingress.enabled -}}
{{/* ========== ВАРИАНТ 1: INGRESS ВКЛЮЧЁН ========== */}}

🌐 Ваше приложение доступно по адресу:
{{- range .Values.ingress.hosts }}
  {{- range .paths }}
  http{{ if $.Values.ingress.tls }}s{{ end }}://{{ .host }}{{ .path }}
  {{- end }}
{{- end }}

{{- else if contains "NodePort" .Values.service.type -}}
{{/* ========== ВАРИАНТ 2: NODEPORT ========== */}}

🌐 Ваше приложение доступно на порту NodePort.

Получите адрес командами:

  export NODE_PORT=$(kubectl get --namespace {{ .Release.Namespace }} \
    -o jsonpath="{.spec.ports[0].nodePort}" \
    services {{ include "mychart.fullname" . }})
    
  export NODE_IP=$(kubectl get nodes --namespace {{ .Release.Namespace }} \
    -o jsonpath="{.items[0].status.addresses[0].address}")
    
  echo "Приложение: http://$NODE_IP:$NODE_PORT"

{{- else if contains "LoadBalancer" .Values.service.type -}}
{{/* ========== ВАРИАНТ 3: LOADBALANCER ========== */}}

🌐 Ваше приложение доступно через LoadBalancer.

⏳ Получение IP может занять несколько минут...

Получите IP командой:

  export SERVICE_IP=$(kubectl get svc --namespace {{ .Release.Namespace }} \
    {{ include "mychart.fullname" . }} \
    -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    
  echo "Приложение: http://$SERVICE_IP:{{ .Values.service.port }}"

Для облачных провайдеров может быть hostname вместо IP:

  kubectl get svc --namespace {{ .Release.Namespace }} \
    {{ include "mychart.fullname" . }} \
    -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

{{- else -}}
{{/* ========== ВАРИАНТ 4: CLUSTERIP (по умолчанию) ========== */}}

🔒 Ваше приложение доступно только внутри кластера (ClusterIP).

Для локального доступа используйте port-forward:

  export POD_NAME=$(kubectl get pods --namespace {{ .Release.Namespace }} \
    -l "app.kubernetes.io/name={{ include "mychart.name" . }},app.kubernetes.io/instance={{ .Release.Name }}" \
    -o jsonpath="{.items[0].metadata.name}")
    
  kubectl --namespace {{ .Release.Namespace }} port-forward $POD_NAME 8080:{{ .Values.service.targetPort | default 8080 }}

  echo "Приложение: http://127.0.0.1:8080"

{{- end }}

==========================================================
📋 ПОЛЕЗНЫЕ КОМАНДЫ
==========================================================

# Проверить статус Pod'ов:
kubectl get pods --namespace {{ .Release.Namespace }} \
  -l "app.kubernetes.io/instance={{ .Release.Name }}"

# Посмотреть логи:
kubectl logs --namespace {{ .Release.Namespace }} \
  -l "app.kubernetes.io/instance={{ .Release.Name }}" -f

# Описание Pod'а (для отладки):
kubectl describe pod --namespace {{ .Release.Namespace }} \
  -l "app.kubernetes.io/instance={{ .Release.Name }}"

# Зайти в контейнер:
kubectl exec -it --namespace {{ .Release.Namespace }} \
  $(kubectl get pods --namespace {{ .Release.Namespace }} \
    -l "app.kubernetes.io/instance={{ .Release.Name }}" \
    -o jsonpath="{.items[0].metadata.name}") \
  -- /bin/sh

{{- if .Values.metrics.enabled }}

# Проверить метрики (если включены):
kubectl port-forward --namespace {{ .Release.Namespace }} \
  svc/{{ include "mychart.fullname" . }} 9090:9090

{{- end }}

==========================================================
⚠️  ВАЖНАЯ ИНФОРМАЦИЯ
==========================================================

{{- if not .Values.persistence.enabled }}

⚠️  ВНИМАНИЕ: Persistent storage ВЫКЛЮЧЕН!
    Данные будут потеряны при перезапуске Pod'а.
    Для production включите: --set persistence.enabled=true
{{- end }}

{{- if not .Values.resources }}

⚠️  ВНИМАНИЕ: Resource limits НЕ ЗАДАНЫ!
    В production обязательно задайте:
    --set resources.limits.cpu=500m
    --set resources.limits.memory=512Mi
{{- end }}

{{- if .Values.ingress.enabled }}
{{- if not .Values.ingress.tls }}

⚠️  ВНИМАНИЕ: TLS НЕ НАСТРОЕН для Ingress!
    В production используйте HTTPS.
{{- end }}
{{- end }}

==========================================================
📚 ДОКУМЕНТАЦИЯ
==========================================================

Helm чарт: {{ .Chart.Home | default "N/A" }}
Приложение: {{ .Chart.Home | default "N/A" }}
{{- range .Chart.Sources }}
Исходный код: {{ . }}
{{- end }}

Версия чарта: {{ .Chart.Version }}
Версия приложения: {{ .Chart.AppVersion }}

==========================================================
```

## 4. Несколько values-файлов — организация конфигурации

### Концепция разделения конфигурации

В реальных проектах одно приложение деплоится в несколько окружений: development, staging, production. Каждое окружение имеет свои особенности:

text

```
┌─────────────────────────────────────────────────────────────────────┐
│                        DEVELOPMENT                                   │
├─────────────────────────────────────────────────────────────────────┤
│  • 1 реплика (экономия ресурсов)                                    │
│  • Минимальные ресурсы (cpu: 100m, memory: 128Mi)                   │
│  • Образ с тегом "develop" или "latest"                             │
│  • Debug режим включён                                               │
│  • Ingress на dev-домене                                            │
│  • Встроенная БД (PostgreSQL в том же кластере)                     │
│  • Логирование: DEBUG уровень                                        │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                         STAGING                                      │
├─────────────────────────────────────────────────────────────────────┤
│  • 2 реплики (тестирование HA)                                      │
│  • Средние ресурсы (cpu: 250m, memory: 256Mi)                       │
│  • Образ с конкретной версией (тег релиза)                          │
│  • Debug режим выключен                                              │
│  • Ingress на staging-домене                                        │
│  • Встроенная БД или тестовая managed БД                            │
│  • Логирование: INFO уровень                                         │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                        PRODUCTION                                    │
├─────────────────────────────────────────────────────────────────────┤
│  • 5+ реплик (высокая доступность)                                  │
│  • HPA включён (автомасштабирование 3-20 реплик)                    │
│  • Максимальные ресурсы (cpu: 1000m, memory: 1Gi)                   │
│  • Образ с конкретной стабильной версией                            │
│  • PodDisruptionBudget (минимум 2 реплики всегда доступны)          │
│  • Ingress на production-домене с TLS                               │
│  • Внешняя managed БД (RDS, Cloud SQL)                              │
│  • Логирование: WARN уровень                                         │
│  • Мониторинг и алерты включены                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Структура проекта с несколькими окружениями

text

```
my-project/
│
├── charts/                          # Директория с чартами
│   └── myapp/                       # Чарт приложения
│       ├── Chart.yaml
│       ├── values.yaml              # Базовые значения по умолчанию
│       ├── templates/
│       │   ├── _helpers.tpl
│       │   ├── deployment.yaml
│       │   ├── service.yaml
│       │   ├── ingress.yaml
│       │   └── ...
│       └── charts/                  # Зависимости
│
├── environments/                    # Конфигурации для окружений
│   │
│   ├── values-common.yaml           # Общие настройки для ВСЕХ окружений
│   │                                # То, что не меняется между env
│   │
│   ├── values-dev.yaml              # Специфичные для development
│   ├── values-staging.yaml          # Специфичные для staging
│   ├── values-prod.yaml             # Специфичные для production
│   │
│   └── secrets/                     # Секреты (НЕ коммитить в git!)
│       ├── values-dev-secrets.yaml
│       ├── values-staging-secrets.yaml
│       └── values-prod-secrets.yaml
│
├── scripts/                         # Скрипты для деплоя
│   ├── deploy-dev.sh
│   ├── deploy-staging.sh
│   └── deploy-prod.sh
│
└── Makefile                         # Команды для удобства
```

### values.yaml (в чарте) — минимальные defaults

YAML

```
# charts/myapp/values.yaml
#
# Это БАЗОВЫЕ значения по умолчанию, встроенные в чарт.
# Они должны позволять чарту работать "из коробки" для разработки.
# Все значения могут быть переопределены через -f или --set.

# ===== ОСНОВНЫЕ ПАРАМЕТРЫ =====

# Количество реплик по умолчанию
# В production будет переопределено на большее число
replicaCount: 1

# ===== ОБРАЗ =====

image:
  # Registry по умолчанию — Docker Hub
  # В production может быть приватный registry
  registry: ""
  
  # Репозиторий образа
  # ДОЛЖЕН быть переопределён для реального использования
  repository: nginx
  
  # Тег по умолчанию — latest (только для разработки!)
  # В production ВСЕГДА используйте конкретную версию
  tag: ""
  
  # Политика загрузки
  pullPolicy: IfNotPresent
  
  # Secrets для приватного registry
  pullSecrets: []

# ===== NAMING =====

# Переопределение имён (обычно не нужно)
nameOverride: ""
fullnameOverride: ""

# ===== SERVICE ACCOUNT =====

serviceAccount:
  create: true
  annotations: {}
  name: ""
  automountServiceAccountToken: true

# ===== БЕЗОПАСНОСТЬ =====

# Базовые настройки безопасности
podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000

securityContext:
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
  readOnlyRootFilesystem: true

# ===== SERVICE =====

service:
  enabled: true
  type: ClusterIP
  port: 80
  targetPort: 8080
  annotations: {}

# ===== INGRESS =====

# По умолчанию выключен — не все окружения имеют Ingress Controller
ingress:
  enabled: false
  className: ""
  annotations: {}
  hosts: []
  tls: []

# ===== РЕСУРСЫ =====

# Минимальные ресурсы по умолчанию
# В production будут значительно увеличены
resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi

# ===== AUTOSCALING =====

# По умолчанию выключен
autoscaling:
  enabled: false
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80

# ===== PROBES =====

# Базовые health checks
livenessProbe:
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: http
  initialDelaySeconds: 10
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3

startupProbe: {}

# ===== PERSISTENCE =====

# По умолчанию выключено
persistence:
  enabled: false
  storageClass: ""
  accessMode: ReadWriteOnce
  size: 1Gi
  mountPath: /data

# ===== NODE ASSIGNMENT =====

nodeSelector: {}
tolerations: []
affinity: {}

# ===== ENVIRONMENT =====

# Переменные окружения
env: []
envFrom: []

# ===== ДОПОЛНИТЕЛЬНЫЕ РЕСУРСЫ =====

# ConfigMap (создаётся если не пустой)
configMap: {}

# Secrets (НЕ используйте в values.yaml чарта!)
secrets: {}

# ===== FEATURES =====

# Дополнительные возможности (по умолчанию выключены)
metrics:
  enabled: false
  serviceMonitor:
    enabled: false

podDisruptionBudget:
  enabled: false

networkPolicy:
  enabled: false
```

### values-common.yaml — общее для всех окружений

YAML

```
# environments/values-common.yaml
#
# Общие настройки, которые одинаковы для ВСЕХ окружений.
# Это то, что не зависит от того, куда мы деплоим.

# ===== ОБРАЗ =====

image:
  # Приватный container registry компании
  # Одинаковый для всех окружений
  registry: registry.example.com
  
  # Репозиторий приложения
  repository: mycompany/myapp
  
  # Политика загрузки — всегда проверять наличие обновлений
  # для образов с тегами (кроме immutable тегов с SHA)
  pullPolicy: IfNotPresent
  
  # Secret для доступа к приватному registry
  # Этот Secret должен существовать в каждом namespace
  pullSecrets:
    - name: registry-credentials

# ===== SERVICE =====

service:
  enabled: true
  type: ClusterIP
  port: 80
  targetPort: 8080
  
  # Аннотации для service mesh (если используется)
  annotations:
    # Prometheus scraping через Service
    prometheus.io/scrape: "true"
    prometheus.io/port: "9090"
    prometheus.io/path: "/metrics"

# ===== INGRESS =====

# Общие настройки Ingress (хосты разные в каждом окружении)
ingress:
  enabled: true
  className: nginx
  
  # Аннотации общие для всех окружений
  annotations:
    # Cert-manager для автоматических TLS сертификатов
    cert-manager.io/cluster-issuer: letsencrypt-prod
    
    # Nginx Ingress настройки
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "60"

# ===== БЕЗОПАСНОСТЬ =====

# Единые стандарты безопасности для всей компании
podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000
  runAsGroup: 1000
  fsGroup: 1000
  seccompProfile:
    type: RuntimeDefault

securityContext:
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
  readOnlyRootFilesystem: true
  runAsNonRoot: true

# ===== SERVICE ACCOUNT =====

serviceAccount:
  create: true
  automountServiceAccountToken: false
  
  # AWS EKS IRSA — IAM роль для ServiceAccount
  # Роль разная в каждом аккаунте, но структура аннотации одинаковая
  annotations: {}

# ===== PROBES =====

# Одинаковые endpoints для health checks во всех окружениях
livenessProbe:
  httpGet:
    path: /api/health
    port: http
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  successThreshold: 1
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /api/ready
    port: http
  initialDelaySeconds: 10
  periodSeconds: 5
  timeoutSeconds: 3
  successThreshold: 1
  failureThreshold: 3

# Startup probe для приложений с долгой инициализацией
startupProbe:
  httpGet:
    path: /api/startup
    port: http
  failureThreshold: 30
  periodSeconds: 10

# ===== МЕТКИ =====

# Общие метки для всех ресурсов
podLabels:
  app.kubernetes.io/component: backend
  app.kubernetes.io/part-of: my-platform

# ===== LIFECYCLE =====

# Graceful shutdown — дать приложению время завершить запросы
lifecycle:
  preStop:
    exec:
      # Подождать 15 секунд перед SIGTERM
      # Это даёт время Ingress/Service убрать Pod из endpoints
      command:
        - /bin/sh
        - -c
        - sleep 15

# Время ожидания graceful shutdown
terminationGracePeriodSeconds: 45

# ===== МОНИТОРИНГ =====

# Метрики Prometheus (общие настройки)
metrics:
  enabled: true
  
  # ServiceMonitor для Prometheus Operator
  serviceMonitor:
    enabled: true
    interval: 30s
    scrapeTimeout: 10s
    
    # Метки для обнаружения Prometheus
    labels:
      release: prometheus

# ===== ОБЩИЕ ПЕРЕМЕННЫЕ ОКРУЖЕНИЯ =====

# Переменные, одинаковые для всех окружений
env:
  # Формат логов
  - name: LOG_FORMAT
    value: json
  
  # Таймзона
  - name: TZ
    value: UTC
  
  # Включить graceful shutdown
  - name: GRACEFUL_SHUTDOWN_TIMEOUT
    value: "30"
```

### values-dev.yaml — development окружение

YAML

```
# environments/values-dev.yaml
#
# Конфигурация для development окружения.
# Фокус на: простота, скорость итераций, минимальные ресурсы.

# ===== ОСНОВНЫЕ ПАРАМЕТРЫ =====

# Одна реплика достаточно для разработки
replicaCount: 1

# ===== ОБРАЗ =====

image:
  # Тег для разработки — последняя сборка из develop ветки
  # В реальности лучше использовать конкретные версии даже в dev
  tag: "develop"
  
  # Always — чтобы всегда получать свежий образ
  pullPolicy: Always

# ===== INGRESS =====

ingress:
  hosts:
    - host: myapp-dev.example.com
      paths:
        - path: /
          pathType: Prefix
  
  tls:
    - secretName: myapp-dev-tls
      hosts:
        - myapp-dev.example.com

# ===== РЕСУРСЫ =====

# Минимальные ресурсы для dev
resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 50m
    memory: 128Mi

# ===== AUTOSCALING =====

# Отключен в dev — не нужен
autoscaling:
  enabled: false

# ===== ПЕРЕМЕННЫЕ ОКРУЖЕНИЯ =====

env:
  # Уровень логирования — максимально подробный
  - name: LOG_LEVEL
    value: debug
  
  # Режим разработки
  - name: APP_ENV
    value: development
  
  # Включить debug endpoints
  - name: ENABLE_DEBUG
    value: "true"
  
  # Подключение к dev БД (внутри кластера)
  - name: DATABASE_HOST
    value: myapp-postgresql.dev.svc.cluster.local
  
  - name: DATABASE_PORT
    value: "5432"
  
  - name: DATABASE_NAME
    value: myapp_dev

# ===== БАЗА ДАННЫХ =====

# Включить PostgreSQL как зависимость (в том же кластере)
postgresql:
  enabled: true
  
  auth:
    username: myapp
    database: myapp_dev
    # Пароль для dev можно захардкодить (не production!)
    password: devpassword123
  
  primary:
    persistence:
      size: 1Gi
    
    resources:
      limits:
        cpu: 200m
        memory: 256Mi
      requests:
        cpu: 100m
        memory: 128Mi

# ===== PERSISTENCE =====

# Отключено в dev — данные можно пересоздать
persistence:
  enabled: false

# ===== POD DISRUPTION BUDGET =====

# Не нужен в dev
podDisruptionBudget:
  enabled: false

# ===== NETWORK POLICY =====

# Не нужен в dev — упрощает отладку
networkPolicy:
  enabled: false

# ===== ANNOTATIONS =====

podAnnotations:
  # Дополнительные аннотации для dev
  environment: development
  
  # Отключить некоторые политики безопасности для отладки
  # ТОЛЬКО ДЛЯ DEV!
  # sidecar.istio.io/inject: "false"

# ===== NODE ASSIGNMENT =====

# В dev можно использовать spot/preemptible instances для экономии
tolerations:
  - key: "cloud.google.com/gke-preemptible"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  
  - key: "kubernetes.azure.com/scalesetpriority"
    operator: "Equal"
    value: "spot"
    effect: "NoSchedule"

nodeSelector:
  # Использовать ноды для dev workloads
  workload-type: development
```

### values-staging.yaml — staging окружение

YAML

```
# environments/values-staging.yaml
#
# Конфигурация для staging окружения.
# Staging максимально похож на production, но с меньшими ресурсами.
# Используется для финального тестирования перед релизом.

# ===== ОСНОВНЫЕ ПАРАМЕТРЫ =====

# 2 реплики — тестируем высокую доступность
replicaCount: 2

# ===== ОБРАЗ =====

image:
  # Конкретная версия релиз-кандидата
  # Обычно переопределяется через CI/CD: --set image.tag=$VERSION
  tag: "1.2.3-rc.1"
  
  pullPolicy: IfNotPresent

# ===== INGRESS =====

ingress:
  hosts:
    - host: myapp-staging.example.com
      paths:
        - path: /
          pathType: Prefix
  
  tls:
    - secretName: myapp-staging-tls
      hosts:
        - myapp-staging.example.com
  
  # Дополнительные аннотации для staging
  annotations:
    # Ограничить доступ по IP (только офис/VPN)
    nginx.ingress.kubernetes.io/whitelist-source-range: "10.0.0.0/8,192.168.0.0/16"

# ===== РЕСУРСЫ =====

# Средние ресурсы — между dev и prod
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 200m
    memory: 256Mi

# ===== AUTOSCALING =====

# Включён для тестирования, но с меньшими лимитами
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 5
  targetCPUUtilizationPercentage: 80

# ===== ПЕРЕМЕННЫЕ ОКРУЖЕНИЯ =====

env:
  # Уровень логирования — INFO
  - name: LOG_LEVEL
    value: info
  
  # Режим staging
  - name: APP_ENV
    value: staging
  
  # Debug выключен
  - name: ENABLE_DEBUG
    value: "false"
  
  # Подключение к staging БД
  - name: DATABASE_HOST
    value: myapp-postgresql.staging.svc.cluster.local
  
  - name: DATABASE_PORT
    value: "5432"
  
  - name: DATABASE_NAME
    value: myapp_staging

# ===== БАЗА ДАННЫХ =====

# PostgreSQL в кластере для staging
postgresql:
  enabled: true
  
  auth:
    username: myapp
    database: myapp_staging
    # Пароль из Secret (см. values-staging-secrets.yaml)
    existingSecret: myapp-postgresql-credentials
    secretKeys:
      userPasswordKey: password
  
  primary:
    persistence:
      size: 5Gi
    
    resources:
      limits:
        cpu: 500m
        memory: 512Mi
      requests:
        cpu: 250m
        memory: 256Mi

# ===== PERSISTENCE =====

# Включено для тестирования persistence
persistence:
  enabled: true
  size: 5Gi
  storageClass: standard

# ===== POD DISRUPTION BUDGET =====

# Включён — тестируем поведение при disruptions
podDisruptionBudget:
  enabled: true
  minAvailable: 1

# ===== NETWORK POLICY =====

# Включён — тестируем сетевые политики
networkPolicy:
  enabled: true
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: ingress-nginx
      ports:
        - protocol: TCP
          port: 8080

# ===== NODE ASSIGNMENT =====

# Staging на обычных нодах (не spot)
tolerations: []

nodeSelector:
  workload-type: staging

# Anti-affinity — распределить реплики по разным нодам
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
              - key: app.kubernetes.io/name
                operator: In
                values:
                  - myapp
          topologyKey: kubernetes.io/hostname
```

### values-prod.yaml — production окружение

YAML

```
# environments/values-prod.yaml
#
# Конфигурация для PRODUCTION окружения.
# Максимальная надёжность, производительность, безопасность.
# Каждый параметр тщательно продуман.

# ===== ОСНОВНЫЕ ПАРАМЕТРЫ =====

# Базовое количество реплик (HPA может увеличить)
replicaCount: 5

# ===== ОБРАЗ =====

image:
  # Только конкретные стабильные версии!
  # НИКОГДА не используйте "latest" в production
  # Версия обычно устанавливается через CI/CD: --set image.tag=$VERSION
  tag: "1.2.3"
  
  # IfNotPresent — образ с конкретным тегом не меняется
  pullPolicy: IfNotPresent

# ===== INGRESS =====

ingress:
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix
    
    # Дополнительный домен (если нужно)
    - host: www.myapp.example.com
      paths:
        - path: /
          pathType: Prefix
  
  tls:
    - secretName: myapp-prod-tls
      hosts:
        - myapp.example.com
        - www.myapp.example.com
  
  # Production-specific аннотации
  annotations:
    # Rate limiting
    nginx.ingress.kubernetes.io/limit-rps: "100"
    nginx.ingress.kubernetes.io/limit-connections: "50"
    
    # Таймауты
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "10"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "60"
    
    # HSTS (HTTP Strict Transport Security)
    nginx.ingress.kubernetes.io/configuration-snippet: |
      add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

# ===== РЕСУРСЫ =====

# Production-grade ресурсы
# Значения определены на основе нагрузочного тестирования
resources:
  limits:
    # Лимит CPU — контейнер будет throttled при превышении
    cpu: 1000m
    
    # Лимит памяти — контейнер будет убит (OOMKilled) при превышении
    memory: 1Gi
  
  requests:
    # Запросы — гарантированные ресурсы
    # Kubernetes планирует Pod только если на ноде есть столько ресурсов
    cpu: 500m
    memory: 512Mi

# ===== AUTOSCALING =====

autoscaling:
  enabled: true
  
  # Минимум — для обеспечения HA даже при низкой нагрузке
  minReplicas: 3
  
  # Максимум — ограничение для контроля затрат и ресурсов кластера
  maxReplicas: 20
  
  # Целевая утилизация CPU (от requests)
  # 70% — оставляем запас для всплесков
  targetCPUUtilizationPercentage: 70
  
  # Целевая утилизация памяти
  targetMemoryUtilizationPercentage: 80
  
  # Поведение масштабирования (Kubernetes 1.18+)
  behavior:
    scaleDown:
      # Стабилизационное окно — предотвращает "flapping"
      stabilizationWindowSeconds: 300
      policies:
        # Уменьшать не более чем на 2 Pod'а за 60 секунд
        - type: Pods
          value: 2
          periodSeconds: 60
    
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        # Увеличивать до 4 Pod'ов за 15 секунд
        - type: Pods
          value: 4
          periodSeconds: 15
        # Или до 200% от текущего количества
        - type: Percent
          value: 200
          periodSeconds: 15
      selectPolicy: Max

# ===== ПЕРЕМЕННЫЕ ОКРУЖЕНИЯ =====

env:
  # Уровень логирования — только важное
  - name: LOG_LEVEL
    value: warn
  
  # Режим production
  - name: APP_ENV
    value: production
  
  # Debug полностью выключен
  - name: ENABLE_DEBUG
    value: "false"
  
  # Connection pool для БД
  - name: DATABASE_POOL_SIZE
    value: "20"
  
  - name: DATABASE_POOL_TIMEOUT
    value: "30"

# Переменные из Secrets
envFrom:
  - secretRef:
      # Secret с credentials для внешних сервисов
      name: myapp-credentials

# ===== БАЗА ДАННЫХ =====

# В production используем ВНЕШНЮЮ managed базу данных
# (AWS RDS, Google Cloud SQL, Azure Database)
postgresql:
  # Не устанавливаем PostgreSQL из чарта
  enabled: false

# Настройки для подключения к внешней БД
externalDatabase:
  # Хост managed базы данных
  host: myapp-db.cluster-abc123.us-east-1.rds.amazonaws.com
  
  port: 5432
  database: myapp_production
  
  # Credentials в отдельном Secret (создаётся вне Helm)
  existingSecret: myapp-database-credentials
  secretKeys:
    username: username
    password: password

# ===== PERSISTENCE =====

persistence:
  enabled: true
  
  # Production StorageClass с репликацией
  storageClass: gp3
  
  size: 50Gi
  
  # Аннотации для backup
  annotations:
    backup.velero.io/backup-volumes: data

# ===== POD DISRUPTION BUDGET =====

podDisruptionBudget:
  enabled: true
  
  # Минимум 2 Pod'а должны быть доступны всегда
  # Это критично для HA при обновлениях и node drains
  minAvailable: 2

# ===== NETWORK POLICY =====

networkPolicy:
  enabled: true
  
  policyTypes:
    - Ingress
    - Egress
  
  # Входящий трафик
  ingress:
    # Только от Ingress Controller
    - from:
        - namespaceSelector:
            matchLabels:
              name: ingress-nginx
      ports:
        - protocol: TCP
          port: 8080
    
    # От Prometheus для метрик
    - from:
        - namespaceSelector:
            matchLabels:
              name: monitoring
      ports:
        - protocol: TCP
          port: 9090
  
  # Исходящий трафик
  egress:
    # DNS
    - to:
        - namespaceSelector: {}
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
    
    # К базе данных
    - to:
        - ipBlock:
            cidr: 10.0.0.0/8
      ports:
        - protocol: TCP
          port: 5432

# ===== NODE ASSIGNMENT =====

# Production workloads на dedicated нодах
nodeSelector:
  workload-type: production
  
  # Только ноды в конкретных зонах доступности
  # topology.kubernetes.io/zone: us-east-1a

# Никаких tolerations для spot instances в production!
tolerations: []

# Распределение по зонам доступности и нодам
affinity:
  # Распределить Pod'ы по разным нодам (HA)
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app.kubernetes.io/name
              operator: In
              values:
                - myapp
        topologyKey: kubernetes.io/hostname
    
    # Желательно распределить по зонам доступности
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
              - key: app.kubernetes.io/name
                operator: In
                values:
                  - myapp
          topologyKey: topology.kubernetes.io/zone

# Распределение по зонам (Kubernetes 1.19+)
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: ScheduleAnyway
    labelSelector:
      matchLabels:
        app.kubernetes.io/name: myapp

# ===== PRIORITY =====

# Высокий приоритет для production workloads
# PriorityClass должен быть создан заранее
priorityClassName: high-priority

# ===== MONITORING =====

metrics:
  enabled: true
  
  serviceMonitor:
    enabled: true
    interval: 15s
    scrapeTimeout: 10s
    
    # Алерты (если используется Prometheus Operator)
    rules:
      enabled: true

# ===== АННОТАЦИИ =====

podAnnotations:
  # Cluster Autoscaler — не эвиктить эти Pod'ы если возможно
  cluster-autoscaler.kubernetes.io/safe-to-evict: "false"

deploymentAnnotations:
  # Уведомления о деплоях
  notifications.argoproj.io/subscribe.slack: deployments-channel
```

### values-prod-secrets.yaml — секреты production

YAML

```
# environments/secrets/values-prod-secrets.yaml
#
# ⚠️  ЭТОТ ФАЙЛ НЕ ДОЛЖЕН БЫТЬ В GIT! ⚠️
#
# Варианты безопасного хранения:
# 1. Encrypted с помощью SOPS/age/GPG
# 2. В Vault и подтягивается при деплое
# 3. В Kubernetes Secrets и используется existingSecret
# 4. В CI/CD secrets и передаётся через --set
#
# Этот файл показан только для примера структуры.

# ===== DATABASE =====

# Credentials для внешней БД
externalDatabase:
  username: myapp_prod_user
  password: "SuperSecurePassword123!@#"

# ===== API KEYS =====

# Ключи внешних сервисов
secrets:
  stringData:
    # API ключи
    STRIPE_API_KEY: "sk_live_xxxxxxxxxxxx"
    SENDGRID_API_KEY: "SG.xxxxxxxxxxxx"
    
    # JWT секрет
    JWT_SECRET: "your-256-bit-secret-key-here"
    
    # Encryption key для данных
    ENCRYPTION_KEY: "32-byte-encryption-key-here!!"

# ===== SERVICE ACCOUNT =====

# AWS IAM роль для IRSA (EKS)
serviceAccount:
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/myapp-prod-role

# GCP Workload Identity
# serviceAccount:
#   annotations:
#     iam.gke.io/gcp-service-account: myapp@myproject.iam.gserviceaccount.com
```

---

## 5. Переопределение через --set

### Синтаксис --set

Флаг `--set` позволяет переопределить значения из values.yaml прямо в командной строке. Это полезно для:

- CI/CD пайплайнов (передача версии образа)
- Быстрых изменений без редактирования файлов
- Переопределения секретов из переменных окружения

Bash

```
# ===== БАЗОВЫЙ СИНТАКСИС =====

# Простое значение
# values.yaml: replicaCount: 1
# Результат: replicaCount: 5
helm install myapp ./mychart --set replicaCount=5

# Вложенное значение (через точку)
# values.yaml: image:
#                repository: nginx
#                tag: latest
# Результат: image.tag = "v2.0.0"
helm install myapp ./mychart --set image.tag=v2.0.0

# Глубоко вложенное значение
helm install myapp ./mychart --set ingress.hosts[0].paths[0].path=/api


# ===== НЕСКОЛЬКО ЗНАЧЕНИЙ =====

# Через запятую (одна опция --set)
helm install myapp ./mychart --set replicaCount=3,image.tag=v2.0.0

# Несколько опций --set (более читаемо)
helm install myapp ./mychart \
  --set replicaCount=3 \
  --set image.tag=v2.0.0 \
  --set service.type=LoadBalancer


# ===== ТИПЫ ДАННЫХ =====

# Строки (по умолчанию)
--set image.tag=v2.0.0

# Числа (автоматически определяются)
--set replicaCount=5
--set service.port=8080

# Boolean
--set ingress.enabled=true
--set autoscaling.enabled=false

# Null (удалить значение)
--set existingSecret=null


# ===== ПРИНУДИТЕЛЬНЫЕ ТИПЫ =====

# --set-string: всегда интерпретировать как строку
# Важно для версий типа "1.0" (иначе станет числом 1)
helm install myapp ./mychart --set-string image.tag=1.0

# Пример проблемы:
--set image.tag=1.20      # → число 1.2 (trailing zero пропадает!)
--set-string image.tag=1.20  # → строка "1.20" (правильно)

# Ещё примеры, когда нужен --set-string:
--set-string podAnnotations.version=007    # "007", не 7
--set-string config.port=0080              # "0080", не 80


# ===== ЗНАЧЕНИЯ ИЗ ФАЙЛОВ =====

# --set-file: загрузить содержимое файла как значение
# Полезно для сертификатов, конфигов, приватных ключей
helm install myapp ./mychart \
  --set-file tls.crt=./certificate.pem \
  --set-file tls.key=./private-key.pem

# Содержимое файла становится значением ключа
# values.yaml эквивалент:
# tls:
#   crt: |
#     -----BEGIN CERTIFICATE-----
#     ...
#     -----END CERTIFICATE-----


# ===== JSON ЗНАЧЕНИЯ =====

# --set-json: парсить значение как JSON
# Полезно для сложных структур
helm install myapp ./mychart \
  --set-json 'resources={"limits":{"cpu":"1000m","memory":"1Gi"}}'

# Массивы
helm install myapp ./mychart \
  --set-json 'env=[{"name":"KEY1","value":"val1"},{"name":"KEY2","value":"val2"}]'

# Вложенные объекты
helm install myapp ./mychart \
  --set-json 'ingress.hosts=[{"host":"example.com","paths":[{"path":"/","pathType":"Prefix"}]}]'


# ===== МАССИВЫ =====

# Установить элемент массива по индексу
--set ingress.hosts[0].host=example.com
--set ingress.hosts[0].paths[0].path=/

# Установить весь массив (через фигурные скобки)
--set 'ingress.hosts={host1.com,host2.com}'

# Это создаст:
# ingress:
#   hosts:
#     - host1.com
#     - host2.com

# Массив объектов (через --set-json проще)
--set env[0].name=KEY1 --set env[0].value=val1 --set env[1].name=KEY2 --set env[1].value=val2


# ===== ЭКРАНИРОВАНИЕ =====

# Запятая в значении (экранировать обратным слэшем)
--set config.data="value1\,value2"

# Точка в ИМЕНИ ключа (использовать кавычки и обратный слэш)
--set 'annotations.prometheus\.io/scrape=true'

# Специальные символы в значении
--set 'password=p@$$w0rd!'   # Одинарные кавычки в bash

# Переменные окружения
--set image.tag="${IMAGE_TAG}"
--set image.tag="$CI_COMMIT_SHA"


# ===== ПРИОРИТЕТ =====

# Порядок приоритета (от низшего к высшему):
# 1. values.yaml в чарте
# 2. Первый -f файл
# 3. Второй -f файл
# 4. ... последующие -f файлы ...
# 5. Первый --set
# 6. Второй --set
# 7. ... последующие --set ...

# Пример: --set переопределяет всё
helm install myapp ./mychart \
  -f values.yaml \           # replicaCount: 1
  -f values-prod.yaml \      # replicaCount: 5
  --set replicaCount=10      # Финальное значение: 10
```

### Практические примеры использования --set

Bash

```
# ===== CI/CD: ДИНАМИЧЕСКАЯ ВЕРСИЯ ОБРАЗА =====

# GitLab CI
helm upgrade --install myapp ./mychart \
  -f values-prod.yaml \
  --set image.tag="${CI_COMMIT_SHA:0:8}" \
  --set image.pullPolicy=Always

# GitHub Actions
helm upgrade --install myapp ./mychart \
  -f values-prod.yaml \
  --set image.tag="${GITHUB_SHA::8}"

# Jenkins
helm upgrade --install myapp ./mychart \
  -f values-prod.yaml \
  --set image.tag="${GIT_COMMIT}"


# ===== БЫСТРОЕ МАСШТАБИРОВАНИЕ =====

# Увеличить реплики для нагрузочного тестирования
helm upgrade myapp ./mychart \
  -f values-prod.yaml \
  --set replicaCount=20 \
  --set autoscaling.enabled=false

# Вернуть обратно
helm upgrade myapp ./mychart \
  -f values-prod.yaml \
  --set autoscaling.enabled=true


# ===== HOTFIX: БЫСТРЫЙ ОТКАТ ВЕРСИИ =====

# Откатить на предыдущую версию образа
helm upgrade myapp ./mychart \
  -f values-prod.yaml \
  --set image.tag=1.2.2  # Предыдущая стабильная версия


# ===== FEATURE FLAGS =====

# Включить/выключить функциональность
helm upgrade myapp ./mychart \
  -f values-prod.yaml \
  --set features.newUI=true \
  --set features.betaAPI=false


# ===== ОТЛАДКА =====

# Временно включить debug для production (осторожно!)
helm upgrade myapp ./mychart \
  -f values-prod.yaml \
  --set env[0].name=LOG_LEVEL \
  --set env[0].value=debug \
  --set env[1].name=ENABLE_DEBUG \
  --set env[1].value=true

# Вернуть обратно
helm upgrade myapp ./mychart -f values-prod.yaml


# ===== СЕКРЕТЫ ИЗ ПЕРЕМЕННЫХ ОКРУЖЕНИЯ =====

# Передача секретов без записи в файлы
export DB_PASSWORD="supersecret"
export API_KEY="sk_live_xxxxx"

helm upgrade --install myapp ./mychart \
  -f values-prod.yaml \
  --set secrets.stringData.DATABASE_PASSWORD="${DB_PASSWORD}" \
  --set secrets.stringData.API_KEY="${API_KEY}"


# ===== РАЗНЫЕ ОКРУЖЕНИЯ С ОДНОЙ КОМАНДОЙ =====

# Функция для деплоя
deploy_app() {
  local ENV=$1
  local TAG=$2
  
  helm upgrade --install "myapp-${ENV}" ./mychart \
    -f "values-${ENV}.yaml" \
    --namespace "${ENV}" \
    --create-namespace \
    --set image.tag="${TAG}" \
    --wait \
    --timeout 5m
}

# Использование
deploy_app dev latest
deploy_app staging 1.2.3-rc.1
deploy_app prod 1.2.3


# ===== СЛОЖНЫЕ ПЕРЕОПРЕДЕЛЕНИЯ =====

# Полностью переопределить resources
helm upgrade myapp ./mychart \
  --set-json 'resources={
    "limits": {"cpu": "2000m", "memory": "2Gi"},
    "requests": {"cpu": "1000m", "memory": "1Gi"}
  }'

# Добавить несколько env переменных
helm upgrade myapp ./mychart \
  --set-json 'extraEnv=[
    {"name": "FEATURE_X", "value": "enabled"},
    {"name": "CACHE_TTL", "value": "3600"},
    {"name": "MAX_CONNECTIONS", "value": "100"}
  ]'

# Сложная структура Ingress
helm upgrade myapp ./mychart \
  --set-json 'ingress.hosts=[
    {
      "host": "api.example.com",
      "paths": [
        {"path": "/v1", "pathType": "Prefix"},
        {"path": "/v2", "pathType": "Prefix"}
      ]
    },
    {
      "host": "admin.example.com",
      "paths": [
        {"path": "/", "pathType": "Prefix"}
      ]
    }
  ]'
```

---

## 6. Просмотр сгенерированных манифестов

### helm template — локальный рендер

`helm template` рендерит шаблоны локально, без обращения к кластеру. Это идеально для:

- Проверки результата перед деплоем
- CI/CD пайплайнов (линтинг, валидация)
- Отладки шаблонов
- Генерации манифестов для GitOps

Bash

```
# ===== БАЗОВОЕ ИСПОЛЬЗОВАНИЕ =====

# Рендерить все шаблоны
# Результат выводится в stdout
helm template myapp ./mychart

# С values-файлом
helm template myapp ./mychart -f values-prod.yaml

# С переопределениями
helm template myapp ./mychart \
  -f values-prod.yaml \
  --set image.tag=v2.0.0 \
  --set replicaCount=5


# ===== NAMESPACE =====

# Указать namespace (влияет на .Release.Namespace в шаблонах)
helm template myapp ./mychart \
  --namespace production \
  -f values-prod.yaml

# Без указания namespace используется "default"


# ===== РЕНДЕР КОНКРЕТНЫХ ШАБЛОНОВ =====

# Только один шаблон
helm template myapp ./mychart \
  --show-only templates/deployment.yaml

# Несколько шаблонов
helm template myapp ./mychart \
  --show-only templates/deployment.yaml \
  --show-only templates/service.yaml \
  --show-only templates/ingress.yaml


# ===== ОТЛАДКА =====

# Debug режим — показывает computed values и другую информацию
helm template myapp ./mychart \
  -f values-prod.yaml \
  --debug

# Debug показывает:
# - Имя релиза
# - Namespace
# - Chart информацию
# - Computed values (все значения после слияния)
# - Сгенерированные манифесты


# ===== СОХРАНЕНИЕ В ФАЙЛ =====

# Сохранить все манифесты в один файл
helm template myapp ./mychart -f values-prod.yaml > manifests.yaml

# Разделить по файлам (с помощью csplit или вручную)
helm template myapp ./mychart | csplit - '/^---$/' '{*}' --prefix=manifest-

# Или использовать --output-dir (записывает в директорию)
helm template myapp ./mychart \
  -f values-prod.yaml \
  --output-dir ./rendered-manifests

# Структура --output-dir:
# ./rendered-manifests/
# └── mychart/
#     └── templates/
#         ├── deployment.yaml
#         ├── service.yaml
#         └── ingress.yaml


# ===== ВАЛИДАЦИЯ СХЕМЫ =====

# Включить валидацию Kubernetes schema
# Требует доступ к кластеру для получения схемы
helm template myapp ./mychart --validate

# Это проверит:
# - Корректность apiVersion
# - Обязательные поля
# - Типы данных


# ===== API VERSIONS =====

# Указать какие API версии считать доступными
# Полезно для тестирования совместимости
helm template myapp ./mychart \
  --api-versions networking.k8s.io/v1 \
  --api-versions autoscaling/v2


# ===== KUBERNETES VERSION =====

# Эмулировать конкретную версию Kubernetes
# Влияет на .Capabilities.KubeVersion в шаблонах
helm template myapp ./mychart \
  --kube-version 1.25.0


# ===== INCLUDE CRDS =====

# Включить CRDs в вывод
helm template myapp ./mychart --include-crds

# Пропустить CRDs
helm template myapp ./mychart --skip-crds


# ===== RELEASE INFO =====

# Установить информацию о релизе
helm template myapp ./mychart \
  --release-name my-release \
  --namespace my-namespace \
  --is-upgrade  # Эмулировать upgrade вместо install


# ===== РАБОТА С ЗАВИСИМОСТЯМИ =====

# Сначала скачать зависимости
helm dependency update ./mychart

# Затем рендерить (включит зависимости)
helm template myapp ./mychart -f values-prod.yaml

# Пропустить тесты
helm template myapp ./mychart --skip-tests
```

### helm install/upgrade --dry-run

В отличие от `helm template`, `--dry-run` обращается к API-серверу для валидации:

Bash

```
# ===== БАЗОВОЕ ИСПОЛЬЗОВАНИЕ =====

# Dry-run для install
# Показывает что будет создано, но НЕ создаёт ресурсы
helm install myapp ./mychart \
  -f values-prod.yaml \
  --dry-run

# Dry-run для upgrade
# Показывает что изменится
helm upgrade myapp ./mychart \
  -f values-prod.yaml \
  --dry-run

# Комбинированная команда (install or upgrade)
helm upgrade --install myapp ./mychart \
  -f values-prod.yaml \
  --dry-run


# ===== РЕЖИМЫ DRY-RUN =====

# Server-side dry-run (по умолчанию в Helm 3.13+)
# Полная валидация на сервере, включая admission webhooks
helm upgrade --install myapp ./mychart \
  --dry-run=server

# Client-side dry-run
# Только рендеринг шаблонов, без обращения к API
# Быстрее, но менее точный
helm upgrade --install myapp ./mychart \
  --dry-run=client

# Старый синтаксис (deprecated, работает как client)
helm upgrade --install myapp ./mychart \
  --dry-run


# ===== С DEBUG =====

# Подробный вывод для отладки
helm upgrade --install myapp ./mychart \
  -f values-prod.yaml \
  --dry-run \
  --debug

# --debug показывает:
# - Полную информацию о релизе
# - Computed values
# - Hooks
# - Манифесты
# - Сообщения о валидации


# ===== NAMESPACE =====

# С указанием namespace
helm upgrade --install myapp ./mychart \
  --namespace production \
  --create-namespace \
  -f values-prod.yaml \
  --dry-run


# ===== ВАЛИДАЦИЯ =====

# Dry-run проверяет:
# 1. Синтаксис YAML
# 2. Корректность шаблонов
# 3. Валидность Kubernetes ресурсов (apiVersion, kind, etc.)
# 4. Admission webhooks (в server режиме)
# 5. RBAC (есть ли права на создание ресурсов)

# Пример ошибки:
# Error: INSTALLATION FAILED: 1 error occurred:
#   * Deployment.apps "myapp" is invalid: spec.replicas: Invalid value: -1
```

### helm get — информация о существующем релизе

Bash

```
# ===== MANIFEST =====

# Получить манифесты текущего релиза
# Показывает что РЕАЛЬНО применено к кластеру
helm get manifest myapp

# Для конкретной ревизии
helm get manifest myapp --revision 5

# Сохранить в файл
helm get manifest myapp > current-manifests.yaml


# ===== VALUES =====

# Получить values, которые были использованы при деплое
# Показывает ТОЛЬКО переопределённые значения (не defaults из чарта)
helm get values myapp

# Все значения (включая defaults)
helm get values myapp --all

# Для конкретной ревизии
helm get values myapp --revision 3

# В YAML формате (по умолчанию)
helm get values myapp -o yaml

# В JSON формате
helm get values myapp -o json


# ===== HOOKS =====

# Показать hooks релиза
helm get hooks myapp

# Hooks — это ресурсы, выполняемые в определённые моменты:
# - pre-install, post-install
# - pre-upgrade, post-upgrade
# - pre-delete, post-delete
# - pre-rollback, post-rollback


# ===== NOTES =====

# Показать NOTES.txt (сообщение после установки)
helm get notes myapp

# Полезно если потеряли вывод после helm install


# ===== ALL =====

# Вся информация о релизе
helm get all myapp

# Включает:
# - Hooks
# - Manifest
# - Notes
# - Values


# ===== METADATA =====

# Метаданные релиза (не манифесты)
helm get metadata myapp

# Показывает:
# - Name
# - Namespace
# - Revision
# - Status
# - Chart
# - AppVersion
# - Deployed timestamp
```

### helm diff — сравнение изменений (плагин)

`helm-diff` — один из самых полезных плагинов. Показывает разницу между тем, что задеплоено, и тем, что будет применено:

Bash

```
# ===== УСТАНОВКА ПЛАГИНА =====

helm plugin install https://github.com/databus23/helm-diff

# Проверить установку
helm plugin list


# ===== БАЗОВОЕ ИСПОЛЬЗОВАНИЕ =====

# Показать что изменится при upgrade
helm diff upgrade myapp ./mychart -f values-prod.yaml

# С переопределениями
helm diff upgrade myapp ./mychart \
  -f values-prod.yaml \
  --set image.tag=v2.0.0


# ===== ПАРАМЕТРЫ ВЫВОДА =====

# Цветной вывод (по умолчанию)
helm diff upgrade myapp ./mychart --color

# Без цветов
helm diff upgrade myapp ./mychart --no-color

# Показать контекст (строки вокруг изменений)
helm diff upgrade myapp ./mychart -C 5


# ===== ФИЛЬТРАЦИЯ =====

# Не показывать secrets (для безопасности)
helm diff upgrade myapp ./mychart --suppress-secrets

# Или через переменную окружения
export HELM_DIFF_IGNORE_UNKNOWN_FLAGS=true


# ===== THREE-WAY MERGE =====

# Трёхсторонний diff:
# - Что в последнем релизе Helm
# - Что сейчас в кластере (могло быть изменено вручную)
# - Что будет применено
helm diff upgrade myapp ./mychart --three-way-merge


# ===== СРАВНЕНИЕ РЕВИЗИЙ =====

# Сравнить две ревизии релиза
helm diff revision myapp 3 5

# Сравнить ревизию с текущим состоянием
helm diff revision myapp 3


# ===== ROLLBACK PREVIEW =====

# Предпросмотр отката
helm diff rollback myapp 2


# ===== CI/CD ИСПОЛЬЗОВАНИЕ =====

# Выйти с ошибкой если есть изменения (для проверки drift)
helm diff upgrade myapp ./mychart \
  --detailed-exitcode

# Exit codes:
# 0 — нет изменений
# 1 — ошибка
# 2 — есть изменения

# Использование в скрипте:
if helm diff upgrade myapp ./mychart --detailed-exitcode; then
  echo "No changes"
else
  if [ $? -eq 2 ]; then
    echo "Changes detected, deploying..."
    helm upgrade myapp ./mychart
  else
    echo "Error occurred"
    exit 1
  fi
fi


# ===== OUTPUT FORMAT =====

# Только добавленные строки
helm diff upgrade myapp ./mychart | grep '^+'

# Только удалённые строки
helm diff upgrade myapp ./mychart | grep '^-'

# JSON вывод (для парсинга)
helm diff upgrade myapp ./mychart -o json
```

### Сравнение helm template vs --dry-run vs helm diff

text

```
┌─────────────────────┬──────────────────┬──────────────────┬──────────────────┐
│                     │  helm template   │  --dry-run       │  helm diff       │
├─────────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Требует кластер?    │  Нет             │  Да              │  Да              │
├─────────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Валидация API?      │  Нет*            │  Да              │  Да              │
├─────────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Admission webhooks? │  Нет             │  Да (server)     │  Нет             │
├─────────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Показывает diff?    │  Нет             │  Нет             │  Да              │
├─────────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Скорость            │  Быстро          │  Средне          │  Средне          │
├─────────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ CI/CD без кластера  │  Да              │  Нет             │  Нет             │
├─────────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Проверка прав       │  Нет             │  Да              │  Нет             │
├─────────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Когда использовать  │  Линтинг,        │  Перед деплоем   │  Code review,    │
│                     │  генерация       │  для валидации   │  перед деплоем   │
└─────────────────────┴──────────────────┴──────────────────┴──────────────────┘

* helm template --validate требует кластер
```

---

## 7. Откат релиза (Rollback)

### Понимание ревизий

Каждый раз когда вы выполняете `helm install` или `helm upgrade`, создаётся новая **ревизия** релиза. Helm хранит историю ревизий, что позволяет откатываться на предыдущие версии.

text

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ИСТОРИЯ РЕВИЗИЙ РЕЛИЗА                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  helm install myapp ./mychart                                       │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  REVISION 1                                                  │   │
│  │  Status: superseded                                          │   │
│  │  Chart: myapp-1.0.0                                          │   │
│  │  Values: replicaCount=1, image.tag=v1.0.0                   │   │
│  │  Description: Install complete                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                             │
│       │  helm upgrade myapp ./mychart --set image.tag=v1.1.0       │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  REVISION 2                                                  │   │
│  │  Status: superseded                                          │   │
│  │  Chart: myapp-1.0.0                                          │   │
│  │  Values: replicaCount=1, image.tag=v1.1.0                   │   │
│  │  Description: Upgrade complete                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                             │
│       │  helm upgrade myapp ./mychart --set replicaCount=3         │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  REVISION 3                                                  │   │
│  │  Status: deployed                    ← ТЕКУЩАЯ               │   │
│  │  Chart: myapp-1.0.0                                          │   │
│  │  Values: replicaCount=3, image.tag=v1.1.0                   │   │
│  │  Description: Upgrade complete                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                             │
│       │  helm rollback myapp 1                                     │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  REVISION 4                                                  │   │
│  │  Status: deployed                    ← ТЕКУЩАЯ               │   │
│  │  Chart: myapp-1.0.0                                          │   │
│  │  Values: replicaCount=1, image.tag=v1.0.0  ← как в REV 1    │   │
│  │  Description: Rollback to 1                                  │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  Откат создаёт НОВУЮ ревизию с конфигурацией из указанной старой   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Команды для работы с историей

Bash

```
# ===== ПРОСМОТР ИСТОРИИ =====

# Показать историю релиза
helm history myapp

# Пример вывода:
# REVISION  UPDATED                   STATUS      CHART         APP VERSION  DESCRIPTION
# 1         Mon Jan 15 10:00:00 2024  superseded  myapp-1.0.0   1.0.0        Install complete
# 2         Mon Jan 15 14:30:00 2024  superseded  myapp-1.0.0   1.1.0        Upgrade complete
# 3         Mon Jan 15 16:45:00 2024  deployed    myapp-1.0.0   1.1.0        Upgrade complete

# Статусы:
# deployed    — текущая активная ревизия
# superseded  — предыдущие успешные ревизии
# failed      — неудачная попытка upgrade/rollback
# pending-install/pending-upgrade — в процессе
# uninstalling — в процессе удаления

# Ограничить количество записей
helm history myapp --max 10

# Выводить в другом формате
helm history myapp -o json
helm history myapp -o yaml
helm history myapp -o table


# ===== ДЕТАЛИ РЕВИЗИИ =====

# Values конкретной ревизии
helm get values myapp --revision 2

# Все values (включая defaults)
helm get values myapp --revision 2 --all

# Манифесты конкретной ревизии
helm get manifest myapp --revision 2

# Вся информация
helm get all myapp --revision 2


# ===== СРАВНЕНИЕ РЕВИЗИЙ =====

# С помощью helm diff (плагин)
helm diff revision myapp 2 3

# Вручную через diff
diff <(helm get manifest myapp --revision 2) <(helm get manifest myapp --revision 3)

# Сравнить values
diff <(helm get values myapp --revision 2) <(helm get values myapp --revision 3)
```

### Выполнение отката

Bash

```
# ===== БАЗОВЫЙ ОТКАТ =====

# Откатиться на предыдущую ревизию (текущая - 1)
helm rollback myapp

# Откатиться на конкретную ревизию
helm rollback myapp 2

# Если текущая ревизия 5, rollback на 2 создаст ревизию 6
# с конфигурацией из ревизии 2


# ===== DRY-RUN =====

# Предпросмотр отката без применения
helm rollback myapp 2 --dry-run

# С помощью helm diff (показывает что изменится)
helm diff rollback myapp 2


# ===== ОПЦИИ ОТКАТА =====

# Ждать завершения rollout
# По умолчанию Helm не ждёт — сразу возвращает управление
helm rollback myapp 2 --wait

# С таймаутом ожидания
helm rollback myapp 2 --wait --timeout 10m

# Без выполнения hooks (pre-rollback, post-rollback)
helm rollback myapp 2 --no-hooks

# Force — удалить и создать заново ресурсы
# Полезно если ресурс "застрял" в неконсистентном состоянии
helm rollback myapp 2 --force

# Recreate pods — удалить pod'ы после rollback
# Аналог kubectl rollout restart
helm rollback myapp 2 --recreate-pods

# Cleanup on fail — если откат не удался, вернуться обратно
helm rollback myapp 2 --cleanup-on-fail


# ===== УКАЗАНИЕ ОПИСАНИЯ =====

# Добавить описание к ревизии
helm rollback myapp 2 --description "Rollback due to critical bug in v1.1.0"

# Описание будет видно в helm history
```

### Hooks при откате

Hooks позволяют выполнить действия до или после отката:

YAML

```
# templates/pre-rollback-job.yaml
#
# Этот Job выполнится ПЕРЕД откатом
# Например: создание бэкапа, уведомление, проверки

apiVersion: batch/v1
kind: Job
metadata:
  # Имя должно быть уникальным для каждого запуска
  name: {{ include "mychart.fullname" . }}-pre-rollback-{{ now | date "20060102150405" }}
  
  annotations:
    # Указываем что это hook
    "helm.sh/hook": pre-rollback
    
    # Порядок выполнения (меньше = раньше)
    # Если несколько hooks, они выполняются по порядку
    "helm.sh/hook-weight": "-5"
    
    # Политика удаления hook ресурса:
    # - before-hook-creation: удалить предыдущий перед созданием
    # - hook-succeeded: удалить после успешного выполнения
    # - hook-failed: удалить после неудачного выполнения
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded

spec:
  # Таймаут Job
  activeDeadlineSeconds: 300
  
  # Количество попыток
  backoffLimit: 1
  
  template:
    spec:
      restartPolicy: Never
      
      containers:
        - name: backup
          image: mycompany/backup-tool:latest
          
          command:
            - /bin/sh
            - -c
            - |
              echo "Creating backup before rollback..."
              # Создаём бэкап базы данных
              pg_dump $DATABASE_URL > /backup/pre-rollback-$(date +%Y%m%d%H%M%S).sql
              
              echo "Notifying team about rollback..."
              # Отправляем уведомление
              curl -X POST $SLACK_WEBHOOK \
                -H "Content-Type: application/json" \
                -d '{"text": "Starting rollback of myapp..."}'
              
              echo "Pre-rollback tasks completed"
          
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: {{ include "mychart.fullname" . }}-secrets
                  key: database-url
            
            - name: SLACK_WEBHOOK
              valueFrom:
                secretKeyRef:
                  name: {{ include "mychart.fullname" . }}-secrets
                  key: slack-webhook
          
          volumeMounts:
            - name: backup-storage
              mountPath: /backup
      
      volumes:
        - name: backup-storage
          persistentVolumeClaim:
            claimName: backup-pvc
```

YAML

```
# templates/post-rollback-job.yaml
#
# Этот Job выполнится ПОСЛЕ отката
# Например: уведомление об успехе, очистка, healthcheck

apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "mychart.fullname" . }}-post-rollback-{{ now | date "20060102150405" }}
  
  annotations:
    "helm.sh/hook": post-rollback
    "helm.sh/hook-weight": "0"
    "helm.sh/hook-delete-policy": hook-succeeded

spec:
  activeDeadlineSeconds: 120
  backoffLimit: 1
  
  template:
    spec:
      restartPolicy: Never
      
      containers:
        - name: notify
          image: curlimages/curl:latest
          
          command:
            - /bin/sh
            - -c
            - |
              echo "Waiting for application to be ready..."
              # Ждём пока приложение станет доступно
              for i in $(seq 1 30); do
                if curl -sf http://{{ include "mychart.fullname" . }}:{{ .Values.service.port }}/health; then
                  echo "Application is healthy!"
                  break
                fi
                echo "Attempt $i: Application not ready, waiting..."
                sleep 5
              done
              
              echo "Sending success notification..."
              # Отправляем уведомление об успешном откате
              curl -X POST $SLACK_WEBHOOK \
                -H "Content-Type: application/json" \
                -d '{
                  "text": "✅ Rollback completed successfully!",
                  "attachments": [{
                    "color": "good",
                    "fields": [{
                      "title": "Release",
                      "value": "{{ .Release.Name }}",
                      "short": true
                    }, {
                      "title": "Namespace",
                      "value": "{{ .Release.Namespace }}",
                      "short": true
                    }]
                  }]
                }'
          
          env:
            - name: SLACK_WEBHOOK
              valueFrom:
                secretKeyRef:
                  name: {{ include "mychart.fullname" . }}-secrets
                  key: slack-webhook
```

### Управление историей ревизий

Bash

```
# ===== ОГРАНИЧЕНИЕ ИСТОРИИ =====

# При install/upgrade можно ограничить количество хранимых ревизий
# По умолчанию Helm хранит все ревизии

helm upgrade --install myapp ./mychart \
  --history-max 10

# Это сохранит только последние 10 ревизий
# Старые ревизии будут автоматически удалены


# ===== ГДЕ ХРАНЯТСЯ РЕВИЗИИ =====

# Helm 3 хранит метаданные релизов в Secrets
kubectl get secrets -l owner=helm

# Пример:
# sh.helm.release.v1.myapp.v1    — ревизия 1
# sh.helm.release.v1.myapp.v2    — ревизия 2
# sh.helm.release.v1.myapp.v3    — ревизия 3

# Посмотреть содержимое (base64 + gzip)
kubectl get secret sh.helm.release.v1.myapp.v1 -o jsonpath='{.data.release}' | base64 -d | gunzip


# ===== ОЧИСТКА ИСТОРИИ =====

# Нет встроенной команды для удаления истории
# Варианты:

# 1. Удалить и переустановить
helm uninstall myapp
helm install myapp ./mychart --history-max 5

# 2. Удалить старые secrets вручную (осторожно!)
kubectl delete secret sh.helm.release.v1.myapp.v1
kubectl delete secret sh.helm.release.v1.myapp.v2

# 3. Использовать плагин helm-mapkubeapis (для миграции устаревших API)
```

### Стратегии отката в production

Bash

```
# ===== СЦЕНАРИЙ 1: БЫСТРЫЙ ОТКАТ =====

# Обнаружена критическая проблема после деплоя
# Цель: откатиться максимально быстро

# 1. Смотрим историю
helm history myapp
# REVISION  STATUS      DESCRIPTION
# 5         superseded  Upgrade to v1.4.0
# 6         deployed    Upgrade to v1.5.0  ← проблемная версия

# 2. Откатываемся на предыдущую стабильную
helm rollback myapp 5 --wait --timeout 5m

# 3. Проверяем
kubectl get pods -l app.kubernetes.io/instance=myapp
helm status myapp


# ===== СЦЕНАРИЙ 2: ОТКАТ С ПРЕДВАРИТЕЛЬНОЙ ПРОВЕРКОЙ =====

# Есть время для проверки перед откатом

# 1. Предпросмотр изменений
helm diff rollback myapp 5

# 2. Dry-run
helm rollback myapp 5 --dry-run

# 3. Выполнить откат
helm rollback myapp 5 --wait
```
```
# ===== СЦЕНАРИЙ 3: ПРОБЛЕМА В ДАННЫХ =====

# Проблемная версия изменила данные в БД
# Откат кода недостаточен — нужен откат данных

# 1. Сначала останавливаем трафик (scale to 0)
kubectl scale deployment myapp --replicas=0

# 2. Восстанавливаем данные из бэкапа
# (зависит от вашей системы бэкапов)
pg_restore --clean -d myapp_db /backups/pre-deploy-backup.sql

# 3. Откатываем код
helm rollback myapp 5 --wait

# 4. Проверяем работоспособность
kubectl logs -l app.kubernetes.io/instance=myapp --tail=100

# 5. Возвращаем трафик (если всё ок, HPA восстановит реплики)
# Или вручную:
kubectl scale deployment myapp --replicas=5


# ===== СЦЕНАРИЙ 4: ЧАСТИЧНЫЙ ОТКАТ =====

# Нужно откатить только часть изменений
# Например: оставить новую версию образа, но вернуть старые ресурсы

# 1. Смотрим values предыдущей ревизии
helm get values myapp --revision 5

# 2. Смотрим values текущей ревизии
helm get values myapp

# 3. Делаем upgrade с комбинацией значений
helm upgrade myapp ./mychart \
  -f values-prod.yaml \
  --set image.tag=v1.5.0 \          # Оставляем новую версию
  --set resources.limits.cpu=500m \ # Возвращаем старые ресурсы
  --set resources.limits.memory=512Mi


# ===== СЦЕНАРИЙ 5: ОТКАТ С МИГРАЦИЕЙ БД =====

# Новая версия содержит миграцию БД
# При откате нужно откатить и миграцию

# Это решается через hooks в чарте:
# pre-rollback hook запускает обратную миграцию

# Или вручную:
# 1. Запускаем rollback миграции
kubectl run db-rollback --rm -it --image=myapp:v1.5.0 \
  --env="DATABASE_URL=$DB_URL" \
  -- ./migrate down

# 2. Откатываем приложение
helm rollback myapp 5 --wait


# ===== СЦЕНАРИЙ 6: CANARY ОТКАТ =====

# Проблема обнаружена на canary деплое
# Нужно быстро убрать canary

# Если canary — отдельный релиз:
helm uninstall myapp-canary

# Если canary реализован через labels/weight:
helm upgrade myapp ./mychart \
  -f values-prod.yaml \
  --set canary.enabled=false
```

### Автоматический откат

YAML

```
# ===== ИСПОЛЬЗОВАНИЕ --atomic =====

# При ошибке деплоя автоматически откатиться на предыдущую версию

# В командной строке:
# helm upgrade --install myapp ./mychart \
#   -f values-prod.yaml \
#   --atomic \
#   --timeout 10m

# Что делает --atomic:
# 1. Устанавливает --wait автоматически
# 2. При ошибке (таймаут, failed pods) выполняет helm rollback
# 3. Помечает релиз как failed

# Пример использования в CI/CD (GitLab CI):
# deploy:
#   script:
#     - helm upgrade --install myapp ./mychart
#         -f values-prod.yaml
#         --set image.tag=$CI_COMMIT_SHA
#         --atomic
#         --timeout 10m


# ===== KUBERNETES ROLLBACK (НЕ HELM) =====

# Kubernetes сам может откатить Deployment при проблемах
# Это настраивается в Deployment spec

# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
spec:
  # Стратегия обновления
  strategy:
    type: RollingUpdate
    rollingUpdate:
      # Максимум недоступных Pod'ов
      maxUnavailable: 0
      # Максимум дополнительных Pod'ов
      maxSurge: 1
  
  # Минимальное время готовности Pod'а
  # Pod считается Ready только после этого времени
  # Помогает обнаружить проблемы на старте
  minReadySeconds: 30
  
  # Время ожидания прогресса
  # Если за это время нет прогресса — деплой считается failed
  progressDeadlineSeconds: 600
  
  template:
    spec:
      containers:
        - name: app
          # Обязательно настройте probes!
          # Без них Kubernetes не узнает о проблемах
          
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
            failureThreshold: 3
          
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
            failureThreshold: 3
```

### Мониторинг и алерты для откатов

YAML

```
# ===== PROMETHEUS ALERTS =====

# Пример PrometheusRule для алертов о деплоях и откатах

apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: helm-deployment-alerts
spec:
  groups:
    - name: helm-deployments
      rules:
        # Алерт: деплой занимает слишком много времени
        - alert: HelmDeploymentStuck
          expr: |
            kube_deployment_status_condition{condition="Progressing",status="false"} == 1
          for: 15m
          labels:
            severity: warning
          annotations:
            summary: "Deployment {{ $labels.deployment }} is stuck"
            description: "Deployment hasn't made progress for 15 minutes"
        
        # Алерт: после деплоя увеличился error rate
        - alert: HighErrorRateAfterDeploy
          expr: |
            (
              sum(rate(http_requests_total{status=~"5.."}[5m])) 
              / 
              sum(rate(http_requests_total[5m]))
            ) > 0.05
            and
            changes(kube_deployment_status_observed_generation[30m]) > 0
          for: 5m
          labels:
            severity: critical
          annotations:
            summary: "High error rate after deployment"
            description: "Error rate > 5% within 30 min after deployment. Consider rollback."
        
        # Алерт: Pod'ы постоянно рестартуют после деплоя
        - alert: PodCrashLoopAfterDeploy
          expr: |
            increase(kube_pod_container_status_restarts_total[30m]) > 3
            and
            changes(kube_deployment_status_observed_generation[30m]) > 0
          for: 10m
          labels:
            severity: critical
          annotations:
            summary: "Pods crash-looping after deployment"
            description: "Pods are restarting frequently. Immediate rollback recommended."


# ===== GRAFANA DASHBOARD ANNOTATIONS =====

# Показать деплои на графиках Grafana
# Используйте аннотации от Helm hooks или CI/CD

# templates/post-upgrade-annotation.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "mychart.fullname" . }}-grafana-annotation
  annotations:
    "helm.sh/hook": post-upgrade,post-install
    "helm.sh/hook-weight": "10"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: annotate
          image: curlimages/curl:latest
          command:
            - /bin/sh
            - -c
            - |
              curl -X POST "$GRAFANA_URL/api/annotations" \
                -H "Authorization: Bearer $GRAFANA_API_KEY" \
                -H "Content-Type: application/json" \
                -d '{
                  "dashboardId": 1,
                  "time": '$(date +%s000)',
                  "tags": ["deployment", "{{ .Release.Name }}", "{{ .Values.image.tag }}"],
                  "text": "Deployed {{ .Release.Name }} version {{ .Values.image.tag }}"
                }'
          env:
            - name: GRAFANA_URL
              value: "http://grafana.monitoring.svc.cluster.local:3000"
            - name: GRAFANA_API_KEY
              valueFrom:
                secretKeyRef:
                  name: grafana-credentials
                  key: api-key
```

---

## 8. Полная шпаргалка команд Helm

### Создание и управление чартами

Bash

```
# ===== СОЗДАНИЕ ЧАРТА =====

# Создать новый чарт с дефолтной структурой
# Создаёт директорию mychart/ со всеми необходимыми файлами
helm create mychart

# Структура после создания:
# mychart/
# ├── Chart.yaml           # Метаданные чарта
# ├── values.yaml          # Значения по умолчанию
# ├── charts/              # Зависимости
# ├── templates/           # Шаблоны
# │   ├── NOTES.txt        # Сообщение после установки
# │   ├── _helpers.tpl     # Вспомогательные функции
# │   ├── deployment.yaml
# │   ├── hpa.yaml
# │   ├── ingress.yaml
# │   ├── service.yaml
# │   ├── serviceaccount.yaml
# │   └── tests/
# │       └── test-connection.yaml
# └── .helmignore          # Файлы для игнорирования


# ===== ПРОВЕРКА ЧАРТА =====

# Проверить синтаксис и структуру чарта
# Выявляет ошибки в шаблонах, неправильные значения
helm lint ./mychart

# Строгая проверка (предупреждения становятся ошибками)
helm lint ./mychart --strict

# Проверка с конкретными values
helm lint ./mychart -f values-prod.yaml

# Пример вывода с ошибкой:
# ==> Linting ./mychart
# [ERROR] templates/deployment.yaml: unable to parse YAML
# [WARNING] templates/ingress.yaml: object name does not conform to Kubernetes naming requirements


# ===== УПАКОВКА ЧАРТА =====

# Упаковать чарт в .tgz архив
# Имя файла: {name}-{version}.tgz
helm package ./mychart

# Результат: mychart-1.0.0.tgz

# С указанием версии (переопределяет version в Chart.yaml)
helm package ./mychart --version 1.2.3

# С указанием app-version
helm package ./mychart --app-version 2.0.0

# Указать директорию для вывода
helm package ./mychart --destination ./releases/

# Подписать пакет (для верификации)
helm package ./mychart --sign --key "mykey" --keyring ~/.gnupg/pubring.gpg


# ===== ЗАВИСИМОСТИ =====

# Скачать зависимости из Chart.yaml
# Создаёт Chart.lock и загружает чарты в charts/
helm dependency update ./mychart

# Собрать зависимости из Chart.lock
# Использует версии из lock-файла (более детерминированно)
helm dependency build ./mychart

# Показать список зависимостей
helm dependency list ./mychart

# Пример вывода:
# NAME        VERSION   REPOSITORY                              STATUS
# postgresql  12.1.2    https://charts.bitnami.com/bitnami     ok
# redis       17.3.7    https://charts.bitnami.com/bitnami     missing

# Статусы:
# ok       — зависимость скачана
# missing  — зависимость не скачана
# unpacked — зависимость распакована (не .tgz)
```

### Установка и обновление

Bash

```
# ===== УСТАНОВКА =====

# Базовая установка
# myapp — имя релиза (уникальное в namespace)
# ./mychart — путь к чарту или имя чарта в репозитории
helm install myapp ./mychart

# С values-файлом
helm install myapp ./mychart -f values.yaml

# С несколькими values-файлами (последний имеет приоритет)
helm install myapp ./mychart \
  -f values.yaml \
  -f values-prod.yaml

# С переопределением значений
helm install myapp ./mychart \
  --set replicaCount=3 \
  --set image.tag=v2.0.0

# В конкретный namespace
helm install myapp ./mychart \
  --namespace production

# Создать namespace если не существует
helm install myapp ./mychart \
  --namespace production \
  --create-namespace

# Ждать готовности всех ресурсов
helm install myapp ./mychart --wait

# С таймаутом ожидания
helm install myapp ./mychart --wait --timeout 10m

# Atomic: откат при ошибке
helm install myapp ./mychart --atomic --timeout 10m

# Генерировать имя релиза автоматически
helm install ./mychart --generate-name
# Результат: mychart-1705312345

# Установить из репозитория
helm install myapp bitnami/postgresql

# Установить конкретную версию чарта
helm install myapp bitnami/postgresql --version 12.1.2

# Dry-run: показать что будет создано без применения
helm install myapp ./mychart --dry-run

# Подробный dry-run
helm install myapp ./mychart --dry-run --debug

# Заменить релиз если существует (удалить и создать заново)
# ОСТОРОЖНО: удаляет все ресурсы!
helm install myapp ./mychart --replace


# ===== ОБНОВЛЕНИЕ =====

# Базовое обновление
helm upgrade myapp ./mychart

# Установить или обновить (если релиз не существует — создать)
# Самая частая команда в CI/CD
helm upgrade --install myapp ./mychart

# С values-файлом
helm upgrade myapp ./mychart -f values-prod.yaml

# С переопределениями
helm upgrade myapp ./mychart \
  --set image.tag=v2.0.0

# Переиспользовать values из предыдущего релиза
# Полезно когда хотите изменить только один параметр
helm upgrade myapp ./mychart --reuse-values --set image.tag=v2.0.0

# Сбросить values к defaults из чарта
helm upgrade myapp ./mychart --reset-values

# Ждать готовности
helm upgrade myapp ./mychart --wait --timeout 10m

# Atomic: откат при ошибке
helm upgrade myapp ./mychart --atomic --timeout 10m

# Force: пересоздать ресурсы даже если нет изменений
helm upgrade myapp ./mychart --force

# Очистить при ошибке (удалить созданные ресурсы)
helm upgrade myapp ./mychart --cleanup-on-fail

# Ограничить историю ревизий
helm upgrade myapp ./mychart --history-max 10

# Пропустить CRDs
helm upgrade myapp ./mychart --skip-crds

# Описание ревизии
helm upgrade myapp ./mychart --description "Upgrade to v2.0.0 with new features"


# ===== КОМБИНИРОВАННЫЙ ПРИМЕР ДЛЯ PRODUCTION =====

helm upgrade --install myapp ./mychart \
  --namespace production \
  --create-namespace \
  -f environments/values-common.yaml \
  -f environments/values-prod.yaml \
  --set image.tag="${CI_COMMIT_SHA:0:8}" \
  --wait \
  --timeout 10m \
  --atomic \
  --history-max 10 \
  --description "Deploy from CI: ${CI_PIPELINE_URL}"
```

### Информация и статус

Bash

```
# ===== СПИСОК РЕЛИЗОВ =====

# Показать все релизы в текущем namespace
helm list

# Сокращённая форма
helm ls

# Во всех namespaces
helm list --all-namespaces
helm list -A

# Показать все релизы (включая failed, uninstalling, etc.)
helm list --all

# Фильтр по статусу
helm list --deployed      # Только успешно установленные
helm list --failed        # Только failed
helm list --pending       # В процессе установки
helm list --uninstalling  # В процессе удаления
helm list --superseded    # Заменённые (после upgrade)
helm list --uninstalled   # Удалённые (если --keep-history)

# Фильтр по имени (регулярное выражение)
helm list --filter 'myapp.*'

# Сортировка
helm list --date          # По дате (новые первые)
helm list --reverse       # В обратном порядке

# Формат вывода
helm list -o json
helm list -o yaml
helm list -o table        # По умолчанию

# Показать только имена (для скриптов)
helm list -q


# ===== СТАТУС РЕЛИЗА =====

# Подробный статус релиза
helm status myapp

# Показывает:
# - Имя, namespace, статус
# - Ревизию
# - Время последнего деплоя
# - Список ресурсов
# - NOTES.txt

# Конкретная ревизия
helm status myapp --revision 5

# Формат вывода
helm status myapp -o json
helm status myapp -o yaml


# ===== ИСТОРИЯ РЕЛИЗА =====

# Показать историю ревизий
helm history myapp

# Пример вывода:
# REVISION  UPDATED                   STATUS      CHART         APP VERSION  DESCRIPTION
# 1         Mon Jan 15 10:00:00 2024  superseded  myapp-1.0.0   1.0.0        Install complete
# 2         Mon Jan 15 14:30:00 2024  superseded  myapp-1.0.0   1.1.0        Upgrade complete
# 3         Mon Jan 15 16:45:00 2024  deployed    myapp-1.0.0   1.1.0        Upgrade complete

# Ограничить количество записей
helm history myapp --max 10

# Формат вывода
helm history myapp -o json


# ===== ПОЛУЧЕНИЕ ДЕТАЛЕЙ РЕЛИЗА =====

# Все манифесты текущего релиза
helm get manifest myapp

# Манифесты конкретной ревизии
helm get manifest myapp --revision 2

# Values, использованные при деплое
helm get values myapp

# Все values (включая defaults из чарта)
helm get values myapp --all

# Values конкретной ревизии
helm get values myapp --revision 2

# Hooks
helm get hooks myapp

# NOTES.txt
helm get notes myapp

# Метаданные
helm get metadata myapp

# Всё вместе
helm get all myapp


# ===== РЕНДЕРИНГ БЕЗ УСТАНОВКИ =====

# Показать сгенерированные манифесты
helm template myapp ./mychart

# С values
helm template myapp ./mychart -f values-prod.yaml

# Конкретные шаблоны
helm template myapp ./mychart --show-only templates/deployment.yaml

# С debug информацией
helm template myapp ./mychart --debug

# Сохранить в файлы
helm template myapp ./mychart --output-dir ./manifests/

# Указать namespace (влияет на .Release.Namespace)
helm template myapp ./mychart --namespace production

# Валидация (требует кластер)
helm template myapp ./mychart --validate

# Эмулировать версию Kubernetes
helm template myapp ./mychart --kube-version 1.25.0

# Указать доступные API
helm template myapp ./mychart --api-versions networking.k8s.io/v1
```

### Удаление и откат

Bash

```
# ===== УДАЛЕНИЕ =====

# Удалить релиз
helm uninstall myapp

# Сокращённые формы
helm delete myapp
helm del myapp

# Из конкретного namespace
helm uninstall myapp --namespace production

# Сохранить историю релиза (для возможности rollback)
helm uninstall myapp --keep-history

# Dry-run
helm uninstall myapp --dry-run

# Не ждать завершения удаления
helm uninstall myapp --no-hooks

# Без выполнения hooks (pre-delete, post-delete)
helm uninstall myapp --no-hooks

# Таймаут ожидания
helm uninstall myapp --wait --timeout 5m

# Удалить с описанием
helm uninstall myapp --description "Removed due to deprecation"


# ===== ОТКАТ =====

# Откатиться на предыдущую ревизию
helm rollback myapp

# На конкретную ревизию
helm rollback myapp 2

# Dry-run
helm rollback myapp 2 --dry-run

# Ждать готовности
helm rollback myapp 2 --wait

# С таймаутом
helm rollback myapp 2 --wait --timeout 10m

# Force: пересоздать ресурсы
helm rollback myapp 2 --force

# Recreate pods
helm rollback myapp 2 --recreate-pods

# Без hooks
helm rollback myapp 2 --no-hooks

# Очистить при ошибке
helm rollback myapp 2 --cleanup-on-fail

# С описанием
helm rollback myapp 2 --description "Rollback due to critical bug"
```

### Репозитории

Bash

```
# ===== ДОБАВЛЕНИЕ РЕПОЗИТОРИЕВ =====

# Добавить репозиторий
helm repo add bitnami https://charts.bitnami.com/bitnami

# С аутентификацией
helm repo add myrepo https://charts.example.com \
  --username admin \
  --password secret

# С TLS сертификатами
helm repo add myrepo https://charts.example.com \
  --ca-file ./ca.pem \
  --cert-file ./cert.pem \
  --key-file ./key.pem

# Принудительное обновление (если уже существует)
helm repo add bitnami https://charts.bitnami.com/bitnami --force-update

# OCI registry (Helm 3.8+)
helm registry login registry.example.com \
  --username admin \
  --password secret


# ===== УПРАВЛЕНИЕ РЕПОЗИТОРИЯМИ =====

# Список репозиториев
helm repo list

# Обновить индекс всех репозиториев
helm repo update

# Обновить конкретный репозиторий
helm repo update bitnami

# Удалить репозиторий
helm repo remove bitnami

# Сгенерировать индекс для своего репозитория
helm repo index ./my-charts/ --url https://charts.example.com


# ===== ПОИСК ЧАРТОВ =====

# Поиск в локальных репозиториях
helm search repo postgresql

# Показать все версии
helm search repo postgresql --versions

# Показать development версии
helm search repo postgresql --devel

# Поиск в Artifact Hub (публичный hub)
helm search hub wordpress

# Фильтр по версии
helm search repo postgresql --version ">=12.0.0"


# ===== ИНФОРМАЦИЯ О ЧАРТЕ =====

# Показать информацию о чарте из репозитория
helm show chart bitnami/postgresql

# Показать README
helm show readme bitnami/postgresql

# Показать values.yaml
helm show values bitnami/postgresql

# Показать всё
helm show all bitnami/postgresql

# Сохранить values в файл
helm show values bitnami/postgresql > postgresql-values.yaml

# Скачать чарт без установки
helm pull bitnami/postgresql

# Скачать и распаковать
helm pull bitnami/postgresql --untar

# Конкретная версия
helm pull bitnami/postgresql --version 12.1.2
```

### Плагины

Bash

```
# ===== УПРАВЛЕНИЕ ПЛАГИНАМИ =====

# Список установленных плагинов
helm plugin list

# Установить плагин
helm plugin install https://github.com/databus23/helm-diff

# Установить конкретную версию
helm plugin install https://github.com/databus23/helm-diff --version v3.8.1

# Обновить плагин
helm plugin update diff

# Удалить плагин
helm plugin uninstall diff


# ===== ПОПУЛЯРНЫЕ ПЛАГИНЫ =====

# helm-diff: показывает разницу между тем что есть и что будет
helm plugin install https://github.com/databus23/helm-diff
helm diff upgrade myapp ./mychart -f values.yaml

# helm-secrets: работа с зашифрованными values (SOPS)
helm plugin install https://github.com/jkroepke/helm-secrets
helm secrets upgrade myapp ./mychart -f secrets.yaml

# helm-s3: хранение чартов в AWS S3
helm plugin install https://github.com/hypnoglow/helm-s3
helm repo add my-charts s3://my-bucket/charts

# helm-git: установка чартов напрямую из git
helm plugin install https://github.com/aslafy-z/helm-git
helm install myapp "git+https://github.com/org/repo@charts/myapp?ref=v1.0.0"

# helm-unittest: unit тесты для чартов
helm plugin install https://github.com/helm-unittest/helm-unittest
helm unittest ./mychart

# helm-push: push чартов в ChartMuseum/OCI
helm plugin install https://github.com/chartmuseum/helm-push
helm cm-push ./mychart my-chartmuseum

# helm-mapkubeapis: обновление устаревших API
helm plugin install https://github.com/helm/helm-mapkubeapis
helm mapkubeapis myapp
```

### Тестирование

Bash

```
# ===== ЗАПУСК ТЕСТОВ =====

# Запустить тесты чарта
# Тесты определены в templates/tests/
helm test myapp

# С логами
helm test myapp --logs

# Удалить тестовые ресурсы после запуска
helm test myapp --cleanup

# Таймаут
helm test myapp --timeout 10m

# Фильтр по имени теста
helm test myapp --filter name=test-connection


# ===== UNIT ТЕСТЫ (плагин helm-unittest) =====

# Установка
helm plugin install https://github.com/helm-unittest/helm-unittest

# Запуск
helm unittest ./mychart

# С цветным выводом
helm unittest ./mychart --color

# Обновить snapshot'ы
helm unittest ./mychart --update-snapshot

# Конкретный файл теста
helm unittest ./mychart -f 'tests/*_test.yaml'
```

### Переменные окружения

Bash

```
# ===== ВАЖНЫЕ ПЕРЕМЕННЫЕ ОКРУЖЕНИЯ =====

# Домашняя директория Helm (по умолчанию ~/.config/helm)
export HELM_CONFIG_HOME=/custom/path

# Директория кэша (по умолчанию ~/.cache/helm)
export HELM_CACHE_HOME=/custom/cache

# Директория данных (по умолчанию ~/.local/share/helm)
export HELM_DATA_HOME=/custom/data

# Путь к kubeconfig
export KUBECONFIG=/path/to/kubeconfig

# Namespace по умолчанию
export HELM_NAMESPACE=production

# Драйвер хранения (secret, configmap, sql)
export HELM_DRIVER=secret

# Максимум истории ревизий
export HELM_MAX_HISTORY=10

# Включить debug
export HELM_DEBUG=true

# Пропустить TLS проверку для репозиториев
export HELM_INSECURE_SKIP_TLS_VERIFY=true

# Таймаут по умолчанию
export HELM_TIMEOUT=5m

# OCI experimental features
export HELM_EXPERIMENTAL_OCI=1
```

---

## 9. Best Practices и рекомендации

### Организация чартов

YAML

```
# ===== СТРУКТУРА ПРОЕКТА =====

# Рекомендуемая структура для проекта с несколькими окружениями:

my-project/
│
├── .gitlab-ci.yml                 # CI/CD конфигурация
├── Makefile                       # Удобные команды
├── README.md                      # Документация
│
├── charts/                        # Helm чарты
│   │
│   ├── myapp/                     # Основное приложение
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   ├── templates/
│   │   └── charts/
│   │
│   └── common/                    # Общая библиотека (type: library)
│       ├── Chart.yaml
│       └── templates/
│           └── _helpers.tpl
│
├── environments/                  # Конфигурации окружений
│   ├── base/                      # Базовые values
│   │   └── values.yaml
│   │
│   ├── dev/
│   │   ├── values.yaml
│   │   └── secrets.yaml.enc      # Зашифрованные секреты
│   │
│   ├── staging/
│   │   ├── values.yaml
│   │   └── secrets.yaml.enc
│   │
│   └── production/
│       ├── values.yaml
│       └── secrets.yaml.enc
│
└── scripts/
    ├── deploy.sh
    └── rollback.sh


# ===== MAKEFILE ДЛЯ УДОБСТВА =====

# Makefile

.PHONY: lint template deploy-dev deploy-staging deploy-prod rollback

# Переменные
CHART_PATH := ./charts/myapp
RELEASE_NAME := myapp

# Линтинг
lint:
	helm lint $(CHART_PATH)
	helm lint $(CHART_PATH) -f environments/production/values.yaml

# Рендеринг шаблонов
template:
	helm template $(RELEASE_NAME) $(CHART_PATH) \
		-f environments/base/values.yaml \
		-f environments/$(ENV)/values.yaml

# Деплой в dev
deploy-dev:
	helm upgrade --install $(RELEASE_NAME) $(CHART_PATH) \
		--namespace dev \
		--create-namespace \
		-f environments/base/values.yaml \
		-f environments/dev/values.yaml \
		--wait

# Деплой в production
deploy-prod:
	helm upgrade --install $(RELEASE_NAME) $(CHART_PATH) \
		--namespace production \
		--create-namespace \
		-f environments/base/values.yaml \
		-f environments/production/values.yaml \
		--atomic \
		--timeout 10m \
		--history-max 10

# Откат
rollback:
	helm rollback $(RELEASE_NAME) $(REVISION) \
		--namespace $(NAMESPACE) \
		--wait
```
### Версионирование

YAML

```
# ===== SEMANTIC VERSIONING ДЛЯ ЧАРТОВ =====

# Chart.yaml
apiVersion: v2
name: myapp

# Версия чарта: MAJOR.MINOR.PATCH
# MAJOR: несовместимые изменения в values/templates
# MINOR: новая функциональность (обратно совместимая)
# PATCH: исправления багов
version: 2.3.1

# Версия приложения (информационное поле)
appVersion: "1.5.0"


# ===== КОГДА УВЕЛИЧИВАТЬ ВЕРСИЮ =====

# PATCH (2.3.0 → 2.3.1):
# - Исправление багов в шаблонах
# - Исправление опечаток
# - Обновление документации
# - Обновление зависимостей (patch версии)

# MINOR (2.3.0 → 2.4.0):
# - Добавление нового опционального параметра в values
# - Добавление нового шаблона (который не включён по умолчанию)
# - Новая фича, которая не ломает существующие деплои
# - Обновление зависимостей (minor версии)

# MAJOR (2.3.0 → 3.0.0):
# - Изменение структуры values (переименование, удаление ключей)
# - Изменение поведения по умолчанию
# - Удаление deprecated параметров
# - Требуется ручная миграция при upgrade
# - Обновление зависимостей (major версии)


# ===== CHANGELOG =====

# CHANGELOG.md
# # Changelog

## [2.3.1] - 2024-01-15
### Fixed
- Fixed ingress template when TLS is disabled (#123)
- Fixed resource limits not applied correctly

## [2.3.0] - 2024-01-10
### Added
- Support for topology spread constraints
- New parameter `podDisruptionBudget.maxUnavailable`

### Changed
- Updated PostgreSQL dependency to 12.1.2

## [2.2.0] - 2024-01-05
### Added
- Support for external secrets

### Deprecated
- Parameter `secrets.stringData` is deprecated, use external secrets instead
```

### Безопасность

YAML

```
# ===== СЕКРЕТЫ =====

# НИКОГДА не храните секреты в values.yaml в git!
# Используйте один из подходов:

# 1. External Secrets Operator
# Секреты хранятся в AWS Secrets Manager, Vault, etc.
# ESO синхронизирует их в Kubernetes Secrets

# values.yaml
externalSecrets:
  enabled: true
  secretStore: aws-secrets-manager
  data:
    - secretKey: database-password
      remoteRef:
        key: /myapp/production/database
        property: password


# 2. Sealed Secrets (Bitnami)
# Секреты шифруются публичным ключом кластера

# Создание sealed secret:
# kubeseal --format=yaml < secret.yaml > sealed-secret.yaml

# values.yaml
sealedSecrets:
  enabled: true
  encryptedData:
    DATABASE_PASSWORD: AgBy3i4OJSWK+...


# 3. SOPS + helm-secrets
# Файлы шифруются с помощью age, GPG или cloud KMS

# environments/production/secrets.yaml (зашифрован SOPS)
database:
  password: ENC[AES256_GCM,data:...,tag:...,type:str]

# Деплой:
# helm secrets upgrade myapp ./mychart -f secrets.yaml


# 4. Передача через CI/CD переменные
# Секреты хранятся в CI/CD системе и передаются через --set

# .gitlab-ci.yml
deploy:
  script:
    - helm upgrade --install myapp ./mychart
        --set database.password="${DATABASE_PASSWORD}"


# ===== SECURITY CONTEXT =====

# Всегда задавайте security context в production

# values.yaml
podSecurityContext:
  runAsNonRoot: true           # Запускать от непривилегированного пользователя
  runAsUser: 1000              # UID пользователя
  runAsGroup: 1000             # GID группы
  fsGroup: 1000                # Группа для volumes
  seccompProfile:
    type: RuntimeDefault       # Использовать seccomp профиль

securityContext:
  allowPrivilegeEscalation: false  # Запретить повышение привилегий
  readOnlyRootFilesystem: true     # Read-only корневая FS
  capabilities:
    drop:
      - ALL                        # Убрать все capabilities
    # add:
    #   - NET_BIND_SERVICE         # Добавить только нужные


# ===== NETWORK POLICIES =====

# Ограничьте сетевой доступ

networkPolicy:
  enabled: true
  policyTypes:
    - Ingress
    - Egress
  
  ingress:
    # Только от Ingress Controller
    - from:
        - namespaceSelector:
            matchLabels:
              name: ingress-nginx
      ports:
        - protocol: TCP
          port: 8080
  
  egress:
    # DNS
    - to:
        - namespaceSelector: {}
      ports:
        - protocol: UDP
          port: 53
    # База данных
    - to:
        - ipBlock:
            cidr: 10.0.0.0/8
      ports:
        - protocol: TCP
          port: 5432


# ===== RBAC =====

# Минимальные права для ServiceAccount

rbac:
  create: true
  rules:
    # Только чтение ConfigMaps
    - apiGroups: [""]
      resources: ["configmaps"]
      verbs: ["get", "list", "watch"]
    
    # Не давайте широкие права типа:
    # - apiGroups: ["*"]
    #   resources: ["*"]
    #   verbs: ["*"]
```

### Производительность и надёжность

YAML

```
# ===== РЕСУРСЫ =====

# Всегда задавайте requests и limits в production

resources:
  # Лимиты — максимум, что может использовать контейнер
  limits:
    cpu: 1000m      # 1 CPU core
    memory: 1Gi     # 1 гигабайт
  
  # Запросы — гарантированные ресурсы
  # Должны быть <= limits
  requests:
    cpu: 500m       # 0.5 CPU core
    memory: 512Mi   # 512 мегабайт

# Как определить правильные значения:
# 1. Начните с низких значений
# 2. Мониторьте реальное потребление (Prometheus/Grafana)
# 3. Увеличивайте limits с запасом 20-30%
# 4. Установите requests на уровне среднего потребления


# ===== PROBES =====

# Обязательно настройте probes для production

# Liveness — перезапускает контейнер если мёртв
livenessProbe:
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 30    # Дать время на старт
  periodSeconds: 10          # Проверять каждые 10 сек
  timeoutSeconds: 5          # Таймаут проверки
  failureThreshold: 3        # 3 неудачи = рестарт

# Readiness — убирает из балансировки если не готов
readinessProbe:
  httpGet:
    path: /ready
    port: http
  initialDelaySeconds: 10    # Меньше чем liveness
  periodSeconds: 5           # Проверять чаще
  failureThreshold: 3

# Startup — для медленно стартующих приложений
startupProbe:
  httpGet:
    path: /startup
    port: http
  failureThreshold: 30       # 30 * 10 = 5 минут на старт
  periodSeconds: 10


# ===== HIGH AVAILABILITY =====

# Минимум 2-3 реплики для production
replicaCount: 3

# Pod Anti-Affinity — распределение по нодам
affinity:
  podAntiAffinity:
    # Жёсткое требование — разные ноды
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app.kubernetes.io/name
              operator: In
              values:
                - myapp
        topologyKey: kubernetes.io/hostname
    
    # Желательно — разные зоны доступности
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
              - key: app.kubernetes.io/name
                operator: In
                values:
                  - myapp
          topologyKey: topology.kubernetes.io/zone

# Topology Spread — равномерное распределение
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app.kubernetes.io/name: myapp

# Pod Disruption Budget — минимум доступных pod'ов
podDisruptionBudget:
  enabled: true
  minAvailable: 2    # Минимум 2 pod'а всегда доступны
  # или
  # maxUnavailable: 1  # Максимум 1 недоступный


# ===== GRACEFUL SHUTDOWN =====

# Дайте приложению время завершить запросы

# Время ожидания SIGTERM → SIGKILL
terminationGracePeriodSeconds: 60

# PreStop hook — задержка перед SIGTERM
lifecycle:
  preStop:
    exec:
      command:
        - /bin/sh
        - -c
        # Ждём пока Service уберёт pod из endpoints
        - sleep 15

# В самом приложении:
# - Обработать SIGTERM
# - Завершить текущие запросы
# - Закрыть соединения с БД
# - Выйти с кодом 0


# ===== AUTOSCALING =====

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20
  
  # Целевая утилизация (от requests)
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80
  
  # Поведение масштабирования
  behavior:
    # Медленное уменьшение (избегаем flapping)
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Pods
          value: 2
          periodSeconds: 60
    
    # Быстрое увеличение
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Pods
          value: 4
          periodSeconds: 15
        - type: Percent
          value: 100
          periodSeconds: 15
      selectPolicy: Max
```

### CI/CD интеграция

YAML

```
# ===== GITLAB CI ПРИМЕР =====

# .gitlab-ci.yml

stages:
  - lint
  - build
  - deploy-dev
  - deploy-staging
  - deploy-prod

variables:
  CHART_PATH: ./charts/myapp
  RELEASE_NAME: myapp

# Шаблон для деплоя
.deploy_template: &deploy_template
  image: alpine/helm:3.13
  before_script:
    - helm repo add bitnami https://charts.bitnami.com/bitnami
    - helm dependency update ${CHART_PATH}

# Линтинг
lint:
  stage: lint
  <<: *deploy_template
  script:
    - helm lint ${CHART_PATH}
    - helm lint ${CHART_PATH} -f environments/production/values.yaml
    # Проверка шаблонов
    - helm template ${RELEASE_NAME} ${CHART_PATH} -f environments/production/values.yaml > /dev/null
  only:
    - merge_requests
    - main

# Сборка и push образа
build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -t ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHA:0:8} .
    - docker push ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHA:0:8}
  only:
    - main

# Деплой в dev (автоматически)
deploy-dev:
  stage: deploy-dev
  <<: *deploy_template
  script:
    - helm upgrade --install ${RELEASE_NAME} ${CHART_PATH}
        --namespace dev
        --create-namespace
        -f environments/base/values.yaml
        -f environments/dev/values.yaml
        --set image.tag=${CI_COMMIT_SHA:0:8}
        --wait
        --timeout 5m
  environment:
    name: dev
    url: https://myapp-dev.example.com
  only:
    - main

# Деплой в staging (автоматически)
deploy-staging:
  stage: deploy-staging
  <<: *deploy_template
  script:
    - helm upgrade --install ${RELEASE_NAME} ${CHART_PATH}
        --namespace staging
        --create-namespace
        -f environments/base/values.yaml
        -f environments/staging/values.yaml
        --set image.tag=${CI_COMMIT_SHA:0:8}
        --wait
        --timeout 10m
  environment:
    name: staging
    url: https://myapp-staging.example.com
  only:
    - main

# Деплой в production (ручной)
deploy-prod:
  stage: deploy-prod
  <<: *deploy_template
  script:
    # Предварительный diff
    - helm diff upgrade ${RELEASE_NAME} ${CHART_PATH}
        --namespace production
        -f environments/base/values.yaml
        -f environments/production/values.yaml
        --set image.tag=${CI_COMMIT_SHA:0:8}
        || true
    # Деплой с atomic
    - helm upgrade --install ${RELEASE_NAME} ${CHART_PATH}
        --namespace production
        --create-namespace
        -f environments/base/values.yaml
        -f environments/production/values.yaml
        --set image.tag=${CI_COMMIT_SHA:0:8}
        --atomic
        --timeout 10m
        --history-max 10
        --description "Deploy from CI: ${CI_PIPELINE_URL}"
  environment:
    name: production
    url: https://myapp.example.com
  when: manual
  only:
    - main

# Откат (ручной)
rollback-prod:
  stage: deploy-prod
  <<: *deploy_template
  script:
    - echo "Current revision:"
    - helm history ${RELEASE_NAME} --namespace production | tail -5
    - echo "Rolling back..."
    - helm rollback ${RELEASE_NAME} --namespace production --wait
    - echo "Rollback complete. New status:"
    - helm status ${RELEASE_NAME} --namespace production
  environment:
    name: production
  when: manual
  only:
    - main
```

---

## 10. Заключение

### Ключевые концепции для запоминания

text

```
╔═══════════════════════════════════════════════════════════════════╗
║                    HELM — КЛЮЧЕВЫЕ КОНЦЕПЦИИ                      ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  CHART (чарт)                                                     ║
║  └── Пакет с шаблонами Kubernetes-манифестов                      ║
║      • Chart.yaml — метаданные                                    ║
║      • values.yaml — конфигурация по умолчанию                    ║
║      • templates/ — шаблоны манифестов                            ║
║                                                                   ║
║  RELEASE (релиз)                                                  ║
║  └── Конкретная установка чарта в кластер                         ║
║      • Уникальное имя в namespace                                 ║
║      • Имеет историю ревизий                                      ║
║                                                                   ║
║  REVISION (ревизия)                                               ║
║  └── Версия релиза                                                ║
║      • Создаётся при каждом install/upgrade                       ║
║      • Можно откатиться на любую                                  ║
║                                                                   ║
║  VALUES (значения)                                                ║
║  └── Конфигурация для шаблонов                                    ║
║      • values.yaml — defaults                                     ║
║      • -f file.yaml — переопределение из файла                    ║
║      • --set key=value — переопределение из CLI                   ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Основные команды

text

```
╔═══════════════════════════════════════════════════════════════════╗
║                    ОСНОВНЫЕ КОМАНДЫ                               ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  УСТАНОВКА / ОБНОВЛЕНИЕ                                           ║
║  helm upgrade --install myapp ./mychart -f values.yaml            ║
║                                                                   ║
║  ПРОСМОТР                                                         ║
║  helm list                    # Список релизов                    ║
║  helm status myapp            # Статус релиза                     ║
║  helm history myapp           # История ревизий                   ║
║  helm get values myapp        # Values релиза                     ║
║  helm get manifest myapp      # Манифесты релиза                  ║
║                                                                   ║
║  РЕНДЕРИНГ (предпросмотр)                                         ║
║  helm template myapp ./mychart -f values.yaml                     ║
║  helm upgrade --install myapp ./mychart --dry-run                 ║
║  helm diff upgrade myapp ./mychart -f values.yaml                 ║
║                                                                   ║
║  ОТКАТ                                                            ║
║  helm rollback myapp 2        # На ревизию 2                      ║
║  helm rollback myapp          # На предыдущую                     ║
║                                                                   ║
║  УДАЛЕНИЕ                                                         ║
║  helm uninstall myapp                                             ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Чек-лист для production

text

```
╔═══════════════════════════════════════════════════════════════════╗
║                 PRODUCTION CHECKLIST                              ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  □ Конкретная версия образа (не latest)                           ║
║  □ Resource requests и limits заданы                              ║
║  □ Liveness и readiness probes настроены                          ║
║  □ Security context настроен (runAsNonRoot, etc.)                 ║
║  □ PodDisruptionBudget создан                                     ║
║  □ Минимум 2-3 реплики                                            ║
║  □ Pod anti-affinity настроен                                     ║
║  □ HPA настроен (если нужен)                                      ║
║  □ Network policies настроены                                     ║
║  □ Secrets НЕ в git (используйте external secrets/sealed)         ║
║  □ Graceful shutdown настроен                                     ║
║  □ Мониторинг и алерты настроены                                  ║
║  □ История ревизий ограничена (--history-max)                     ║
║  □ CI/CD использует --atomic для автоотката                       ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝
```