# 📗 ЧАСТЬ 1 — ОСНОВЫ (JUNIOR)

---

## 1. GitLab Runner

text

```
┌────────────────────────────────────────────────────────────┐
│                    GITLAB RUNNER                           │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  GitLab Runner — это агент (программа), которая           │
│  ВЫПОЛНЯЕТ задачи CI/CD pipeline.                         │
│                                                            │
│  Архитектура:                                             │
│  ┌──────────────┐                                         │
│  │   GitLab     │  ← Сервер (GitLab.com или self-hosted) │
│  │   Server     │                                         │
│  └──────┬───────┘                                         │
│         │                                                 │
│         │ API (задачи для выполнения)                    │
│         │                                                 │
│    ┌────┴─────┬─────────┬─────────┐                      │
│    ▼          ▼         ▼         ▼                      │
│  ┌────┐    ┌────┐    ┌────┐    ┌────┐                   │
│  │R1  │    │R2  │    │R3  │    │R4  │  ← Runners        │
│  └────┘    └────┘    └────┘    └────┘                   │
│  Runner1   Runner2   Runner3   Runner4                   │
│  (Docker)  (Shell)   (K8s)     (SSH)                     │
│                                                            │
│  Что делает Runner:                                       │
│  1. Получает задачу от GitLab Server                     │
│  2. Клонирует репозиторий                                │
│  3. Выполняет скрипты из .gitlab-ci.yml                  │
│  4. Отправляет результаты обратно на сервер              │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Типы Runner'ов (по Executor)

text

```
┌────────────────────────────────────────────────────────────┐
│              ТИПЫ EXECUTOR'ОВ (КАК ЗАПУСКАТЬ)              │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  1. SHELL EXECUTOR                                        │
│     ──────────────                                         │
│     Выполняет команды напрямую в shell на машине runner   │
│                                                            │
│     ✅ Простота                                            │
│     ✅ Быстрая работа                                      │
│     ❌ Загрязнение окружения между запусками              │
│     ❌ Нет изоляции между джобами                         │
│                                                            │
│     Использование:                                        │
│     • Локальная разработка                               │
│     • Простые скрипты                                    │
│                                                            │
│  ────────────────────────────────────────────────────────  │
│                                                            │
│  2. DOCKER EXECUTOR ⭐ (Самый популярный)                 │
│     ───────────────                                        │
│     Каждый джоб запускается в отдельном Docker контейнере│
│                                                            │
│     ✅ Изоляция                                            │
│     ✅ Чистое окружение каждый раз                        │
│     ✅ Любой образ (node, python, go...)                  │
│     ✅ Параллельное выполнение                            │
│     ❌ Немного медленнее (запуск контейнера)              │
│                                                            │
│     Использование:                                        │
│     • Production CI/CD                                   │
│     • Изолированные сборки                               │
│                                                            │
│  ────────────────────────────────────────────────────────  │
│                                                            │
│  3. KUBERNETES EXECUTOR                                   │
│     ───────────────────                                    │
│     Каждый джоб — под в Kubernetes кластере              │
│                                                            │
│     ✅ Масштабируемость                                    │
│     ✅ Автоматическое управление ресурсами                │
│     ✅ Интеграция с инфраструктурой K8s                   │
│     ❌ Сложность настройки                                │
│                                                            │
│     Использование:                                        │
│     • Большие проекты                                    │
│     • Облачная инфраструктура                            │
│                                                            │
│  ────────────────────────────────────────────────────────  │
│                                                            │
│  4. SSH EXECUTOR                                          │
│     ─────────────                                          │
│     Подключается по SSH к удалённой машине                │
│                                                            │
│  5. VirtualBox / Parallels                                │
│     Запуск в виртуальных машинах                          │
│                                                            │
│  6. Custom Executor                                       │
│     Свой кастомный способ выполнения                      │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Типы Runner'ов (по Scope)

text

```
┌────────────────────────────────────────────────────────────┐
│              ТИПЫ RUNNER'ОВ (ПО ОБЛАСТИ ВИДИМОСТИ)         │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  1. SHARED RUNNERS                                        │
│     ───────────────                                        │
│     Доступны ВСЕМ проектам в GitLab instance              │
│                                                            │
│     • Настраиваются администратором GitLab               │
│     • Используются по умолчанию                          │
│     • На GitLab.com предоставляются бесплатно (лимиты)   │
│                                                            │
│     ┌──────────────────────────────────────────┐          │
│     │           GitLab Instance                │          │
│     │  ┌─────────┐  ┌─────────┐  ┌─────────┐  │          │
│     │  │Project A│  │Project B│  │Project C│  │          │
│     │  └────┬────┘  └────┬────┘  └────┬────┘  │          │
│     │       └───────────┬┴─────────────┘       │          │
│     │                   ▼                       │          │
│     │           [Shared Runner]                 │          │
│     └──────────────────────────────────────────┘          │
│                                                            │
│  ────────────────────────────────────────────────────────  │
│                                                            │
│  2. GROUP RUNNERS                                         │
│     ─────────────                                          │
│     Доступны всем проектам в ГРУППЕ                       │
│                                                            │
│     • Настраиваются на уровне группы                     │
│     • Общие для подпроектов                              │
│                                                            │
│     ┌──────────────────────────────────────────┐          │
│     │              Group                        │          │
│     │  ┌─────────┐  ┌─────────┐                │          │
│     │  │Project A│  │Project B│                │          │
│     │  └────┬────┘  └────┬────┘                │          │
│     │       └───────────┬┘                      │          │
│     │                   ▼                       │          │
│     │           [Group Runner]                  │          │
│     └──────────────────────────────────────────┘          │
│                                                            │
│  ────────────────────────────────────────────────────────  │
│                                                            │
│  3. SPECIFIC (PROJECT) RUNNERS                            │
│     ──────────────────────────                             │
│     Доступны ТОЛЬКО конкретному проекту                   │
│                                                            │
│     • Настраиваются на уровне проекта                    │
│     • Полный контроль                                    │
│     • Можно использовать специальное окружение           │
│                                                            │
│     ┌──────────────────────────────────────────┐          │
│     │           Project A                       │          │
│     │                   ▼                       │          │
│     │         [Specific Runner]                 │          │
│     │        (только для Project A)             │          │
│     └──────────────────────────────────────────┘          │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## 2. Триггеры запуска пайплайнов

text

```
┌────────────────────────────────────────────────────────────┐
│              ТРИГГЕРЫ ЗАПУСКА PIPELINE                     │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  1. PUSH (Коммит в ветку)                                 │
│     ────────────────────                                   │
│     Автоматический запуск при git push                    │
│                                                            │
│     rules:                                                │
│       - if: $CI_PIPELINE_SOURCE == "push"                 │
│                                                            │
│  ────────────────────────────────────────────────────────  │
│                                                            │
│  2. MERGE REQUEST (MR/PR)                                 │
│     ────────────────────────                               │
│     При создании/обновлении Merge Request                 │
│                                                            │
│     rules:                                                │
│       - if: $CI_PIPELINE_SOURCE == "merge_request_event"  │
│                                                            │
│  ────────────────────────────────────────────────────────  │
│                                                            │
│  3. TAG                                                   │
│     ───                                                    │
│     При создании git tag                                  │
│                                                            │
│     rules:                                                │
│       - if: $CI_COMMIT_TAG                                │
│                                                            │
│  ────────────────────────────────────────────────────────  │
│                                                            │
│  4. SCHEDULE                                              │
│     ────────                                               │
│     По расписанию (cron)                                  │
│                                                            │
│     rules:                                                │
│       - if: $CI_PIPELINE_SOURCE == "schedule"             │
│                                                            │
│  ────────────────────────────────────────────────────────  │
│                                                            │
│  5. WEB (Manual)                                          │
│     ────────────                                           │
│     Ручной запуск из Web UI                               │
│                                                            │
│     rules:                                                │
│       - if: $CI_PIPELINE_SOURCE == "web"                  │
│                                                            │
│  ────────────────────────────────────────────────────────  │
│                                                            │
│  6. API / TRIGGER                                         │
│     ─────────────                                          │
│     Запуск через API с токеном                            │
│                                                            │
│     rules:                                                │
│       - if: $CI_PIPELINE_SOURCE == "trigger"              │
│                                                            │
│  ────────────────────────────────────────────────────────  │
│                                                            │
│  7. PARENT PIPELINE                                       │
│     ───────────────                                        │
│     Запуск из другого pipeline (multi-project)            │
│                                                            │
│     rules:                                                │
│       - if: $CI_PIPELINE_SOURCE == "parent_pipeline"      │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Примеры использования триггеров

YAML

```
# .gitlab-ci.yml

# ═══════════════════════════════════════════════════════════
#              ТРИГГЕРЫ ЗАПУСКА
# ═══════════════════════════════════════════════════════════

# 1. Запуск на каждый push в любую ветку
job1:
  script:
    - echo "Running on every push"
  rules:
    - if: $CI_PIPELINE_SOURCE == "push"

# ────────────────────────────────────────────────────────────

# 2. Запуск только при Merge Request
job2:
  script:
    - echo "Running on MR"
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

# ────────────────────────────────────────────────────────────

# 3. Запуск только при push в main/master
job3:
  script:
    - echo "Running on main branch"
  rules:
    - if: $CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "push"

# ────────────────────────────────────────────────────────────

# 4. Запуск только при создании тега
deploy:
  script:
    - echo "Deploying version $CI_COMMIT_TAG"
  rules:
    - if: $CI_COMMIT_TAG

# Или только для тегов формата v*
deploy_versioned:
  script:
    - echo "Deploying $CI_COMMIT_TAG"
  rules:
    - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/

# ────────────────────────────────────────────────────────────

# 5. Запуск по расписанию (настраивается в UI)
nightly_tests:
  script:
    - echo "Running nightly tests"
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"

# ────────────────────────────────────────────────────────────

# 6. Комбинация условий (push в main ИЛИ MR)
job6:
  script:
    - echo "Running on main or MR"
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

# ────────────────────────────────────────────────────────────

# 7. Исключение веток
job7:
  script:
    - echo "Running on all branches except dev"
  rules:
    - if: $CI_COMMIT_BRANCH != "dev"

# ────────────────────────────────────────────────────────────

# 8. Запуск только при изменении конкретных файлов
test_frontend:
  script:
    - npm test
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      changes:
        - "frontend/**/*"
        - "package.json"
```

---

# 📘 ЧАСТЬ 2 — СРЕДНИЙ УРОВЕНЬ (MIDDLE)

---

## 3. Подключение нового Runner'а

text

```
┌────────────────────────────────────────────────────────────┐
│              УСТАНОВКА И РЕГИСТРАЦИЯ RUNNER                │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ЭТАПЫ:                                                   │
│  1. Установить GitLab Runner на машину                   │
│  2. Зарегистрировать Runner в GitLab                      │
│  3. Настроить executor и параметры                        │
│  4. Запустить Runner                                      │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Шаг 1: Установка GitLab Runner

Bash

```
# ═══════════════════════════════════════════════════════════
#              УСТАНОВКА GITLAB RUNNER
# ═══════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────
# Linux (Debian/Ubuntu)
# ─────────────────────────────────────────────────────────

# Добавить официальный репозиторий GitLab
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash

# Установить GitLab Runner
sudo apt-get install gitlab-runner

# ─────────────────────────────────────────────────────────
# Linux (RHEL/CentOS)
# ─────────────────────────────────────────────────────────

curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | sudo bash
sudo yum install gitlab-runner

# ─────────────────────────────────────────────────────────
# macOS
# ─────────────────────────────────────────────────────────

brew install gitlab-runner

# ─────────────────────────────────────────────────────────
# Docker (запустить Runner в контейнере)
# ─────────────────────────────────────────────────────────

docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest

# ─────────────────────────────────────────────────────────
# Kubernetes (через Helm)
# ─────────────────────────────────────────────────────────

helm repo add gitlab https://charts.gitlab.io
helm install gitlab-runner gitlab/gitlab-runner \
  --set gitlabUrl=https://gitlab.com/ \
  --set runnerRegistrationToken=YOUR_TOKEN
```

### Шаг 2: Регистрация Runner

Bash

```
# ═══════════════════════════════════════════════════════════
#              РЕГИСТРАЦИЯ RUNNER
# ═══════════════════════════════════════════════════════════

# Интерактивная регистрация
sudo gitlab-runner register

# Вас спросят:
# 1. GitLab instance URL:
#    https://gitlab.com/  (или ваш self-hosted)
#
# 2. Registration token:
#    Берётся из Settings → CI/CD → Runners
#
# 3. Description:
#    my-docker-runner
#
# 4. Tags (метки для выбора runner):
#    docker,linux,production
#
# 5. Executor:
#    docker
#
# 6. Default Docker image:
#    alpine:latest


# ─────────────────────────────────────────────────────────
# Неинтерактивная регистрация (для автоматизации)
# ─────────────────────────────────────────────────────────

sudo gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --registration-token "YOUR_REGISTRATION_TOKEN" \
  --executor "docker" \
  --docker-image "alpine:latest" \
  --description "my-docker-runner" \
  --tag-list "docker,linux" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected"
```

### Шаг 3: Настройка config.toml

toml

```
# /etc/gitlab-runner/config.toml

# ═══════════════════════════════════════════════════════════
#         КОНФИГУРАЦИЯ RUNNER
# ═══════════════════════════════════════════════════════════

concurrent = 4  # Сколько джобов параллельно
check_interval = 0

[session_server]
  session_timeout = 1800

# ─────────────────────────────────────────────────────────
# Docker Executor
# ─────────────────────────────────────────────────────────

[[runners]]
  name = "my-docker-runner"
  url = "https://gitlab.com/"
  token = "RUNNER_TOKEN"
  executor = "docker"
  
  [runners.custom_build_dir]
  
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = false  # true если нужен Docker-in-Docker
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0

# ─────────────────────────────────────────────────────────
# Shell Executor
# ─────────────────────────────────────────────────────────

[[runners]]
  name = "shell-runner"
  url = "https://gitlab.com/"
  token = "RUNNER_TOKEN"
  executor = "shell"

# ─────────────────────────────────────────────────────────
# Kubernetes Executor
# ─────────────────────────────────────────────────────────

[[runners]]
  name = "k8s-runner"
  url = "https://gitlab.com/"
  token = "RUNNER_TOKEN"
  executor = "kubernetes"
  
  [runners.kubernetes]
    host = "https://kubernetes.default.svc"
    namespace = "gitlab-runner"
    privileged = false
    image = "alpine:latest"
```

### Шаг 4: Запуск и управление

Bash

```
# Запустить Runner
sudo gitlab-runner start

# Остановить
sudo gitlab-runner stop

# Перезапустить
sudo gitlab-runner restart

# Посмотреть статус
sudo gitlab-runner status

# Список зарегистрированных runners
sudo gitlab-runner list

# Проверить конфигурацию
sudo gitlab-runner verify

# Удалить runner
sudo gitlab-runner unregister --name my-docker-runner
# или по URL
sudo gitlab-runner unregister --url https://gitlab.com/ --token TOKEN
```

---

## 4. Pipeline только при MR в master

YAML

```
# .gitlab-ci.yml

# ═══════════════════════════════════════════════════════════
#         ЗАПУСК ТОЛЬКО ПРИ MR В MASTER
# ═══════════════════════════════════════════════════════════

# Способ 1: Через rules
test:
  script:
    - npm test
  rules:
    - if: $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"

# ─────────────────────────────────────────────────────────

# Способ 2: Более строгий (только MR именно в master)
test:
  script:
    - npm test
  rules:
    - if: >
        $CI_PIPELINE_SOURCE == "merge_request_event" &&
        $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"

# ─────────────────────────────────────────────────────────

# Способ 3: Использовать workflow (для всего pipeline)
workflow:
  rules:
    - if: >
        $CI_PIPELINE_SOURCE == "merge_request_event" &&
        $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"

stages:
  - test
  - build

test:
  stage: test
  script:
    - npm test

build:
  stage: build
  script:
    - npm run build

# ─────────────────────────────────────────────────────────

# Способ 4: С учётом также и коммитов в master
test:
  script:
    - npm test
  rules:
    # MR в master
    - if: >
        $CI_PIPELINE_SOURCE == "merge_request_event" &&
        $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"
    # Прямой push в master
    - if: $CI_COMMIT_BRANCH == "master"

# ─────────────────────────────────────────────────────────

# Способ 5: Более сложные условия
test:
  script:
    - npm test
  rules:
    # MR в master или main
    - if: >
        $CI_PIPELINE_SOURCE == "merge_request_event" &&
        ($CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master" ||
         $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "main")
      when: always
    # Запретить запуск в других случаях
    - when: never

# ─────────────────────────────────────────────────────────

# ПОЛЕЗНЫЕ ПЕРЕМЕННЫЕ ДЛЯ MR:
# $CI_MERGE_REQUEST_ID
# $CI_MERGE_REQUEST_IID
# $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME
# $CI_MERGE_REQUEST_TARGET_BRANCH_NAME
# $CI_MERGE_REQUEST_TITLE
# $CI_MERGE_REQUEST_LABELS
```

---

## 5. Работа с секретами (CI/CD Variables)

text

```
┌────────────────────────────────────────────────────────────┐
│              СЕКРЕТЫ В GITLAB CI                           │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  CI/CD Variables — безопасное хранение секретов            │
│  (пароли, API-ключи, токены)                              │
│                                                            │
│  УРОВНИ ПЕРЕМЕННЫХ:                                       │
│  1. Instance-level   — для всего GitLab instance          │
│  2. Group-level      — для группы проектов                │
│  3. Project-level    — для конкретного проекта            │
│                                                            │
│  ТИПЫ ПЕРЕМЕННЫХ:                                         │
│  • Variable — обычная переменная                          │
│  • File     — сохраняется в временный файл                │
│                                                            │
│  ЗАЩИТА:                                                  │
│  • Protected  — только для protected веток/тегов          │
│  • Masked     — скрывается в логах                        │
│  • Expanded   — подстановка других переменных             │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Настройка переменных через UI

text

```
┌────────────────────────────────────────────────────────────┐
│          ДОБАВЛЕНИЕ ПЕРЕМЕННЫХ ЧЕРЕЗ WEB UI                │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Settings → CI/CD → Variables → Expand → Add variable     │
│                                                            │
│  Key:   DATABASE_PASSWORD                                 │
│  Value: my_secret_password                                │
│  Type:  Variable (или File)                               │
│                                                            │
│  Flags:                                                   │
│  [✓] Protect variable  — только protected branches       │
│  [✓] Mask variable     — скрыть в логах                  │
│  [ ] Expand variable   — разрешить $VAR подстановки      │
│                                                            │
│  Environment scope: All (default)                         │
│    или конкретное: production, staging, dev               │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Использование переменных в .gitlab-ci.yml

YAML

```
# .gitlab-ci.yml

# ═══════════════════════════════════════════════════════════
#         ИСПОЛЬЗОВАНИЕ СЕКРЕТОВ
# ═══════════════════════════════════════════════════════════

# 1. Базовое использование
deploy:
  script:
    # Переменные доступны как обычные env vars
    - echo "Deploying to $DEPLOY_HOST"
    - sshpass -p "$SSH_PASSWORD" ssh user@$DEPLOY_HOST "deploy.sh"

# ─────────────────────────────────────────────────────────

# 2. Переменная типа File (для сертификатов, ключей)
deploy_with_key:
  script:
    # GitLab создаст временный файл с содержимым переменной
    # Путь к файлу будет в $SSH_PRIVATE_KEY
    - chmod 600 $SSH_PRIVATE_KEY
    - ssh -i $SSH_PRIVATE_KEY user@host "deploy.sh"

# Настройка в UI:
# Key:   SSH_PRIVATE_KEY
# Type:  File
# Value: -----BEGIN RSA PRIVATE KEY-----
#        ...содержимое приватного ключа...

# ─────────────────────────────────────────────────────────

# 3. Переменные для разных окружений
deploy_production:
  script:
    - deploy.sh
  environment:
    name: production
  # Переменные с scope "production" будут доступны только здесь

deploy_staging:
  script:
    - deploy.sh
  environment:
    name: staging
  # А здесь — переменные со scope "staging"

# ─────────────────────────────────────────────────────────

# 4. Определение переменных в .gitlab-ci.yml
variables:
  # Глобальные переменные для всех джобов
  NODE_VERSION: "18"
  DOCKER_DRIVER: overlay2
  
  # Можно ссылаться на другие переменные
  APP_URL: "https://$CI_COMMIT_REF_SLUG.example.com"

# Переопределение для конкретного джоба
test:
  variables:
    NODE_VERSION: "16"  # Переопределили глобальную
  script:
    - node --version

# ─────────────────────────────────────────────────────────

# 5. Защищённые переменные
deploy_production:
  script:
    # $PRODUCTION_API_KEY доступен только если:
    # 1. Переменная помечена как "Protected"
    # 2. Ветка/тег тоже protected
    - curl -H "Authorization: $PRODUCTION_API_KEY" api.example.com
  only:
    - master  # protected branch

# ─────────────────────────────────────────────────────────

# 6. Динамические переменные
build:
  script:
    # Вычисление переменной в runtime
    - export IMAGE_TAG="$CI_COMMIT_SHORT_SHA-$(date +%s)"
    - docker build -t myapp:$IMAGE_TAG .
    - echo "IMAGE_TAG=$IMAGE_TAG" > build.env
  artifacts:
    reports:
      dotenv: build.env  # Экспорт для следующих джобов

deploy:
  script:
    - echo "Deploying $IMAGE_TAG"  # Доступна из артефакта
  dependencies:
    - build

# ─────────────────────────────────────────────────────────

# 7. Работа с Docker Registry
docker_build:
  script:
    # $CI_REGISTRY, $CI_REGISTRY_USER, $CI_REGISTRY_PASSWORD
    # доступны автоматически для GitLab Container Registry
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG
```

### Встроенные предопределённые переменные

YAML

```
# ═══════════════════════════════════════════════════════════
#         ПРЕДОПРЕДЕЛЁННЫЕ ПЕРЕМЕННЫЕ GITLAB CI
# ═══════════════════════════════════════════════════════════

debug:
  script:
    # Git-related
    - echo "Branch: $CI_COMMIT_BRANCH"
    - echo "Tag: $CI_COMMIT_TAG"
    - echo "SHA: $CI_COMMIT_SHA"
    - echo "Short SHA: $CI_COMMIT_SHORT_SHA"
    - echo "Message: $CI_COMMIT_MESSAGE"
    - echo "Author: $CI_COMMIT_AUTHOR"
    
    # Project-related
    - echo "Project: $CI_PROJECT_NAME"
    - echo "Project ID: $CI_PROJECT_ID"
    - echo "Project Path: $CI_PROJECT_PATH"
    - echo "Project URL: $CI_PROJECT_URL"
    
    # Pipeline-related
    - echo "Pipeline ID: $CI_PIPELINE_ID"
    - echo "Pipeline Source: $CI_PIPELINE_SOURCE"
    - echo "Job ID: $CI_JOB_ID"
    - echo "Job Name: $CI_JOB_NAME"
    - echo "Stage: $CI_JOB_STAGE"
    
    # Registry-related
    - echo "Registry: $CI_REGISTRY"
    - echo "Registry Image: $CI_REGISTRY_IMAGE"
    - echo "Registry User: $CI_REGISTRY_USER"
    
    # MR-related (только в merge request pipelines)
    - echo "MR ID: $CI_MERGE_REQUEST_IID"
    - echo "MR Source Branch: $CI_MERGE_REQUEST_SOURCE_BRANCH_NAME"
    - echo "MR Target Branch: $CI_MERGE_REQUEST_TARGET_BRANCH_NAME"
    
    # Runner-related
    - echo "Runner: $CI_RUNNER_ID"
    - echo "Runner Tags: $CI_RUNNER_TAGS"
```

### Best Practices для секретов

YAML

```
# ═══════════════════════════════════════════════════════════
#         BEST PRACTICES
# ═══════════════════════════════════════════════════════════

# ❌ ПЛОХО: Секрет в коде
deploy:
  script:
    - docker login -u admin -p hardcoded_password

# ✅ ХОРОШО: Секрет в CI/CD переменной
deploy:
  script:
    - docker login -u $DOCKER_USER -p $DOCKER_PASSWORD

# ─────────────────────────────────────────────────────────

# ❌ ПЛОХО: Вывод секрета в лог
deploy:
  script:
    - echo "API Key: $API_KEY"  # Появится в логах!

# ✅ ХОРОШО: Отметить переменную как "Masked" в UI
# И не выводить её явно в echo

# ─────────────────────────────────────────────────────────

# ✅ ХОРОШО: Использовать Protected variables для prod
deploy_production:
  script:
    - deploy.sh
  only:
    - master  # protected branch
  # $PROD_API_KEY доступен только здесь

# ─────────────────────────────────────────────────────────

# ✅ ХОРОШО: Разные переменные для разных окружений
# В UI создать:
# DATABASE_URL (scope: production)  = prod-db.example.com
# DATABASE_URL (scope: staging)     = staging-db.example.com

deploy:
  script:
    - echo "Connecting to $DATABASE_URL"
  environment:
    name: $CI_COMMIT_REF_NAME  # production или staging
```

---

# 📕 ЧАСТЬ 3 — ПРОДВИНУТЫЙ УРОВЕНЬ (SENIOR)

---

## 6. Переиспользование кода в GitLab CI

text

```
┌────────────────────────────────────────────────────────────┐
│         СПОСОБЫ ПЕРЕИСПОЛЬЗОВАНИЯ КОДА                     │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  1. YAML Anchors (&, *, <<)                               │
│  2. extends                                               │
│  3. include                                               │
│  4. !reference                                            │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### 1. YAML Anchors

YAML

```
# ═══════════════════════════════════════════════════════════
#              YAML ANCHORS
# ═══════════════════════════════════════════════════════════

# Определить шаблон (anchor)
.node_template: &node_defaults
  image: node:18
  cache:
    paths:
      - node_modules/
  before_script:
    - npm install

# Переиспользовать через *
test_frontend:
  <<: *node_defaults  # Вставить все поля из шаблона
  script:
    - npm test

build_frontend:
  <<: *node_defaults
  script:
    - npm run build

# ─────────────────────────────────────────────────────────

# Комбинация нескольких anchors
.docker_template: &docker_defaults
  image: docker:latest
  services:
    - docker:dind

.deploy_template: &deploy_defaults
  only:
    - master
  environment:
    name: production

deploy_app:
  <<: [*docker_defaults, *deploy_defaults]  # Множественное слияние
  script:
    - docker build -t myapp .
    - docker push myapp
```

### 2. Extends (рекомендуемый способ)

YAML

```
# ═══════════════════════════════════════════════════════════
#              EXTENDS
# ═══════════════════════════════════════════════════════════

# Определить базовый джоб (начинается с точки = скрытый)
.node_job:
  image: node:18
  cache:
    paths:
      - node_modules/
  before_script:
    - npm ci

# Расширить базовый джоб
test:
  extends: .node_job
  script:
    - npm test

lint:
  extends: .node_job
  script:
    - npm run lint

build:
  extends: .node_job
  script:
    - npm run build
  artifacts:
    paths:
      - dist/

# ─────────────────────────────────────────────────────────

# Множественное наследование
.docker_job:
  image: docker:latest
  services:
    - docker:dind

.production_deploy:
  only:
    - master
  environment:
    name: production

deploy:
  extends:
    - .docker_job
    - .production_deploy
  script:
    - docker build -t app .
    - docker push app

# ─────────────────────────────────────────────────────────

# Переопределение полей
.base_test:
  image: node:18
  script:
    - npm test

test_node_16:
  extends: .base_test
  image: node:16  # Переопределили image

test_with_coverage:
  extends: .base_test
  script:
    - npm test -- --coverage  # Переопределили script
```

### 3. Include (внешние файлы)

YAML

```
# ═══════════════════════════════════════════════════════════
#              INCLUDE — ИМПОРТ КОНФИГУРАЦИЙ
# ═══════════════════════════════════════════════════════════

# .gitlab-ci.yml (основной файл)

include:
  # ─────────────────────────────────────────────────────────
  # 1. Локальный файл из того же репозитория
  # ─────────────────────────────────────────────────────────
  - local: '/ci/templates/docker-build.yml'
  
  # ─────────────────────────────────────────────────────────
  # 2. Файл из другого проекта GitLab
  # ─────────────────────────────────────────────────────────
  - project: 'my-group/ci-templates'
    ref: main  # Ветка/тег/коммит
    file: '/templates/deploy.yml'
  
  # ─────────────────────────────────────────────────────────
  # 3. Удалённый файл по HTTP(S)
  # ─────────────────────────────────────────────────────────
  - remote: 'https://example.com/ci/common.yml'
  
  # ─────────────────────────────────────────────────────────
  # 4. Шаблоны GitLab (встроенные)
  # ─────────────────────────────────────────────────────────
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml

stages:
  - build
  - test
  - deploy

# Использовать импортированные джобы
```

YAML

```
# ci/templates/docker-build.yml (отдельный файл)

.docker_build_template:
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG
```

### 4. !reference (вставка отдельных ключей)

YAML

```
# ═══════════════════════════════════════════════════════════
#              !reference (GitLab 13.9+)
# ═══════════════════════════════════════════════════════════

.setup:
  before_script:
    - echo "Setting up environment"
    - npm install

.teardown:
  after_script:
    - echo "Cleaning up"
    - rm -rf node_modules

test:
  # Вставить before_script из .setup
  before_script:
    - !reference [.setup, before_script]
    - echo "Additional setup for test"
  
  script:
    - npm test
  
  # Вставить after_script из .teardown
  after_script:
    - !reference [.teardown, after_script]

# ─────────────────────────────────────────────────────────

# Комбинация нескольких references
deploy:
  script:
    - !reference [.docker_login, script]
    - !reference [.build_image, script]
    - !reference [.push_image, script]
    - echo "Deploy complete"
```

### Практический пример: Полная структура

YAML

```
# .gitlab-ci.yml (основной файл)

include:
  - local: '/ci/base-jobs.yml'
  - local: '/ci/docker-jobs.yml'
  - project: 'company/ci-templates'
    file: '/security.yml'

variables:
  NODE_VERSION: "18"

stages:
  - test
  - build
  - deploy

test:
  extends: .node_test_job
  script:
    - npm test

build:
  extends: .docker_build_job
  variables:
    IMAGE_NAME: myapp

deploy_staging:
  extends: .deploy_template
  environment:
    name: staging

deploy_production:
  extends: .deploy_template
  environment:
    name: production
  only:
    - master
```

YAML

```
# ci/base-jobs.yml

.node_test_job:
  image: node:${NODE_VERSION}
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
  before_script:
    - npm ci
```

YAML

```
# ci/docker-jobs.yml

.docker_build_job:
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE/${IMAGE_NAME}:$CI_COMMIT_TAG .
    - docker push $CI_REGISTRY_IMAGE/${IMAGE_NAME}:$CI_COMMIT_TAG

.deploy_template:
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - curl -X POST "$DEPLOY_WEBHOOK_URL"
```

---

## 7. Параллельный запуск стадий

text

```
┌────────────────────────────────────────────────────────────┐
│              ПАРАЛЛЕЛЬНОЕ ВЫПОЛНЕНИЕ                       │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ДЖОБЫ ВНУТРИ ОДНОЙ СТАДИИ выполняются ПАРАЛЛЕЛЬНО        │
│  (если есть свободные runner'ы)                           │
│                                                            │
│  СТАДИИ выполняются ПОСЛЕДОВАТЕЛЬНО                       │
│                                                            │
│  ┌─────────────────────────────────────────────────────┐  │
│  │  Stage: test                                        │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐         │  │
│  │  │ test:py  │  │ test:js  │  │ test:go  │         │  │
│  │  │  (parallel) │  (parallel) │  (parallel) │      │  │
│  │  └──────────┘  └──────────┘  └──────────┘         │  │
│  └─────────────────────────────────────────────────────┘  │
│                     ▼ (после завершения всех)            │
│  ┌─────────────────────────────────────────────────────┐  │
│  │  Stage: build                                       │  │
│  │  ┌──────────┐  ┌──────────┐                        │  │
│  │  │ build:ui │  │ build:api│                        │  │
│  │  └──────────┘  └──────────┘                        │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### 1. Параллельные джобы в одной стадии

YAML

```
# ═══════════════════════════════════════════════════════════
#         ПАРАЛЛЕЛЬНЫЕ ДЖОБЫ (по умолчанию)
# ═══════════════════════════════════════════════════════════

stages:
  - test
  - build

# Все эти джобы запустятся ОДНОВРЕМЕННО
test_python:
  stage: test
  script:
    - pytest

test_javascript:
  stage: test
  script:
    - npm test

test_go:
  stage: test
  script:
    - go test ./...

lint_python:
  stage: test
  script:
    - flake8 .

# Stage "build" начнётся только после завершения ВСЕХ тестов
build:
  stage: build
  script:
    - docker build .
```

### 2. Параллельные вариации одного джоба

YAML

```
# ═══════════════════════════════════════════════════════════
#         PARALLEL (матричное тестирование)
# ═══════════════════════════════════════════════════════════

# Запустить один джоб N раз параллельно
test:
  parallel: 5  # Создаст 5 копий джоба
  script:
    - echo "Running instance $CI_NODE_INDEX of $CI_NODE_TOTAL"
    # $CI_NODE_INDEX — номер копии (1-5)
    # $CI_NODE_TOTAL — всего копий (5)

# ─────────────────────────────────────────────────────────

# Матричное тестирование (разные версии)
test:
  parallel:
    matrix:
      - PYTHON_VERSION: ["3.9", "3.10", "3.11", "3.12"]
        OS: ["ubuntu", "alpine"]
  image: python:${PYTHON_VERSION}-${OS}
  script:
    - python --version
    - pytest

# Создаст 8 джобов:
# test: [3.9, ubuntu]
# test: [3.9, alpine]
# test: [3.10, ubuntu]
# ...

# ─────────────────────────────────────────────────────────

# Реальный пример: разбиение тестов
test:
  parallel: 4
  script:
    # Используем $CI_NODE_INDEX для разбиения тестов
    - pytest --splits 4 --group $CI_NODE_INDEX

# ─────────────────────────────────────────────────────────

# Комбинация матрицы
e2e_test:
  parallel:
    matrix:
      - BROWSER: [chrome, firefox, safari]
        RESOLUTION: ["1920x1080", "1366x768", "768x1024"]
  script:
    - cypress run --browser $BROWSER --viewport $RESOLUTION

# Создаст 9 джобов (3 браузера × 3 разрешения)
```

### 3. Needs (DAG — направленный ациклический граф)

YAML

```
# ═══════════════════════════════════════════════════════════
#         NEEDS — ЗАВИСИМОСТИ МЕЖДУ ДЖОБАМИ
# ═══════════════════════════════════════════════════════════

stages:
  - build
  - test
  - deploy

# Обычно build_frontend ждал бы завершения build_backend
# Но с needs — может начаться сразу
build_backend:
  stage: build
  script:
    - go build -o api

build_frontend:
  stage: build
  script:
    - npm run build

# Тест API может начаться сразу после сборки API
# Не дожидаясь build_frontend!
test_api:
  stage: test
  needs: [build_backend]  # Зависит только от build_backend
  script:
    - go test ./...

# Тест UI зависит только от frontend
test_ui:
  stage: test
  needs: [build_frontend]
  script:
    - npm test

# E2E тесты нужны оба
test_e2e:
  stage: test
  needs:
    - build_backend
    - build_frontend
  script:
    - npm run e2e

# Deploy зависит от всех тестов
deploy:
  stage: deploy
  needs:
    - test_api
    - test_ui
    - test_e2e
  script:
    - kubectl apply -f deployment.yaml
```

Визуализация с `needs`:

text

```
build_backend ──┬──► test_api ─────┐
                │                   ├──► deploy
                └──► test_e2e ──────┤
                     ▲              │
build_frontend ──┬───┘              │
                 └──► test_ui ───────┘

Без needs (последовательно):
build_backend ────►│
build_frontend ───►│──► test_api ──►│
                         test_ui ───►│──► deploy
                         test_e2e ──►│
```

### 4. Когда нужен параллельный запуск

text

```
┌────────────────────────────────────────────────────────────┐
│         СИТУАЦИИ ДЛЯ ПАРАЛЛЕЛЬНОГО ЗАПУСКА                 │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  1. НЕЗАВИСИМЫЕ ТЕСТЫ                                     │
│     • Unit-тесты разных модулей                          │
│     • Тесты для разных языков/фреймворков                │
│     • Линтеры для разных частей кода                     │
│                                                            │
│  2. МАТРИЧНОЕ ТЕСТИРОВАНИЕ                                │
│     • Разные версии Python/Node/etc.                     │
│     • Разные ОС (Linux/Windows/macOS)                    │
│     • Разные браузеры для E2E                            │
│                                                            │
│  3. РАЗБИЕНИЕ ДОЛГИХ ТЕСТОВ                               │
│     • 1000 тестов → 10 джобов по 100 тестов              │
│     • Ускорение в N раз (при наличии N runner'ов)        │
│                                                            │
│  4. НЕЗАВИСИМЫЕ СБОРКИ                                    │
│     • Frontend и Backend собираются отдельно             │
│     • Разные микросервисы                                │
│     • Разные Docker-образы                               │
│                                                            │
│  5. НЕЗАВИСИМЫЕ ДЕПЛОИ                                    │
│     • В разные окружения (staging1, staging2, staging3)  │
│     • В разные регионы                                   │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Пример: Микросервисная архитектура

YAML

```
# ═══════════════════════════════════════════════════════════
#    ПАРАЛЛЕЛЬНАЯ CI/CD ДЛЯ МИКРОСЕРВИСОВ
# ═══════════════════════════════════════════════════════════

stages:
  - test
  - build
  - deploy

# ─────────────────────────────────────────────────────────
# Тесты — все параллельно
# ─────────────────────────────────────────────────────────

test:auth-service:
  stage: test
  script:
    - cd services/auth
    - go test ./...

test:api-gateway:
  stage: test
  script:
    - cd services/api-gateway
    - npm test

test:payment-service:
  stage: test
  script:
    - cd services/payment
    - pytest

# ─────────────────────────────────────────────────────────
# Сборки — тоже параллельно
# ─────────────────────────────────────────────────────────

build:auth-service:
  stage: build
  needs: [test:auth-service]  # Не ждёт другие тесты!
  script:
    - docker build -t auth-service ./services/auth

build:api-gateway:
  stage: build
  needs: [test:api-gateway]
  script:
    - docker build -t api-gateway ./services/api-gateway

build:payment-service:
  stage: build
  needs: [test:payment-service]
  script:
    - docker build -t payment-service ./services/payment

# ─────────────────────────────────────────────────────────
# Деплой — можно деплоить сервисы по готовности
# ─────────────────────────────────────────────────────────

deploy:auth-service:
  stage: deploy
  needs: [build:auth-service]
  script:
    - kubectl apply -f k8s/auth-service.yaml

# ...
```

Выигрыш во времени:

text

```
Последовательно: 3 теста (по 5 мин) + 3 сборки (по 3 мин) = 24 мин

Параллельно: max(тесты) + max(сборки) = 5 мин + 3 мин = 8 мин

Ускорение: 3x
```

---

## Полная шпаргалка

YAML

```
# ═══════════════════════════════════════════════════════════
#              GITLAB CI/CD ШПАРГАЛКА
# ═══════════════════════════════════════════════════════════

# ─── RUNNER ───
# Установка:
# curl -L https://packages.gitlab.com/.../script.deb.sh | sudo bash
# sudo apt-get install gitlab-runner
# sudo gitlab-runner register

# ─── ТРИГГЕРЫ ───
rules:
  - if: $CI_PIPELINE_SOURCE == "merge_request_event"
  - if: $CI_COMMIT_TAG
  - if: $CI_PIPELINE_SOURCE == "schedule"

# ─── MR В MASTER ───
rules:
  - if: >
      $CI_PIPELINE_SOURCE == "merge_request_event" &&
      $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"

# ─── СЕКРЕТЫ ───
# Settings → CI/CD → Variables
# $DATABASE_PASSWORD, $SSH_PRIVATE_KEY (type: File)

# ─── ПЕРЕИСПОЛЬЗОВАНИЕ КОДА ───
# extends:
.base: &base
  image: node:18
test:
  extends: .base

# include:
include:
  - local: '/ci/templates.yml'
  - template: Security/SAST.gitlab-ci.yml

# ─── ПАРАЛЛЕЛЬНЫЙ ЗАПУСК ───
# В одной стадии — автоматически параллельно
test1:
  stage: test
test2:
  stage: test  # Запустятся одновременно

# Матрица:
test:
  parallel:
    matrix:
      - VERSION: ["3.9", "3.10", "3.11"]

# needs (DAG):
deploy:
  needs: [build, test]  # Не ждёт всю стадию

# ─── ПЕРЕМЕННЫЕ ───
variables:
  VAR: "value"

# Предопределённые:
# $CI_COMMIT_SHA, $CI_COMMIT_BRANCH, $CI_COMMIT_TAG
# $CI_PIPELINE_SOURCE, $CI_MERGE_REQUEST_TARGET_BRANCH_NAME
# $CI_REGISTRY_IMAGE, $CI_REGISTRY_USER, $CI_REGISTRY_PASSWORD
```