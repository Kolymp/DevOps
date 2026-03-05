## 1. Continuous Integration (CI)

### Что такое CI

**Continuous Integration (Непрерывная интеграция)** — это практика разработки, при которой разработчики регулярно (несколько раз в день) интегрируют свой код в общий репозиторий. Каждая интеграция автоматически проверяется сборкой и тестами.

text

```
┌─────────────────────────────────────────────────────────────────────┐
│                    БЕЗ CI (традиционный подход)                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Developer A ──────────────────────────────────────┐                │
│  (работает 2 недели)                                │               │
│                                                      ▼              │
│  Developer B ──────────────────────────────┐     MERGE              │
│  (работает 2 недели)                        │    CONFLICT!          │
│                                             ▼                       │
│  Developer C ────────────────────┐       INTEGRATION                │
│  (работает 2 недели)              │         HELL                    │
│                                    ▼                                │
│                                                                     │
│  Проблемы:                                                          │
│  • Огромные merge conflicts                                         │
│  • "Integration hell" в конце спринта                               │
│  • Баги обнаруживаются поздно                                       │
│  • Сложно найти источник проблемы                                   │
│  • Релизы откладываются                                             │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                       С CI (современный подход)                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Developer A ─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─► main                            │
│               │ │ │ │ │ │ │ │ │ │                                   │
│  Developer B ─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─►  ✓                              │
│               │ │ │ │ │ │ │ │ │ │                                   │
│  Developer C ─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─►  ✓                              │
│                                                                     │
│               ↑ ↑ ↑ ↑ ↑ ↑ ↑ ↑ ↑ ↑                                   │
│               Частые маленькие интеграции                           │
│               + автоматические тесты                                │
│                                                                     │
│  Преимущества:                                                      │
│  • Маленькие, легко разрешаемые конфликты                           │
│  • Баги обнаруживаются сразу                                        │
│  • Всегда известно кто сломал билд                                  │
│  • Код всегда в рабочем состоянии                                   │
│  • Быстрые релизы                                                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Проблемы, которые решает CI

text

```
╔═══════════════════════════════════════════════════════════════════╗
║              ПРОБЛЕМЫ, КОТОРЫЕ РЕШАЕТ CI                          ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  1️⃣  INTEGRATION HELL                                             ║
║     ❌ Без CI: Разработчики работают изолированно неделями,      ║
║        потом мучительно мержат код                                ║
║     ✅ С CI: Маленькие частые интеграции, конфликты минимальны    ║
║                                                                    ║
║  2️⃣  "РАБОТАЕТ НА МОЕЙ МАШИНЕ"                                    ║
║     ❌ Без CI: Код работает локально, но падает у других          ║
║     ✅ С CI: Единая среда сборки, одинаковая для всех             ║
║                                                                    ║
║  3️⃣  ПОЗДНЕЕ ОБНАРУЖЕНИЕ БАГОВ                                    ║
║     ❌ Без CI: Баги находят через недели после написания кода     ║
║     ✅ С CI: Баги обнаруживаются сразу при коммите                ║
║                                                                    ║
║  4️⃣  СЛОЖНОСТЬ ПОИСКА ПРИЧИНЫ ПРОБЛЕМЫ                            ║
║     ❌ Без CI: "Что-то сломалось" в куче изменений                ║
║     ✅ С CI: Точно известно какой коммит сломал билд              ║
║                                                                    ║
║  5️⃣  ОТСУТСТВИЕ СТАНДАРТОВ КАЧЕСТВА                               ║
║     ❌ Без CI: Каждый пишет код как хочет                         ║
║     ✅ С CI: Автоматические проверки стиля, качества, тестов      ║
║                                                                    ║
║  6️⃣  СТРАХ РЕФАКТОРИНГА                                           ║
║     ❌ Без CI: "Лучше не трогать, а то сломается"                 ║
║     ✅ С CI: Тесты гарантируют что рефакторинг ничего не сломал   ║
║                                                                    ║
║  7️⃣  ДОЛГИЕ РЕЛИЗЫ                                                ║
║     ❌ Без CI: Релиз — это стресс и ручная работа на неделю       ║
║     ✅ С CI: Релиз можно делать хоть каждый день                  ║
║                                                                    ║
║  8️⃣  ОТСУТСТВИЕ ДОКУМЕНТАЦИИ ПРОЦЕССА СБОРКИ                      ║
║     ❌ Без CI: "Спроси Васю, он знает как собирать"               ║
║     ✅ С CI: Процесс сборки документирован в виде кода            ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Стадии CI пайплайна

text

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ТИПИЧНЫЙ CI PIPELINE                              │
└─────────────────────────────────────────────────────────────────────┘

   git push
      │
      ▼
┌─────────────────┐
│  1. CHECKOUT    │  Получение кода из репозитория
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  2. INSTALL     │  Установка зависимостей
│  DEPENDENCIES   │  (npm install, pip install, etc.)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  3. LINT        │  Проверка стиля кода
│  (Code Quality) │  (ESLint, Pylint, etc.)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  4. SECURITY    │  Проверка безопасности
│  SCAN           │  (SAST, dependency scan)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  5. UNIT        │  Модульные тесты
│  TESTS          │  (Jest, Pytest, JUnit)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  6. BUILD       │  Сборка приложения
│                 │  (компиляция, бандлинг)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  7. INTEGRATION │  Интеграционные тесты
│  TESTS          │  (тесты с реальной БД, API)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  8. BUILD       │  Сборка Docker образа
│  IMAGE          │  (docker build)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  9. SCAN        │  Сканирование образа
│  IMAGE          │  (Trivy, Clair)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 10. PUSH        │  Публикация образа
│ TO REGISTRY     │  (docker push)
└────────┬────────┘
         │
         ▼
      SUCCESS ✓
```

### Детальное описание каждой стадии

YAML

```
# ===== 1. CHECKOUT =====
# Получение исходного кода из Git репозитория

# Что происходит:
# - Клонирование репозитория
# - Checkout нужной ветки/коммита
# - Получение submodules (если есть)

# GitLab CI пример:
checkout:
  stage: checkout
  script:
    # GitLab CI делает checkout автоматически
    - git submodule update --init --recursive


# ===== 2. INSTALL DEPENDENCIES =====
# Установка зависимостей проекта

# Что происходит:
# - Скачивание библиотек
# - Кэширование для ускорения
# - Проверка lock-файлов

# Примеры:
install:
  stage: install
  script:
    # Node.js
    - npm ci  # ci лучше чем install (использует package-lock.json)
    
    # Python
    - pip install -r requirements.txt
    
    # Go
    - go mod download
    
    # Java
    - mvn dependency:resolve
  cache:
    paths:
      - node_modules/
      - .pip-cache/


# ===== 3. LINT (Code Quality) =====
# Проверка стиля и качества кода

# Что проверяется:
# - Соответствие code style (отступы, именование)
# - Потенциальные ошибки (неиспользуемые переменные)
# - Сложность кода (cyclomatic complexity)
# - Форматирование

lint:
  stage: lint
  script:
    # JavaScript/TypeScript
    - npx eslint src/
    - npx prettier --check src/
    
    # Python
    - flake8 src/
    - black --check src/
    - isort --check-only src/
    - mypy src/  # Type checking
    
    # Go
    - golangci-lint run
    
    # Terraform
    - terraform fmt -check
    - tflint


# ===== 4. SECURITY SCAN (SAST) =====
# Статический анализ безопасности

# Что проверяется:
# - Уязвимости в коде (SQL injection, XSS)
# - Уязвимые зависимости
# - Секреты в коде (пароли, ключи)
# - Небезопасные конфигурации

security:
  stage: security
  script:
    # Проверка зависимостей
    - npm audit --audit-level=high
    - safety check  # Python
    - snyk test     # Универсальный
    
    # SAST (Static Application Security Testing)
    - semgrep --config=auto src/
    - bandit -r src/  # Python
    - gosec ./...     # Go
    
    # Поиск секретов
    - gitleaks detect --source .
    - trufflehog filesystem .
    
    # Проверка Docker
    - hadolint Dockerfile


# ===== 5. UNIT TESTS =====
# Модульные тесты (быстрые, изолированные)

# Характеристики:
# - Тестируют отдельные функции/классы
# - Не требуют внешних сервисов
# - Должны быть быстрыми (< 1 мин для всех)
# - Покрытие кода (code coverage)

unit_tests:
  stage: test
  script:
    # JavaScript
    - npm test -- --coverage
    
    # Python
    - pytest tests/unit/ --cov=src --cov-report=xml
    
    # Go
    - go test -v -race -coverprofile=coverage.out ./...
    
    # Java
    - mvn test jacoco:report
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml


# ===== 6. BUILD =====
# Компиляция и сборка приложения

# Что происходит:
# - Компиляция (для compiled languages)
# - Бандлинг (для frontend)
# - Создание артефактов

build:
  stage: build
  script:
    # Frontend (React, Vue, etc.)
    - npm run build
    
    # Go
    - CGO_ENABLED=0 go build -o app ./cmd/server
    
    # Java
    - mvn package -DskipTests
    
    # .NET
    - dotnet publish -c Release -o out
  artifacts:
    paths:
      - dist/
      - app
      - target/*.jar


# ===== 7. INTEGRATION TESTS =====
# Интеграционные тесты (с реальными сервисами)

# Характеристики:
# - Тестируют взаимодействие компонентов
# - Используют реальную БД (testcontainers)
# - Медленнее unit тестов
# - Могут использовать docker-compose

integration_tests:
  stage: integration
  services:
    - postgres:15
    - redis:7
  variables:
    DATABASE_URL: postgres://postgres:postgres@postgres:5432/test
    REDIS_URL: redis://redis:6379
  script:
    - npm run test:integration
    # или
    - pytest tests/integration/


# ===== 8. BUILD IMAGE =====
# Сборка Docker образа

build_image:
  stage: package
  script:
    - docker build -t myapp:${CI_COMMIT_SHA} .
    
    # Multi-stage build для оптимизации
    - |
      docker build \
        --target production \
        --cache-from myapp:latest \
        -t myapp:${CI_COMMIT_SHA} \
        .


# ===== 9. SCAN IMAGE =====
# Сканирование Docker образа на уязвимости

scan_image:
  stage: scan
  script:
    # Trivy (популярный, бесплатный)
    - trivy image --severity HIGH,CRITICAL myapp:${CI_COMMIT_SHA}
    
    # Grype
    - grype myapp:${CI_COMMIT_SHA}
    
    # Clair
    - clairctl analyze myapp:${CI_COMMIT_SHA}


# ===== 10. PUSH TO REGISTRY =====
# Публикация образа в container registry

push_image:
  stage: publish
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker tag myapp:${CI_COMMIT_SHA} $CI_REGISTRY_IMAGE:${CI_COMMIT_SHA}
    - docker tag myapp:${CI_COMMIT_SHA} $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:${CI_COMMIT_SHA}
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
```

---

## 2. Continuous Delivery / Continuous Deployment (CD)

### Определения

text

```
╔═══════════════════════════════════════════════════════════════════╗
║           CONTINUOUS DELIVERY vs CONTINUOUS DEPLOYMENT             ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  CONTINUOUS DELIVERY (Непрерывная доставка)                       ║
║  ─────────────────────────────────────────                        ║
║  Код ГОТОВ к деплою в любой момент, но деплой в production       ║
║  требует РУЧНОГО подтверждения.                                   ║
║                                                                    ║
║  CI → Build → Test → Stage → [MANUAL APPROVAL] → Production       ║
║                                      ↑                             ║
║                               Человек решает                       ║
║                                                                    ║
║  Подходит когда:                                                   ║
║  • Нужен контроль над релизами (финансы, медицина)               ║
║  • Требуется координация с другими командами                      ║
║  • Регуляторные требования                                        ║
║                                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  CONTINUOUS DEPLOYMENT (Непрерывный деплой)                       ║
║  ──────────────────────────────────────────                       ║
║  Каждое изменение, прошедшее все тесты, АВТОМАТИЧЕСКИ             ║
║  деплоится в production.                                          ║
║                                                                    ║
║  CI → Build → Test → Stage → Production                           ║
║                           (автоматически)                          ║
║                                                                    ║
║  Подходит когда:                                                   ║
║  • Высокое покрытие тестами                                       ║
║  • Быстрый feedback loop критичен                                 ║
║  • Зрелая команда и процессы                                      ║
║  • Возможность быстрого отката                                    ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Визуализация CI/CD Pipeline

text

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        ПОЛНЫЙ CI/CD PIPELINE                             │
└─────────────────────────────────────────────────────────────────────────┘

                    ┌─────────────────────────────────────┐
                    │         CONTINUOUS INTEGRATION       │
                    └─────────────────────────────────────┘
                    
  git push          Lint &        Unit         Build &      Push
     │              Security      Tests        Package      Image
     ▼                 │            │            │            │
┌─────────┐      ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
│ COMMIT  │ ──►  │  CHECK  │─►│  TEST   │─►│  BUILD  │─►│  PUSH   │
└─────────┘      └─────────┘  └─────────┘  └─────────┘  └─────────┘
                                                              │
                                                              ▼
                    ┌─────────────────────────────────────┐
                    │        CONTINUOUS DELIVERY           │
                    └─────────────────────────────────────┘
                    
  Deploy to         Integration    E2E         Performance
  Staging           Tests          Tests       Tests
     │                │              │             │
┌─────────┐      ┌─────────┐   ┌─────────┐   ┌─────────┐
│ STAGING │ ──►  │  TEST   │ ─►│   E2E   │ ─►│  PERF   │
└─────────┘      └─────────┘   └─────────┘   └─────────┘
                                                   │
                                                   ▼
                    ┌─────────────────────────────────────┐
                    │       CONTINUOUS DEPLOYMENT          │
                    │       (или Manual Approval)          │
                    └─────────────────────────────────────┘
                    
  Manual or          Canary/        Monitor &
  Auto Trigger       Blue-Green     Rollback
     │                  │              │
┌─────────┐      ┌───────────┐   ┌─────────┐
│ APPROVE │ ──►  │ PRODUCTION│ ─►│ MONITOR │
└─────────┘      └───────────┘   └─────────┘
```

### Проблемы, которые решает CD

text

```
╔═══════════════════════════════════════════════════════════════════╗
║              ПРОБЛЕМЫ, КОТОРЫЕ РЕШАЕТ CD                           ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  1️⃣  РЕДКИЕ И БОЛЕЗНЕННЫЕ РЕЛИЗЫ                                  ║
║     ❌ Без CD: Релиз раз в месяц, огромный changelog, стресс      ║
║     ✅ С CD: Маленькие частые релизы, минимальный риск            ║
║                                                                    ║
║  2️⃣  РУЧНОЙ ДЕПЛОЙ                                                ║
║     ❌ Без CD: "Вася знает как деплоить", ручные скрипты          ║
║     ✅ С CD: Автоматизированный, повторяемый процесс              ║
║                                                                    ║
║  3️⃣  РАЗЛИЧИЯ МЕЖДУ ОКРУЖЕНИЯМИ                                   ║
║     ❌ Без CD: "На staging работало, на prod сломалось"           ║
║     ✅ С CD: Одинаковый процесс деплоя для всех окружений         ║
║                                                                    ║
║  4️⃣  ДОЛГИЙ TIME-TO-MARKET                                        ║
║     ❌ Без CD: Фича готова, но релиз через 2 недели               ║
║     ✅ С CD: Фича в production в тот же день                      ║
║                                                                    ║
║  5️⃣  СЛОЖНЫЙ ОТКАТ                                                ║
║     ❌ Без CD: "Как откатить? Где предыдущая версия?"             ║
║     ✅ С CD: Откат одной кнопкой/командой                         ║
║                                                                    ║
║  6️⃣  ОТСУТСТВИЕ АУДИТА                                            ║
║     ❌ Без CD: "Кто и когда деплоил?"                             ║
║     ✅ С CD: Полная история деплоев                               ║
║                                                                    ║
║  7️⃣  DOWNTIME ПРИ ДЕПЛОЕ                                          ║
║     ❌ Без CD: "Сайт недоступен 30 минут во время релиза"         ║
║     ✅ С CD: Zero-downtime deployments                            ║
║                                                                    ║
║  8️⃣  НЕВОЗМОЖНОСТЬ ЭКСПЕРИМЕНТТОВ                                 ║
║     ❌ Без CD: Слишком рискованно пробовать новое                 ║
║     ✅ С CD: Feature flags, canary releases, A/B тесты            ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

---

## 3. Инструменты CI/CD

### Обзор инструментов по категориям

text

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ИНСТРУМЕНТЫ CI/CD ПО КАТЕГОРИЯМ                   │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  CI/CD ПЛАТФОРМЫ                                                     │
├─────────────────────────────────────────────────────────────────────┤
│  • Jenkins         — Self-hosted, максимальная гибкость             │
│  • GitLab CI       — Встроен в GitLab, YAML конфигурация           │
│  • GitHub Actions  — Встроен в GitHub, marketplace actions          │
│  • CircleCI        — Cloud-first, быстрый                          │
│  • Travis CI       — Open source friendly                          │
│  • TeamCity        — JetBrains, хорош для Java                     │
│  • Azure DevOps    — Microsoft ecosystem                           │
│  • Bitbucket Pipelines — Atlassian ecosystem                       │
│  • Drone CI        — Container-native, lightweight                 │
│  • Tekton          — Kubernetes-native, CNCF                       │
│  • ArgoCD          — GitOps для Kubernetes                         │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  CODE QUALITY & LINTING                                              │
├─────────────────────────────────────────────────────────────────────┤
│  JavaScript/TypeScript:                                              │
│  • ESLint          — Линтер с правилами                             │
│  • Prettier        — Форматирование кода                            │
│  • TypeScript      — Type checking                                  │
│                                                                      │
│  Python:                                                             │
│  • Pylint, Flake8  — Линтеры                                        │
│  • Black           — Форматирование                                 │
│  • mypy            — Type checking                                  │
│  • isort           — Сортировка импортов                            │
│                                                                      │
│  Go:                                                                 │
│  • golangci-lint   — Мета-линтер (много линтеров в одном)          │
│  • gofmt           — Форматирование                                 │
│                                                                      │
│  Multi-language:                                                     │
│  • SonarQube       — Платформа анализа качества кода                │
│  • CodeClimate     — Cloud code quality                             │
│  • Codacy          — Automated code review                          │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  SECURITY SCANNING                                                   │
├─────────────────────────────────────────────────────────────────────┤
│  SAST (Static Application Security Testing):                        │
│  • Semgrep         — Быстрый, много правил, бесплатный             │
│  • SonarQube       — Security + Quality                             │
│  • Checkmarx       — Enterprise SAST                                │
│  • Fortify         — Enterprise SAST                                │
│  • Bandit          — Python security                                │
│  • Gosec           — Go security                                    │
│                                                                      │
│  Dependency Scanning:                                                │
│  • Snyk            — Популярный, интеграции с всем                  │
│  • Dependabot      — GitHub native, автоматические PR               │
│  • npm audit       — Встроен в npm                                  │
│  • Safety          — Python dependencies                            │
│  • OWASP Dependency-Check — Open source                             │
│                                                                      │
│  Secret Detection:                                                   │
│  • Gitleaks        — Поиск секретов в Git истории                   │
│  • TruffleHog      — Энтропия + паттерны                            │
│  • detect-secrets  — Yelp, baseline подход                          │
│                                                                      │
│  Container Scanning:                                                 │
│  • Trivy           — Бесплатный, быстрый, популярный               │
│  • Grype           — Anchore, точный                                │
│  • Clair           — CoreOS/Quay                                    │
│  • Snyk Container  — Snyk для контейнеров                          │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  TESTING                                                             │
├─────────────────────────────────────────────────────────────────────┤
│  Unit Testing:                                                       │
│  • Jest            — JavaScript, React                              │
│  • Pytest          — Python                                         │
│  • JUnit           — Java                                           │
│  • Go testing      — Go standard library                            │
│  • xUnit           — .NET                                           │
│                                                                      │
│  Integration/E2E Testing:                                            │
│  • Cypress         — Frontend E2E                                   │
│  • Playwright      — Microsoft, cross-browser                       │
│  • Selenium        — Classic, все языки                             │
│  • Testcontainers  — Docker для интеграционных тестов              │
│  • Postman/Newman  — API testing                                    │
│                                                                      │
│  Performance Testing:                                                │
│  • k6              — Modern, JS scripting                           │
│  • JMeter          — Classic, мощный                                │
│  • Gatling         — Scala, хорош для CI                            │
│  • Locust          — Python, distributed                            │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  BUILD & PACKAGE                                                     │
├─────────────────────────────────────────────────────────────────────┤
│  Container:                                                          │
│  • Docker          — Standard                                       │
│  • Buildah         — Rootless, OCI                                  │
│  • Kaniko          — Build в Kubernetes без Docker daemon           │
│  • Buildx          — Docker multi-platform builds                   │
│                                                                      │
│  Artifact Storage:                                                   │
│  • Docker Hub      — Public container registry                      │
│  • Harbor          — Self-hosted registry                           │
│  • AWS ECR         — AWS container registry                         │
│  • Google GCR/GAR  — GCP registries                                 │
│  • Artifactory     — Universal artifact management                  │
│  • Nexus           — Repository manager                             │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  DEPLOYMENT                                                          │
├─────────────────────────────────────────────────────────────────────┤
│  Kubernetes:                                                         │
│  • Helm            — Package manager для K8s                        │
│  • Kustomize       — Overlay-based customization                    │
│  • ArgoCD          — GitOps CD                                      │
│  • Flux            — GitOps CD                                      │
│                                                                      │
│  Infrastructure as Code:                                             │
│  • Terraform       — Multi-cloud IaC                                │
│  • Pulumi          — IaC с настоящими языками программирования      │
│  • AWS CDK         — AWS IaC                                        │
│  • Ansible         — Configuration management                       │
└─────────────────────────────────────────────────────────────────────┘
```

### Jenkins vs GitLab CI — детальное сравнение

text

```
╔═══════════════════════════════════════════════════════════════════╗
║                    JENKINS vs GITLAB CI                            ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  АРХИТЕКТУРА                                                       ║
║  ───────────                                                       ║
║                                                                    ║
║  Jenkins:                                                          ║
║  ┌─────────────┐     ┌──────────────┐                             ║
║  │   Jenkins   │────►│   Agent 1    │                             ║
║  │   Master    │────►│   Agent 2    │  (отдельные машины/поды)    ║
║  │             │────►│   Agent N    │                             ║
║  └─────────────┘     └──────────────┘                             ║
║                                                                    ║
║  GitLab CI:                                                        ║
║  ┌─────────────┐     ┌──────────────┐                             ║
║  │   GitLab    │────►│  Runner 1    │                             ║
║  │   Server    │────►│  Runner 2    │  (shared или dedicated)     ║
║  │             │────►│  Runner N    │                             ║
║  └─────────────┘     └──────────────┘                             ║
║                                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  КОНФИГУРАЦИЯ                                                      ║
║  ────────────                                                      ║
║                                                                    ║
║  Jenkins (Jenkinsfile):                                            ║
║  • Declarative Pipeline (проще) или Scripted (гибче)              ║
║  • Groovy DSL                                                      ║
║  • Shared Libraries для переиспользования                          ║
║  • Blue Ocean UI для визуализации                                  ║
║                                                                    ║
║  pipeline {                                                        ║
║    agent any                                                       ║
║    stages {                                                        ║
║      stage('Build') {                                              ║
║        steps {                                                     ║
║          sh 'npm install'                                          ║
║          sh 'npm run build'                                        ║
║        }                                                           ║
║      }                                                             ║
║    }                                                               ║
║  }                                                                 ║
║                                                                    ║
║  ───────────────────────────────────────────────                  ║
║                                                                    ║
║  GitLab CI (.gitlab-ci.yml):                                       ║
║  • Чистый YAML                                                     ║
║  • Include для переиспользования                                   ║
║  • Auto DevOps из коробки                                          ║
║  • Встроенные шаблоны                                              ║
║                                                                    ║
║  stages:                                                           ║
║    - build                                                         ║
║                                                                    ║
║  build:                                                            ║
║    stage: build                                                    ║
║    script:                                                         ║
║      - npm install                                                 ║
║      - npm run build                                               ║
║                                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  СРАВНИТЕЛЬНАЯ ТАБЛИЦА                                             ║
║  ─────────────────────                                             ║
║                                                                    ║
║  Критерий              │ Jenkins          │ GitLab CI             ║
║  ──────────────────────┼──────────────────┼───────────────────────║
║  Установка             │ Self-hosted      │ SaaS или Self-hosted  ║
║  Сложность настройки   │ Высокая          │ Низкая                ║
║  Конфигурация          │ Groovy DSL       │ YAML                  ║
║  Плагины               │ 1800+ плагинов   │ Встроенные фичи       ║
║  UI                    │ Устаревший*      │ Современный           ║
║  Git интеграция        │ Через плагины    │ Нативная              ║
║  Container Registry    │ Через плагины    │ Встроен               ║
║  Секреты               │ Credentials      │ CI/CD Variables       ║
║  Параллелизм           │ Настраиваемый    │ Из коробки            ║
║  Кэширование           │ Через плагины    │ Встроено              ║
║  Масштабируемость      │ Kubernetes agents│ Kubernetes runners    ║
║  Стоимость             │ Бесплатный       │ Бесплатный/Платный    ║
║  Enterprise support    │ CloudBees        │ GitLab Enterprise     ║
║                                                                    ║
║  * Blue Ocean UI улучшает ситуацию                                ║
║                                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  КОГДА ВЫБРАТЬ                                                     ║
║  ─────────────                                                     ║
║                                                                    ║
║  Jenkins:                                                          ║
║  ✅ Сложные, нестандартные пайплайны                               ║
║  ✅ Интеграция с legacy системами                                  ║
║  ✅ Уже есть экспертиза в команде                                  ║
║  ✅ Нужна максимальная гибкость                                    ║
║  ✅ Много разных проектов (не только Git)                          ║
║                                                                    ║
║  GitLab CI:                                                        ║
║  ✅ GitLab как основной Git                                        ║
║  ✅ Простота и скорость настройки                                  ║
║  ✅ Всё в одном месте (Git + CI + Registry + Issues)              ║
║  ✅ DevSecOps из коробки                                           ║
║  ✅ Kubernetes-native деплой                                       ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Примеры конфигурации

groovy

```
// ===== JENKINS (Jenkinsfile) =====

pipeline {
    // На каком агенте запускать
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: node
                    image: node:18
                    command: ["sleep", "infinity"]
                  - name: docker
                    image: docker:24-dind
                    securityContext:
                      privileged: true
            '''
        }
    }
    
    // Переменные окружения
    environment {
        REGISTRY = 'registry.example.com'
        IMAGE_NAME = 'myapp'
    }
    
    // Стадии пайплайна
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install') {
            steps {
                container('node') {
                    sh 'npm ci'
                }
            }
        }
        
        stage('Lint & Test') {
            parallel {
                stage('Lint') {
                    steps {
                        container('node') {
                            sh 'npm run lint'
                        }
                    }
                }
                stage('Test') {
                    steps {
                        container('node') {
                            sh 'npm test'
                        }
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                container('node') {
                    sh 'npm run build'
                }
            }
        }
        
        stage('Build Image') {
            steps {
                container('docker') {
                    sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} ."
                }
            }
        }
        
        stage('Push Image') {
            when {
                branch 'main'
            }
            steps {
                container('docker') {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-registry',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin $REGISTRY
                            docker push ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                sh 'kubectl apply -f k8s/staging/'
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            input {
                message "Deploy to production?"
                ok "Yes, deploy"
            }
            steps {
                sh 'kubectl apply -f k8s/production/'
            }
        }
    }
    
    // Действия после завершения
    post {
        always {
            junit 'reports/*.xml'
            archiveArtifacts artifacts: 'dist/**'
        }
        failure {
            slackSend channel: '#builds', color: 'danger',
                message: "Build failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
        success {
            slackSend channel: '#builds', color: 'good',
                message: "Build succeeded: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
    }
}
```

YAML

```
# ===== GITLAB CI (.gitlab-ci.yml) =====

# Глобальные настройки
default:
  image: node:18
  
  # Кэширование зависимостей
  cache:
    key:
      files:
        - package-lock.json
    paths:
      - node_modules/
    policy: pull-push

# Переменные
variables:
  DOCKER_HOST: tcp://docker:2376
  DOCKER_TLS_CERTDIR: "/certs"
  IMAGE_NAME: $CI_REGISTRY_IMAGE

# Стадии
stages:
  - install
  - validate
  - test
  - build
  - package
  - deploy

# Установка зависимостей
install:
  stage: install
  script:
    - npm ci
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 hour

# Параллельные проверки
lint:
  stage: validate
  script:
    - npm run lint
  needs:
    - install

security:
  stage: validate
  script:
    - npm audit --audit-level=high
  needs:
    - install
  allow_failure: true  # Не блокирует пайплайн

# Тесты
unit_tests:
  stage: test
  script:
    - npm test -- --coverage
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
  needs:
    - install

integration_tests:
  stage: test
  services:
    - postgres:15
    - redis:7
  variables:
    POSTGRES_DB: test
    POSTGRES_USER: test
    POSTGRES_PASSWORD: test
    DATABASE_URL: postgres://test:test@postgres:5432/test
  script:
    - npm run test:integration
  needs:
    - install

# Сборка приложения
build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
  needs:
    - lint
    - unit_tests

# Сборка Docker образа
build_image:
  stage: package
  image: docker:24
  services:
    - docker:24-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $IMAGE_NAME:$CI_COMMIT_SHA -t $IMAGE_NAME:latest .
    - docker push $IMAGE_NAME:$CI_COMMIT_SHA
    - docker push $IMAGE_NAME:latest
  needs:
    - build
  only:
    - main
    - develop

# Сканирование образа
scan_image:
  stage: package
  image:
    name: aquasec/trivy
    entrypoint: [""]
  script:
    - trivy image --exit-code 1 --severity HIGH,CRITICAL $IMAGE_NAME:$CI_COMMIT_SHA
  needs:
    - build_image
  only:
    - main
  allow_failure: true

# Деплой на staging
deploy_staging:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: staging
    url: https://staging.example.com
  script:
    - kubectl config use-context staging
    - kubectl set image deployment/myapp myapp=$IMAGE_NAME:$CI_COMMIT_SHA
    - kubectl rollout status deployment/myapp
  needs:
    - build_image
  only:
    - main

# Деплой на production (ручной)
deploy_production:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: production
    url: https://example.com
  script:
    - kubectl config use-context production
    - kubectl set image deployment/myapp myapp=$IMAGE_NAME:$CI_COMMIT_SHA
    - kubectl rollout status deployment/myapp
  needs:
    - deploy_staging
  when: manual
  only:
    - main

# Include внешних шаблонов
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - project: 'myorg/ci-templates'
    file: '/templates/notify.yml'
```

---

## 4. Работа с секретами в пайплайнах

### Типы секретов и угрозы

text

```
╔═══════════════════════════════════════════════════════════════════╗
║                    ТИПЫ СЕКРЕТОВ В CI/CD                           ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  1️⃣  CREDENTIALS ДЛЯ СЕРВИСОВ                                     ║
║     • API ключи (AWS, GCP, Stripe, etc.)                          ║
║     • Database passwords                                           ║
║     • Service account tokens                                       ║
║     • SSH keys                                                     ║
║                                                                    ║
║  2️⃣  REGISTRY CREDENTIALS                                         ║
║     • Docker Hub tokens                                            ║
║     • Private registry passwords                                   ║
║     • NPM tokens                                                   ║
║                                                                    ║
║  3️⃣  SIGNING KEYS                                                 ║
║     • Code signing certificates                                    ║
║     • GPG keys                                                     ║
║     • TLS certificates                                             ║
║                                                                    ║
║  4️⃣  DEPLOYMENT SECRETS                                           ║
║     • Kubeconfig                                                   ║
║     • Cloud provider credentials                                   ║
║     • SSH keys для серверов                                        ║
║                                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║                         УГРОЗЫ                                     ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  ⚠️  Секреты в Git истории                                        ║
║  ⚠️  Секреты в логах CI                                           ║
║  ⚠️  Секреты в Docker слоях                                       ║
║  ⚠️  Утечка через переменные окружения                            ║
║  ⚠️  Компрометация CI системы                                     ║
║  ⚠️  Insecure fork handling                                       ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Методы управления секретами

YAML

```
# ===== 1. CI/CD ПЛАТФОРМЕННЫЕ СЕКРЕТЫ =====
# Самый простой способ — использовать встроенные механизмы

# GitLab CI: Settings → CI/CD → Variables
# - Можно ограничить по branch/environment
# - Можно маскировать в логах
# - Можно защитить (только protected branches)

variables:
  # Обычная переменная
  APP_ENV: production
  
  # Секретная переменная (настраивается в UI)
  # DATABASE_PASSWORD: (из CI/CD Variables)
  # API_KEY: (из CI/CD Variables)

deploy:
  script:
    # Переменные автоматически доступны
    - echo "Connecting to database..."
    - ./deploy.sh
  variables:
    # Можно переопределить для конкретного job
    APP_ENV: staging


# ===== 2. GITHUB ACTIONS SECRETS =====

# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    # Environment secrets (разные для staging/production)
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy
        env:
          # Secrets доступны через ${{ secrets.NAME }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: |
          ./deploy.sh
      
      # OIDC для AWS (без долгоживущих ключей)
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsRole
          aws-region: us-east-1


# ===== 3. JENKINS CREDENTIALS =====

// Jenkinsfile
pipeline {
    agent any
    
    stages {
        stage('Deploy') {
            steps {
                // Username/Password
                withCredentials([usernamePassword(
                    credentialsId: 'docker-registry',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
                }
                
                // Secret Text
                withCredentials([string(
                    credentialsId: 'api-key',
                    variable: 'API_KEY'
                )]) {
                    sh 'curl -H "Authorization: Bearer $API_KEY" ...'
                }
                
                // SSH Key
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'deploy-key',
                    keyFileVariable: 'SSH_KEY'
                )]) {
                    sh 'ssh -i $SSH_KEY user@server'
                }
                
                // File
                withCredentials([file(
                    credentialsId: 'kubeconfig',
                    variable: 'KUBECONFIG'
                )]) {
                    sh 'kubectl get pods'
                }
            }
        }
    }
}


# ===== 4. HASHICORP VAULT =====

# Централизованное хранение секретов

# .gitlab-ci.yml с Vault
deploy:
  variables:
    VAULT_ADDR: https://vault.example.com
  before_script:
    # Аутентификация через JWT (GitLab CI)
    - export VAULT_TOKEN=$(vault write -field=token auth/jwt/login role=myapp jwt=$CI_JOB_JWT)
    
    # Получение секретов
    - export DATABASE_URL=$(vault kv get -field=url secret/myapp/database)
    - export API_KEY=$(vault kv get -field=key secret/myapp/api)
  script:
    - ./deploy.sh

# GitHub Actions с Vault
- name: Import Secrets
  uses: hashicorp/vault-action@v2
  with:
    url: https://vault.example.com
    method: jwt
    role: myapp
    secrets: |
      secret/data/myapp/database url | DATABASE_URL ;
      secret/data/myapp/api key | API_KEY


# ===== 5. AWS SECRETS MANAGER / PARAMETER STORE =====

# GitHub Actions
- name: Get secrets
  uses: aws-actions/aws-secretsmanager-get-secrets@v1
  with:
    secret-ids: |
      myapp/production/database
      myapp/production/api

# Или через CLI
- name: Get secrets
  run: |
    export DATABASE_URL=$(aws secretsmanager get-secret-value \
      --secret-id myapp/production/database \
      --query SecretString --output text)


# ===== 6. SOPS (ENCRYPTED FILES) =====

# Секреты хранятся в Git, но зашифрованы

# secrets.enc.yaml (зашифрованный)
database_url: ENC[AES256_GCM,data:...,iv:...,tag:...,type:str]
api_key: ENC[AES256_GCM,data:...,iv:...,tag:...,type:str]
sops:
  age:
    - recipient: age1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
      enc: |
        -----BEGIN AGE ENCRYPTED FILE-----
        ...
        -----END AGE ENCRYPTED FILE-----

# CI/CD
deploy:
  before_script:
    # Расшифровка (ключ в CI/CD переменных)
    - sops -d secrets.enc.yaml > secrets.yaml
    - export $(cat secrets.yaml | xargs)
  script:
    - ./deploy.sh
  after_script:
    # Удаление расшифрованного файла
    - rm -f secrets.yaml
```

### Best Practices для секретов

YAML

```
# ===== BEST PRACTICES =====

# 1️⃣  НИКОГДА не храните секреты в Git
# Даже если удалить — останутся в истории
# Используйте .gitignore и pre-commit hooks

# .gitignore
*.env
*.pem
*.key
secrets.yaml
credentials.json

# pre-commit hook (detect-secrets)
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']


# 2️⃣  Маскируйте секреты в логах
# GitLab CI — включено по умолчанию для masked variables
# Jenkins — автоматически для withCredentials
# GitHub Actions — автоматически для secrets

# Дополнительно: не выводите секреты в echo
# ❌ echo "API_KEY=$API_KEY"
# ✅ echo "API_KEY=***"


# 3️⃣  Ограничивайте scope секретов
# - Только для protected branches
# - Только для конкретных environments
# - Минимальные права (least privilege)

# GitLab: protected + masked + environment-scoped
# GitHub: Environment secrets с approvals


# 4️⃣  Ротируйте секреты регулярно
# - Автоматическая ротация в Vault/AWS SM
# - Alerts при долгоживущих secrets
# - Процедура ротации документирована


# 5️⃣  Используйте short-lived credentials
# - OIDC вместо долгоживущих ключей
# - Temporary credentials
# - Service account tokens с TTL

# GitHub Actions OIDC для AWS
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123456789012:role/GitHubActions
    aws-region: us-east-1
    # Нет ACCESS_KEY_ID и SECRET_ACCESS_KEY!


# 6️⃣  Не передавайте секреты в fork
# - Отключите secrets для forked PRs
# - Используйте pull_request_target осторожно

# GitHub Actions
on:
  pull_request:
    # Secrets НЕ доступны для форков по умолчанию

  pull_request_target:
    # ⚠️ Secrets доступны, но код из форка!
    # Используйте только для labeling, не для build


# 7️⃣  Аудит доступа к секретам
# - Логи доступа в Vault
# - CloudTrail для AWS
# - Alerts при аномальном доступе
```

---

## 5. Стратегии развёртывания

### Обзор стратегий

text

```
╔═══════════════════════════════════════════════════════════════════╗
║                    СТРАТЕГИИ РАЗВЁРТЫВАНИЯ                         ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  1️⃣  RECREATE (Big Bang)                                          ║
║     Остановить старое → Запустить новое                           ║
║     ❌ Downtime                                                    ║
║     ✅ Простота, гарантия версии                                  ║
║                                                                    ║
║  2️⃣  ROLLING UPDATE                                               ║
║     Постепенная замена инстансов                                   ║
║     ✅ Zero downtime                                               ║
║     ⚠️ Временно две версии одновременно                           ║
║                                                                    ║
║  3️⃣  BLUE-GREEN                                                   ║
║     Два идентичных окружения, переключение трафика                 ║
║     ✅ Мгновенный откат                                            ║
║     ❌ Двойные ресурсы                                             ║
║                                                                    ║
║  4️⃣  CANARY                                                       ║
║     Новая версия для части пользователей                           ║
║     ✅ Раннее обнаружение проблем                                  ║
║     ⚠️ Сложность управления трафиком                              ║
║                                                                    ║
║  5️⃣  A/B TESTING                                                  ║
║     Разные версии для разных сегментов                             ║
║     ✅ Бизнес-эксперименты                                         ║
║     ⚠️ Требует инфраструктуры для роутинга                        ║
║                                                                    ║
║  6️⃣  SHADOW (Mirror)                                              ║
║     Копия трафика на новую версию                                  ║
║     ✅ Тестирование под реальной нагрузкой                         ║
║     ⚠️ Не влияет на пользователей                                 ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Детальное описание стратегий

text

```
┌─────────────────────────────────────────────────────────────────────┐
│                    1. RECREATE                                       │
└─────────────────────────────────────────────────────────────────────┘

  Состояние 1: Все v1           Состояние 2: Остановка
  ┌────┐ ┌────┐ ┌────┐          ┌────┐ ┌────┐ ┌────┐
  │ v1 │ │ v1 │ │ v1 │    →     │ ❌ │ │ ❌ │ │ ❌ │   DOWNTIME!
  └────┘ └────┘ └────┘          └────┘ └────┘ └────┘
  
  Состояние 3: Все v2
  ┌────┐ ┌────┐ ┌────┐
  │ v2 │ │ v2 │ │ v2 │
  └────┘ └────┘ └────┘

  Использовать когда:
  • БД миграция несовместима с предыдущей версией
  • Приложение не поддерживает несколько версий
  • Dev/staging окружения
  • Scheduled maintenance window


┌─────────────────────────────────────────────────────────────────────┐
│                    2. ROLLING UPDATE                                 │
└─────────────────────────────────────────────────────────────────────┘

  Состояние 1     Состояние 2     Состояние 3     Состояние 4
  ┌────┐ ┌────┐   ┌────┐ ┌────┐   ┌────┐ ┌────┐   ┌────┐ ┌────┐
  │ v1 │ │ v1 │   │ v2 │ │ v1 │   │ v2 │ │ v2 │   │ v2 │ │ v2 │
  └────┘ └────┘ → └────┘ └────┘ → └────┘ └────┘ → └────┘ └────┘
  ┌────┐ ┌────┐   ┌────┐ ┌────┐   ┌────┐ ┌────┐   ┌────┐ ┌────┐
  │ v1 │ │ v1 │   │ v1 │ │ v1 │   │ v1 │ │ v2 │   │ v2 │ │ v2 │
  └────┘ └────┘   └────┘ └────┘   └────┘ └────┘   └────┘ └────┘

  ✅ Zero downtime
  ✅ Стандарт в Kubernetes
  ⚠️ Две версии работают одновременно (backward compatibility!)
  
  Kubernetes config:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1   # Макс недоступных подов
      maxSurge: 1         # Макс дополнительных подов


┌─────────────────────────────────────────────────────────────────────┐
│                    3. BLUE-GREEN                                     │
└─────────────────────────────────────────────────────────────────────┘

                        LOAD BALANCER
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
      ┌───────────────┐            ┌───────────────┐
      │  BLUE (v1)    │            │  GREEN (v2)   │
      │  ┌────┐ ┌────┐│            │  ┌────┐ ┌────┐│
      │  │ v1 │ │ v1 ││  ACTIVE    │  │ v2 │ │ v2 ││ IDLE
      │  └────┘ └────┘│            │  └────┘ └────┘│
      └───────────────┘            └───────────────┘
                │                         │
                └────── SWITCH ───────────┘
                         ↓
      ┌───────────────┐            ┌───────────────┐
      │  BLUE (v1)    │            │  GREEN (v2)   │
      │  ┌────┐ ┌────┐│            │  ┌────┐ ┌────┐│
      │  │ v1 │ │ v1 ││  IDLE      │  │ v2 │ │ v2 ││ ACTIVE
      │  └────┘ └────┘│            │  └────┘ └────┘│
      └───────────────┘            └───────────────┘

  ✅ Мгновенное переключение
  ✅ Мгновенный откат
  ✅ Тестирование перед переключением
  ❌ Двойные ресурсы (2x инфраструктура)
  ❌ Синхронизация данных между окружениями


┌─────────────────────────────────────────────────────────────────────┐
│                    4. CANARY                                         │
└─────────────────────────────────────────────────────────────────────┘

                        LOAD BALANCER
                             │
              ┌──────────────┴──────────────┐
              │ 90%                    10%  │
              ▼                             ▼
      ┌───────────────┐            ┌───────────────┐
      │  STABLE (v1)  │            │  CANARY (v2)  │
      │  ┌────┐ ┌────┐│            │     ┌────┐    │
      │  │ v1 │ │ v1 ││            │     │ v2 │    │
      │  ┌────┐ ┌────┐│            │     └────┘    │
      │  │ v1 │ │ v1 ││            │               │
      │  └────┘ └────┘│            │               │
      └───────────────┘            └───────────────┘

  Постепенное увеличение: 10% → 25% → 50% → 100%
  
  При проблемах: откат на 0%
  
  Мониторинг:
  • Error rate
  • Latency
  • Business metrics

  ✅ Минимальный risk exposure
  ✅ Раннее обнаружение проблем
  ⚠️ Требует хорошего мониторинга
  ⚠️ Сложнее реализовать


┌─────────────────────────────────────────────────────────────────────┐
│                    5. A/B TESTING                                    │
└─────────────────────────────────────────────────────────────────────┘

                        LOAD BALANCER
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
   User Segment A     User Segment B     User Segment C
   (US users)        (EU users)         (Beta users)
          │                  │                  │
          ▼                  ▼                  ▼
      ┌───────┐          ┌───────┐          ┌───────┐
      │  v1   │          │  v2   │          │  v3   │
      └───────┘          └───────┘          └───────┘

  Роутинг по:
  • Географии
  • User ID / Cookie
  • Headers
  • Query params
  • Random %

  ✅ Бизнес-эксперименты (conversion rate, etc.)
  ✅ Feature testing
  ⚠️ Требует feature flags infrastructure
  ⚠️ Статистическая значимость


┌─────────────────────────────────────────────────────────────────────┐
│                    6. SHADOW (Mirror)                                │
└─────────────────────────────────────────────────────────────────────┘

                        LOAD BALANCER
                             │
                    ┌────────┴────────┐
                    │    Responses    │
                    │    to users     │
                    ▼                 │
      ┌───────────────┐               │
      │  PRODUCTION   │ ──────────────┘
      │      v1       │
      └───────────────┘
             │
             │ COPY of requests
             │ (async, no response)
             ▼
      ┌───────────────┐
      │    SHADOW     │
      │      v2       │
      └───────────────┘

  ✅ Тестирование под реальной нагрузкой
  ✅ Нет влияния на пользователей
  ✅ Обнаружение проблем с производительностью
  ⚠️ Осторожно с side effects (emails, payments!)
  ⚠️ Двойная нагрузка на downstream сервисы
```

---

## 6. Zero Downtime Deployment

### Требования для Zero Downtime

text

```
╔═══════════════════════════════════════════════════════════════════╗
║              ТРЕБОВАНИЯ ДЛЯ ZERO DOWNTIME                          ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  1️⃣  BACKWARD COMPATIBLE API                                      ║
║     • Новая версия должна работать со старыми клиентами          ║
║     • Добавление полей — ОК, удаление — нет                       ║
║     • API versioning (/api/v1, /api/v2)                           ║
║                                                                    ║
║  2️⃣  BACKWARD COMPATIBLE DATABASE                                 ║
║     • Миграции должны работать с обеими версиями кода             ║
║     • Expand-Contract pattern                                      ║
║     • Нельзя удалять/переименовывать колонки сразу                ║
║                                                                    ║
║  3️⃣  HEALTH CHECKS                                                ║
║     • Liveness — приложение живо                                  ║
║     • Readiness — готово принимать трафик                         ║
║     • Startup — для медленного старта                             ║
║                                                                    ║
║  4️⃣  GRACEFUL SHUTDOWN                                            ║
║     • Завершение текущих запросов                                 ║
║     • Закрытие соединений                                         ║
║     • PreStop hooks                                                ║
║                                                                    ║
║  5️⃣  LOAD BALANCER DRAIN                                          ║
║     • Удаление из балансировки перед остановкой                   ║
║     • Ожидание завершения in-flight requests                      ║
║                                                                    ║
║  6️⃣  МИНИМУМ 2 РЕПЛИКИ                                            ║
║     • Одна всегда доступна во время обновления                    ║
║     • PodDisruptionBudget                                          ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Реализация в Kubernetes

YAML

```
# ===== DEPLOYMENT С ZERO DOWNTIME =====

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  # Минимум 2 реплики для HA
  replicas: 3
  
  # Rolling Update стратегия
  strategy:
    type: RollingUpdate
    rollingUpdate:
      # Не более 1 пода недоступно
      maxUnavailable: 0
      # Можно создать 1 дополнительный под
      maxSurge: 1
  
  template:
    spec:
      # Graceful shutdown
      terminationGracePeriodSeconds: 60
      
      containers:
        - name: myapp
          image: myapp:v2
          
          # Health checks
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
          
          startupProbe:
            httpGet:
              path: /startup
              port: 8080
            failureThreshold: 30
            periodSeconds: 10
          
          # Lifecycle hooks
          lifecycle:
            preStop:
              exec:
                command:
                  - /bin/sh
                  - -c
                  # Ждём пока LB уберёт pod из endpoints
                  # Затем даём время на завершение запросов
                  - |
                    sleep 15
                    # Или graceful shutdown endpoint
                    curl -X POST localhost:8080/shutdown

---
# PodDisruptionBudget — гарантия минимума доступных подов
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  # Минимум 2 пода всегда доступны
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp

---
# HPA для автомасштабирования
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```
## 6. Zero Downtime Deployment (продолжение)

### Expand-Contract Pattern для миграций БД

text

```
┌─────────────────────────────────────────────────────────────────────┐
│              EXPAND-CONTRACT DATABASE MIGRATION                      │
│                                                                      │
│  Проблема: Нужно переименовать колонку "name" → "full_name"         │
│                                                                      │
│  ❌ НЕПРАВИЛЬНО (downtime):                                         │
│  ALTER TABLE users RENAME COLUMN name TO full_name;                  │
│  → Старый код ломается мгновенно                                    │
│                                                                      │
│  ✅ ПРАВИЛЬНО (zero downtime):                                      │
│                                                                      │
│  ФАЗА 1: EXPAND (расширение)                                        │
│  ─────────────────────────                                          │
│  Деплой 1: Добавить новую колонку                                   │
│  ALTER TABLE users ADD COLUMN full_name VARCHAR(255);                │
│                                                                      │
│  Деплой 2: Код пишет в ОБЕ колонки                                  │
│  user.name = value;                                                  │
│  user.full_name = value;  // Дублирование                           │
│                                                                      │
│  Деплой 3: Бэкфилл старых данных                                    │
│  UPDATE users SET full_name = name WHERE full_name IS NULL;          │
│                                                                      │
│  ФАЗА 2: MIGRATE (миграция)                                         │
│  ─────────────────────────                                          │
│  Деплой 4: Код читает из full_name, пишет в обе                     │
│                                                                      │
│  Деплой 5: Код читает и пишет ТОЛЬКО в full_name                    │
│                                                                      │
│  ФАЗА 3: CONTRACT (сжатие)                                          │
│  ─────────────────────────                                          │
│  Деплой 6: Удалить старую колонку                                   │
│  ALTER TABLE users DROP COLUMN name;                                 │
│                                                                      │
│  Итого: 6 деплоев вместо 1, но ZERO DOWNTIME                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

YAML

```
# ===== ПРИМЕР: МИГРАЦИИ С FLYWAY =====

# V1__initial_schema.sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL
);

# V2__add_full_name_column.sql (EXPAND)
-- Добавляем новую колонку, старая остаётся
ALTER TABLE users ADD COLUMN full_name VARCHAR(255);

# V3__backfill_full_name.sql
-- Копируем данные (можно делать батчами для больших таблиц)
UPDATE users SET full_name = name WHERE full_name IS NULL;

# V4__make_full_name_not_null.sql
-- Теперь можно сделать NOT NULL
ALTER TABLE users ALTER COLUMN full_name SET NOT NULL;

# V5__drop_name_column.sql (CONTRACT)
-- Только после того как весь код перешёл на full_name
ALTER TABLE users DROP COLUMN name;


# ===== CI/CD Pipeline с миграциями =====

# .gitlab-ci.yml
stages:
  - migrate
  - deploy

# Миграция выполняется ДО деплоя нового кода
migrate:
  stage: migrate
  image: flyway/flyway:9
  script:
    - flyway -url=$DATABASE_URL -user=$DB_USER -password=$DB_PASSWORD migrate
  only:
    - main

deploy:
  stage: deploy
  script:
    - kubectl set image deployment/myapp myapp=$IMAGE
  needs:
    - migrate
  only:
    - main
```

### Blue-Green Deployment в Kubernetes

YAML

```
# ===== BLUE-GREEN С ARGO ROLLOUTS =====

# Установка Argo Rollouts
# kubectl create namespace argo-rollouts
# kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 3
  
  # Стратегия Blue-Green
  strategy:
    blueGreen:
      # Service для активной версии (production traffic)
      activeService: myapp-active
      
      # Service для preview версии (для тестирования)
      previewService: myapp-preview
      
      # Автоматическое переключение после N секунд
      # Или убрать для ручного переключения
      autoPromotionEnabled: true
      autoPromotionSeconds: 60
      
      # Сколько держать старую версию после переключения
      scaleDownDelaySeconds: 30
      
      # Ревизий для отката
      scaleDownDelayRevisionLimit: 2
  
  selector:
    matchLabels:
      app: myapp
  
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:v2
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5

---
# Active Service (production traffic)
apiVersion: v1
kind: Service
metadata:
  name: myapp-active
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080

---
# Preview Service (для тестирования новой версии)
apiVersion: v1
kind: Service
metadata:
  name: myapp-preview
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
```

Bash

```
# Управление Blue-Green через kubectl plugin
kubectl argo rollouts get rollout myapp --watch

# Ручное переключение (promote)
kubectl argo rollouts promote myapp

# Откат
kubectl argo rollouts undo myapp

# Abort (отмена текущего rollout)
kubectl argo rollouts abort myapp
```

### Canary Deployment в Kubernetes

YAML

```
# ===== CANARY С ARGO ROLLOUTS =====

apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 10
  
  strategy:
    canary:
      # Шаги канареечного развёртывания
      steps:
        # 1. 10% трафика на canary
        - setWeight: 10
        # Пауза для мониторинга (можно указать duration или ждать manual)
        - pause: { duration: 5m }
        
        # 2. 25% трафика
        - setWeight: 25
        - pause: { duration: 5m }
        
        # 3. 50% трафика
        - setWeight: 50
        - pause: { duration: 5m }
        
        # 4. 75% трафика
        - setWeight: 75
        - pause: { duration: 5m }
        
        # 5. 100% — полный rollout
        # (происходит автоматически после последнего шага)
      
      # Автоматический откат при проблемах
      # Интеграция с Prometheus для анализа метрик
      analysis:
        templates:
          - templateName: success-rate
        startingStep: 1  # Начать анализ с первого шага
        args:
          - name: service-name
            value: myapp
      
      # Traffic routing через Istio/Nginx
      trafficRouting:
        nginx:
          stableIngress: myapp-stable
          # или
        istio:
          virtualService:
            name: myapp-vsvc
            routes:
              - primary
  
  selector:
    matchLabels:
      app: myapp
  
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:v2
          ports:
            - containerPort: 8080

---
# AnalysisTemplate для автоматической проверки
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
spec:
  args:
    - name: service-name
  
  metrics:
    - name: success-rate
      # Интервал проверки
      interval: 1m
      
      # Условие успеха: success rate > 95%
      successCondition: result[0] >= 0.95
      
      # Условие провала
      failureCondition: result[0] < 0.90
      
      # Сколько провалов до отката
      failureLimit: 3
      
      # Prometheus query
      provider:
        prometheus:
          address: http://prometheus:9090
          query: |
            sum(rate(http_requests_total{
              service="{{args.service-name}}",
              status=~"2.*"
            }[5m])) /
            sum(rate(http_requests_total{
              service="{{args.service-name}}"
            }[5m]))
```

---

## 7. GitOps

### Что такое GitOps

text

```
╔═══════════════════════════════════════════════════════════════════╗
║                        GITOPS                                      ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  GitOps — методология управления инфраструктурой и приложениями,  ║
║  где Git репозиторий является единственным источником правды      ║
║  (Single Source of Truth) для декларативного описания системы.    ║
║                                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  4 ПРИНЦИПА GITOPS:                                                ║
║                                                                    ║
║  1️⃣  ДЕКЛАРАТИВНОСТЬ                                              ║
║     Система описывается декларативно (что, а не как)              ║
║     YAML манифесты вместо императивных скриптов                   ║
║                                                                    ║
║  2️⃣  ВЕРСИОНИРОВАНИЕ В GIT                                        ║
║     Вся конфигурация хранится в Git                               ║
║     История, аудит, откат через git revert                        ║
║                                                                    ║
║  3️⃣  АВТОМАТИЧЕСКОЕ ПРИМЕНЕНИЕ                                    ║
║     Изменения в Git автоматически применяются к системе           ║
║     GitOps оператор следит за репозиторием                        ║
║                                                                    ║
║  4️⃣  НЕПРЕРЫВНАЯ СВЕРКА (RECONCILIATION)                          ║
║     Оператор постоянно сравнивает desired (Git) и actual          ║
║     (кластер) состояния и исправляет drift                        ║
║                                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  PUSH vs PULL MODEL:                                               ║
║                                                                    ║
║  PUSH (традиционный CI/CD):                                        ║
║  CI System → kubectl apply → Cluster                               ║
║  • CI нужны credentials к кластеру                                ║
║  • Нет автоматического исправления drift                          ║
║                                                                    ║
║  PULL (GitOps):                                                    ║
║  Git ← GitOps Operator (в кластере) → Cluster                     ║
║  • Кластер сам тянет изменения                                    ║
║  • Автоматическое исправление drift                               ║
║  • CI не нужен доступ к кластеру                                  ║
║                                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  ИНСТРУМЕНТЫ:                                                      ║
║  • ArgoCD — самый популярный, Web UI, CNCF Graduated              ║
║  • Flux — модульный, CLI-ориентированный, CNCF Graduated          ║
║  • Jenkins X — CI/CD + GitOps                                      ║
║                                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  ПРЕИМУЩЕСТВА:                                                     ║
║  ✅ Полная история изменений в Git                                ║
║  ✅ Code review для инфраструктуры                                 ║
║  ✅ Откат через git revert                                         ║
║  ✅ Автоматическое исправление drift                               ║
║  ✅ Disaster recovery: git clone + sync                            ║
║  ✅ Безопасность: кластер не выставлен наружу                      ║
║                                                                    ║
║  НЕДОСТАТКИ:                                                       ║
║  ❌ Дополнительная сложность                                       ║
║  ❌ Задержка деплоя (polling)                                      ║
║  ❌ Управление секретами                                           ║
║  ❌ Git как bottleneck                                             ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

### GitOps Workflow

text

```
┌─────────────────────────────────────────────────────────────────────┐
│                        GITOPS WORKFLOW                               │
└─────────────────────────────────────────────────────────────────────┘

┌───────────────┐      ┌───────────────┐      ┌───────────────┐
│  APPLICATION  │      │    CONFIG     │      │   KUBERNETES  │
│  REPOSITORY   │      │  REPOSITORY   │      │    CLUSTER    │
│               │      │               │      │               │
│  (исходный    │      │  (манифесты   │      │  (ArgoCD/Flux │
│   код)        │      │   K8s)        │      │   оператор)   │
└───────┬───────┘      └───────┬───────┘      └───────┬───────┘
        │                      │                      │
        │                      │                      │
   1. git push            4. git push           6. auto sync
   (код)                  (image tag)              │
        │                      │                   │
        ▼                      ▼                   ▼
┌───────────────┐      ┌───────────────┐      ┌───────────────┐
│   CI SYSTEM   │      │   GIT REPO    │      │   ARGOCD      │
│               │─────►│               │◄─────│               │
│ • Build       │  3.  │ • deployment  │  5.  │ • Monitors    │
│ • Test        │ Clone│   .yaml       │ Poll │ • Compares    │
│ • Push Image  │      │ • values.yaml │      │ • Syncs       │
└───────┬───────┘      └───────────────┘      └───────────────┘
        │                                              │
        │ 2. docker push                               │
        ▼                                              │
┌───────────────┐                                      │
│   CONTAINER   │                                      │
│   REGISTRY    │◄─────────────────────────────────────┘
│               │         7. kubectl apply
│  myapp:v1.2.3 │
└───────────────┘


ПОШАГОВО:
─────────
1. Developer пушит код в app repository
2. CI собирает образ и пушит в registry (myapp:abc123)
3. CI клонирует config repository
4. CI обновляет image tag в манифестах, пушит
5. ArgoCD обнаруживает изменения (polling каждые 3 мин или webhook)
6. ArgoCD синхронизирует — применяет манифесты к кластеру
7. Kubernetes разворачивает новую версию
```

### Пример GitOps с ArgoCD

YAML

```
# ===== CONFIG REPOSITORY STRUCTURE =====
#
# k8s-configs/
# ├── apps/
# │   └── myapp/
# │       ├── base/
# │       │   ├── deployment.yaml
# │       │   ├── service.yaml
# │       │   └── kustomization.yaml
# │       └── overlays/
# │           ├── dev/
# │           │   └── kustomization.yaml
# │           ├── staging/
# │           │   └── kustomization.yaml
# │           └── production/
# │               └── kustomization.yaml
# └── argocd/
#     └── applications/
#         ├── myapp-dev.yaml
#         ├── myapp-staging.yaml
#         └── myapp-production.yaml


# apps/myapp/base/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:latest  # Будет переопределено в overlay
          ports:
            - containerPort: 8080


# apps/myapp/base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml


# apps/myapp/overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: production

resources:
  - ../../base

images:
  - name: myapp
    newName: registry.example.com/myapp
    newTag: abc123  # Этот тег обновляется CI

replicas:
  - name: myapp
    count: 5

patchesStrategicMerge:
  - deployment-patch.yaml


# apps/myapp/overlays/production/deployment-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
        - name: myapp
          resources:
            limits:
              cpu: 1000m
              memory: 1Gi
            requests:
              cpu: 500m
              memory: 512Mi


# argocd/applications/myapp-production.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-production
  namespace: argocd
spec:
  project: default
  
  source:
    repoURL: https://github.com/myorg/k8s-configs
    targetRevision: HEAD
    path: apps/myapp/overlays/production
  
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  
  syncPolicy:
    automated:
      selfHeal: true  # Автоматически исправлять drift
      prune: true     # Удалять ресурсы, которых нет в Git
    syncOptions:
      - CreateNamespace=true
```

YAML

```
# ===== CI PIPELINE ДЛЯ GITOPS =====

# .gitlab-ci.yml (в APPLICATION repository)
stages:
  - build
  - test
  - publish
  - update-manifests

variables:
  IMAGE_NAME: registry.example.com/myapp
  CONFIG_REPO: https://gitlab.com/myorg/k8s-configs.git

build:
  stage: build
  script:
    - docker build -t $IMAGE_NAME:$CI_COMMIT_SHORT_SHA .

test:
  stage: test
  script:
    - docker run $IMAGE_NAME:$CI_COMMIT_SHORT_SHA npm test

publish:
  stage: publish
  script:
    - docker push $IMAGE_NAME:$CI_COMMIT_SHORT_SHA
  only:
    - main

# Обновление манифестов в config repository
update-manifests:
  stage: update-manifests
  image: alpine/git
  before_script:
    - apk add --no-cache curl
    - curl -L https://github.com/mikefarah/yq/releases/download/v4.35.1/yq_linux_amd64 -o /usr/local/bin/yq
    - chmod +x /usr/local/bin/yq
  script:
    # Клонируем config repo
    - git clone https://oauth2:${CONFIG_REPO_TOKEN}@gitlab.com/myorg/k8s-configs.git
    - cd k8s-configs
    
    # Обновляем image tag
    - |
      cd apps/myapp/overlays/production
      kustomize edit set image myapp=$IMAGE_NAME:$CI_COMMIT_SHORT_SHA
    
    # Коммитим и пушим
    - git config user.name "GitLab CI"
    - git config user.email "ci@example.com"
    - git add .
    - git commit -m "Update myapp to $CI_COMMIT_SHORT_SHA"
    - git push origin main
  only:
    - main
  
  # ArgoCD обнаружит изменения и задеплоит автоматически
```

---

## 8. Версионный контроль схемы базы данных

### Проблемы без версионирования миграций

text

```
╔═══════════════════════════════════════════════════════════════════╗
║          ПРОБЛЕМЫ БЕЗ ВЕРСИОНИРОВАНИЯ МИГРАЦИЙ                     ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  ❌ "Какая версия схемы на production?"                           ║
║  ❌ "Кто и когда добавил эту колонку?"                            ║
║  ❌ "Почему на dev и prod разная схема?"                          ║
║  ❌ "Как откатить последнее изменение схемы?"                     ║
║  ❌ "Merge conflict в SQL скриптах"                               ║
║  ❌ "Ручное применение ALTER TABLE на каждом окружении"           ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Инструменты миграций

text

```
┌─────────────────────────────────────────────────────────────────────┐
│                   ИНСТРУМЕНТЫ МИГРАЦИЙ БД                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  FLYWAY (Java-based, но работает с любым приложением)               │
│  ─────────────────────────────────────────────────────              │
│  • SQL миграции (V1__name.sql, V2__name.sql)                        │
│  • Java миграции для сложной логики                                 │
│  • Версионирование и undo миграции                                  │
│  • CLI, Maven/Gradle плагины, Docker                                │
│  • Поддержка: PostgreSQL, MySQL, Oracle, SQL Server, etc.           │
│                                                                      │
│  LIQUIBASE (XML/YAML/JSON/SQL)                                      │
│  ─────────────────────────────                                      │
│  • Changeset-based (более гибкий, чем версии)                       │
│  • Генерация rollback автоматически                                 │
│  • Diff между базами                                                │
│  • Генерация документации                                           │
│  • Enterprise features (Datical)                                    │
│                                                                      │
│  ALEMBIC (Python/SQLAlchemy)                                        │
│  ───────────────────────────                                        │
│  • Python миграции                                                  │
│  • Автогенерация миграций из моделей                                │
│  • Branching миграций                                               │
│  • Интеграция с SQLAlchemy ORM                                      │
│                                                                      │
│  GOOSE (Go)                                                         │
│  ─────────                                                          │
│  • SQL или Go миграции                                              │
│  • Простой CLI                                                      │
│  • Встраивание в Go приложения                                      │
│                                                                      │
│  KNEX.JS (Node.js)                                                  │
│  ─────────────────                                                  │
│  • JavaScript миграции                                              │
│  • Query builder                                                    │
│  • Seed data                                                        │
│                                                                      │
│  PRISMA MIGRATE (Node.js)                                           │
│  ─────────────────────────                                          │
│  • Декларативная схема                                              │
│  • Автогенерация миграций                                           │
│  • Type-safe ORM                                                    │
│                                                                      │
│  RAILS MIGRATIONS (Ruby)                                            │
│  ───────────────────────                                            │
│  • Ruby DSL                                                         │
│  • Встроены в Rails                                                 │
│  • Автогенерация rollback                                           │
│                                                                      │
│  DJANGO MIGRATIONS (Python)                                         │
│  ──────────────────────────                                         │
│  • Python миграции                                                  │
│  • Автогенерация из моделей                                         │
│  • Встроены в Django                                                │
│                                                                      │
│  DBMATE (Universal)                                                 │
│  ─────────────────                                                  │
│  • SQL миграции                                                     │
│  • Простой CLI                                                      │
│  • Не зависит от языка приложения                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Пример: Flyway

SQL

```
-- ===== СТРУКТУРА МИГРАЦИЙ FLYWAY =====
--
-- migrations/
-- ├── V1__create_users_table.sql
-- ├── V2__add_email_to_users.sql
-- ├── V3__create_orders_table.sql
-- ├── V4__add_indexes.sql
-- └── V5__add_full_name_to_users.sql


-- V1__create_users_table.sql
-- Имя файла: V{version}__{description}.sql
-- Версия: целое число или с точками (1, 1.1, 1.1.1)

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Индексы
CREATE INDEX idx_users_username ON users(username);


-- V2__add_email_to_users.sql
ALTER TABLE users ADD COLUMN email VARCHAR(255);


-- V3__create_orders_table.sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id),
    total DECIMAL(10,2) NOT NULL,
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);


-- V4__add_indexes.sql
-- Можно добавлять индексы отдельными миграциями
CREATE INDEX CONCURRENTLY idx_orders_created_at ON orders(created_at);


-- V5__add_full_name_to_users.sql
-- Expand-Contract: добавляем колонку
ALTER TABLE users ADD COLUMN full_name VARCHAR(255);

-- Бэкфилл (для небольших таблиц)
UPDATE users SET full_name = username WHERE full_name IS NULL;
```

YAML

```
# ===== FLYWAY В CI/CD =====

# docker-compose.yml для локальной разработки
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: myapp
      POSTGRES_PASSWORD: myapp
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  flyway:
    image: flyway/flyway:9
    command: migrate
    environment:
      FLYWAY_URL: jdbc:postgresql://postgres:5432/myapp
      FLYWAY_USER: myapp
      FLYWAY_PASSWORD: myapp
    volumes:
      - ./migrations:/flyway/sql
    depends_on:
      - postgres

volumes:
  postgres_data:


# .gitlab-ci.yml
stages:
  - migrate
  - deploy

variables:
  FLYWAY_URL: jdbc:postgresql://${DB_HOST}:5432/${DB_NAME}
  FLYWAY_USER: ${DB_USER}
  FLYWAY_PASSWORD: ${DB_PASSWORD}

# Validate — проверка миграций без применения
validate_migrations:
  stage: migrate
  image: flyway/flyway:9
  script:
    - flyway validate
  except:
    - main

# Migrate — применение миграций
migrate_database:
  stage: migrate
  image: flyway/flyway:9
  script:
    # Info — показать статус миграций
    - flyway info
    # Migrate — применить новые миграции
    - flyway migrate
    # Info — показать результат
    - flyway info
  only:
    - main
  environment:
    name: production

deploy:
  stage: deploy
  script:
    - kubectl set image deployment/myapp myapp=$IMAGE
  needs:
    - migrate_database
  only:
    - main
```

Bash

```
# ===== КОМАНДЫ FLYWAY =====

# Информация о статусе миграций
flyway info

# Пример вывода:
# +-----------+---------+----------------------------+------+---------------------+---------+
# | Category  | Version | Description                | Type | Installed On        | State   |
# +-----------+---------+----------------------------+------+---------------------+---------+
# | Versioned | 1       | create users table         | SQL  | 2024-01-15 10:00:00 | Success |
# | Versioned | 2       | add email to users         | SQL  | 2024-01-15 10:00:01 | Success |
# | Versioned | 3       | create orders table        | SQL  | 2024-01-15 10:00:02 | Success |
# | Versioned | 4       | add indexes                | SQL  |                     | Pending |
# +-----------+---------+----------------------------+------+---------------------+---------+

# Применить миграции
flyway migrate

# Проверить что миграции корректны
flyway validate

# Очистить БД (осторожно!)
flyway clean

# Создать baseline для существующей БД
flyway baseline -baselineVersion=1 -baselineDescription="Initial baseline"

# Восстановить таблицу истории
flyway repair

# Undo последней миграции (требует Flyway Teams)
flyway undo
```

### Пример: Alembic (Python)

Python

```
# ===== ALEMBIC SETUP =====

# alembic.ini (конфигурация)
[alembic]
script_location = migrations
sqlalchemy.url = postgresql://user:pass@localhost/myapp

# migrations/env.py
from alembic import context
from sqlalchemy import engine_from_config
from myapp.models import Base  # Ваши SQLAlchemy модели

def run_migrations_online():
    connectable = engine_from_config(
        config.get_section(config.config_ini_section),
        prefix='sqlalchemy.',
    )
    
    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=Base.metadata,  # Для автогенерации
        )
        
        with context.begin_transaction():
            context.run_migrations()


# ===== СОЗДАНИЕ МИГРАЦИИ =====

# Автогенерация из моделей (сравнивает модели с БД)
# alembic revision --autogenerate -m "add users table"

# Ручное создание
# alembic revision -m "add indexes"


# migrations/versions/001_create_users_table.py
"""create users table

Revision ID: 001
Revises: 
Create Date: 2024-01-15 10:00:00
"""
from alembic import op
import sqlalchemy as sa

# Revision identifiers
revision = '001'
down_revision = None
branch_labels = None
depends_on = None


def upgrade():
    """Применение миграции."""
    op.create_table(
        'users',
        sa.Column('id', sa.Integer(), primary_key=True),
        sa.Column('username', sa.String(50), nullable=False, unique=True),
        sa.Column('email', sa.String(255)),
        sa.Column('created_at', sa.DateTime(), server_default=sa.func.now()),
    )
    
    # Индекс
    op.create_index('idx_users_username', 'users', ['username'])


def downgrade():
    """Откат миграции."""
    op.drop_index('idx_users_username')
    op.drop_table('users')


# migrations/versions/002_add_orders_table.py
"""add orders table

Revision ID: 002
Revises: 001
"""
revision = '002'
down_revision = '001'


def upgrade():
    op.create_table(
        'orders',
        sa.Column('id', sa.Integer(), primary_key=True),
        sa.Column('user_id', sa.Integer(), sa.ForeignKey('users.id'), nullable=False),
        sa.Column('total', sa.Numeric(10, 2), nullable=False),
        sa.Column('status', sa.String(20), server_default='pending'),
        sa.Column('created_at', sa.DateTime(), server_default=sa.func.now()),
    )
    
    op.create_index('idx_orders_user_id', 'orders', ['user_id'])
    op.create_index('idx_orders_status', 'orders', ['status'])


def downgrade():
    op.drop_index('idx_orders_status')
    op.drop_index('idx_orders_user_id')
    op.drop_table('orders')
```

Bash

```
# ===== КОМАНДЫ ALEMBIC =====

# Показать текущую ревизию
alembic current

# Показать историю миграций
alembic history

# Применить все миграции
alembic upgrade head

# Применить до конкретной ревизии
alembic upgrade 002

# Применить следующую миграцию
alembic upgrade +1

# Откатить последнюю миграцию
alembic downgrade -1

# Откатить все миграции
alembic downgrade base

# Сгенерировать миграцию из моделей
alembic revision --autogenerate -m "add new column"

# Создать пустую миграцию
alembic revision -m "custom migration"
```

### Пример: Liquibase

XML

```
<!-- ===== LIQUIBASE CHANGELOG ===== -->

<!-- db/changelog/db.changelog-master.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.20.xsd">

    <!-- Включение других changelog файлов -->
    <include file="db/changelog/changes/001-create-users.xml"/>
    <include file="db/changelog/changes/002-create-orders.xml"/>
    <include file="db/changelog/changes/003-add-indexes.xml"/>
    
</databaseChangeLog>


<!-- db/changelog/changes/001-create-users.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.20.xsd">

    <changeSet id="1" author="developer">
        <comment>Create users table</comment>
        
        <createTable tableName="users">
            <column name="id" type="serial" autoIncrement="true">
                <constraints primaryKey="true"/>
            </column>
            <column name="username" type="varchar(50)">
                <constraints nullable="false" unique="true"/>
            </column>
            <column name="email" type="varchar(255)"/>
            <column name="created_at" type="timestamp" defaultValueComputed="CURRENT_TIMESTAMP"/>
        </createTable>
        
        <!-- Rollback автоматически генерируется для createTable -->
    </changeSet>
    
    <changeSet id="2" author="developer">
        <comment>Add index on username</comment>
        
        <createIndex tableName="users" indexName="idx_users_username">
            <column name="username"/>
        </createIndex>
        
        <rollback>
            <dropIndex tableName="users" indexName="idx_users_username"/>
        </rollback>
    </changeSet>

</databaseChangeLog>
```

YAML

```
# Liquibase также поддерживает YAML формат
# db/changelog/changes/003-add-column.yaml

databaseChangeLog:
  - changeSet:
      id: 3
      author: developer
      comment: Add full_name column to users
      changes:
        - addColumn:
            tableName: users
            columns:
              - column:
                  name: full_name
                  type: varchar(255)
      rollback:
        - dropColumn:
            tableName: users
            columnName: full_name
  
  - changeSet:
      id: 4
      author: developer
      comment: Backfill full_name
      changes:
        - sql:
            sql: UPDATE users SET full_name = username WHERE full_name IS NULL
      # Нет rollback — это data migration
```

Bash

```
# ===== КОМАНДЫ LIQUIBASE =====

# Показать статус
liquibase status

# Применить миграции
liquibase update

# Откатить последний changeset
liquibase rollbackCount 1

# Откатить до определённого тега
liquibase rollback release-1.0

# Сгенерировать SQL без применения
liquibase updateSQL > changes.sql

# Сравнить две базы данных
liquibase diff --referenceUrl=jdbc:postgresql://localhost/dev

# Сгенерировать changelog из существующей БД
liquibase generateChangeLog --changeLogFile=changelog.xml

# Пометить тег (для rollback)
liquibase tag release-1.0
```

### Best Practices для миграций

YAML

```
# ===== BEST PRACTICES =====

# 1️⃣  ВЕРСИОНИРОВАНИЕ В GIT
# Миграции — часть кода, хранятся в Git
migrations/
├── V001__initial_schema.sql
├── V002__add_users_table.sql
└── V003__add_orders_table.sql


# 2️⃣  ОДНА МИГРАЦИЯ = ОДНА ЛОГИЧЕСКАЯ ЗАДАЧА
# ❌ V001__create_everything.sql (500 строк)
# ✅ V001__create_users.sql
# ✅ V002__create_orders.sql
# ✅ V003__create_products.sql


# 3️⃣  ИММУТАБЕЛЬНОСТЬ
# Никогда не изменяйте уже применённые миграции!
# Создавайте новую миграцию для исправления

# ❌ Изменить V001__create_users.sql после деплоя
# ✅ Создать V002__fix_users_table.sql


# 4️⃣  BACKWARD COMPATIBILITY
# Миграция должна работать с текущей и новой версией кода

# ❌ Сразу удалить колонку
# ✅ Expand-Contract pattern (см. выше)


# 5️⃣  ТЕСТИРОВАНИЕ МИГРАЦИЙ
# - Тесты на пустой БД (fresh install)
# - Тесты на существующей БД (upgrade)
# - Тесты rollback (если поддерживается)

test_migrations:
  script:
    # Fresh install
    - flyway migrate
    - flyway validate
    
    # Rollback test (если есть undo)
    # - flyway undo
    # - flyway migrate


# 6️⃣  ОТДЕЛЬНЫЕ МИГРАЦИИ ДЛЯ ДАННЫХ
# Структурные изменения отдельно от миграции данных

# V001__add_status_column.sql
ALTER TABLE orders ADD COLUMN status_new VARCHAR(20);

# V002__migrate_status_data.sql
UPDATE orders SET status_new = 
  CASE status 
    WHEN 0 THEN 'pending'
    WHEN 1 THEN 'completed'
  END;

# V003__finalize_status.sql
ALTER TABLE orders DROP COLUMN status;
ALTER TABLE orders RENAME COLUMN status_new TO status;


# 7️⃣  ИДЕМПОТЕНТНОСТЬ (где возможно)
# Миграция должна быть безопасна при повторном применении

# ✅ CREATE TABLE IF NOT EXISTS
# ✅ CREATE INDEX IF NOT EXISTS
# ✅ DROP TABLE IF EXISTS (осторожно!)


# 8️⃣  ДОКУМЕНТИРОВАНИЕ
# Описание в имени файла и/или комментариях

-- V015__add_order_tracking.sql
-- Добавляет поля для отслеживания статуса заказа
-- JIRA: PROJECT-123
-- Author: developer@example.com

ALTER TABLE orders ADD COLUMN tracking_number VARCHAR(100);
ALTER TABLE orders ADD COLUMN shipped_at TIMESTAMP;
```

---

## Полная шпаргалка

Bash

```
# ===== CI СТАДИИ =====
# 1. Checkout      — получение кода
# 2. Dependencies  — установка зависимостей
# 3. Lint          — проверка стиля (ESLint, Pylint)
# 4. Security      — SAST, dependency scan (Snyk, Semgrep)
# 5. Unit Tests    — модульные тесты (Jest, Pytest)
# 6. Build         — сборка приложения
# 7. Integration   — интеграционные тесты
# 8. Build Image   — сборка Docker образа
# 9. Scan Image    — проверка образа (Trivy)
# 10. Push Image   — публикация в registry


# ===== CD СТАДИИ =====
# 1. Deploy Staging  — деплой на staging
# 2. E2E Tests       — end-to-end тесты
# 3. Performance     — нагрузочные тесты
# 4. Approval        — ручное подтверждение (Continuous Delivery)
# 5. Deploy Prod     — деплой на production
# 6. Monitor         — мониторинг и откат


# ===== СТРАТЕГИИ ДЕПЛОЯ =====
# Recreate       — остановить старое, запустить новое (downtime)
# Rolling Update — постепенная замена (стандарт K8s)
# Blue-Green     — переключение между окружениями
# Canary         — постепенное увеличение трафика
# A/B Testing    — разные версии для разных сегментов
# Shadow         — копия трафика на новую версию


# ===== ZERO DOWNTIME ТРЕБОВАНИЯ =====
# • Backward compatible API
# • Backward compatible DB migrations
# • Health checks (liveness, readiness, startup)
# • Graceful shutdown
# • Минимум 2 реплики
# • PodDisruptionBudget


# ===== ИНСТРУМЕНТЫ МИГРАЦИЙ БД =====
# Flyway     — SQL миграции, Java
# Liquibase  — XML/YAML/SQL, rollback
# Alembic    — Python, SQLAlchemy
# Goose      — Go
# Knex.js    — Node.js
# Prisma     — Node.js, ORM
# Django     — Python, встроенные
# Rails      — Ruby, встроенные


# ===== GITOPS =====
# ArgoCD — самый популярный
# Flux   — модульный, CNCF
# 
# Принципы:
# 1. Декларативность
# 2. Версионирование в Git
# 3. Автоматическое применение
# 4. Непрерывная сверка (reconciliation)
```
## 9. Полные примеры CI/CD Pipeline

### Полный пример: GitHub Actions

YAML

```
# .github/workflows/ci-cd.yml
#
# Полный CI/CD pipeline для Node.js приложения

name: CI/CD Pipeline

# Триггеры запуска
on:
  push:
    branches:
      - main
      - develop
  pull_request:
    branches:
      - main

# Переменные окружения для всего workflow
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
  NODE_VERSION: '20'

# Отмена предыдущих запусков при новом push
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  # ═══════════════════════════════════════════════════════════
  # СТАДИЯ 1: УСТАНОВКА ЗАВИСИМОСТЕЙ
  # ═══════════════════════════════════════════════════════════
  install:
    name: 📦 Install Dependencies
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      # Кэшируем node_modules для следующих jobs
      - name: Cache node_modules
        uses: actions/cache@v4
        with:
          path: node_modules
          key: modules-${{ hashFiles('package-lock.json') }}

  # ═══════════════════════════════════════════════════════════
  # СТАДИЯ 2: ПРОВЕРКИ КАЧЕСТВА КОДА
  # ═══════════════════════════════════════════════════════════
  lint:
    name: 🔍 Lint
    needs: install
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      
      - name: Restore node_modules
        uses: actions/cache@v4
        with:
          path: node_modules
          key: modules-${{ hashFiles('package-lock.json') }}
      
      - name: Run ESLint
        run: npm run lint
      
      - name: Run Prettier check
        run: npm run format:check
      
      - name: Run TypeScript check
        run: npm run type-check

  # ═══════════════════════════════════════════════════════════
  # СТАДИЯ 3: ПРОВЕРКА БЕЗОПАСНОСТИ
  # ═══════════════════════════════════════════════════════════
  security:
    name: 🔒 Security Scan
    needs: install
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      
      - name: Restore node_modules
        uses: actions/cache@v4
        with:
          path: node_modules
          key: modules-${{ hashFiles('package-lock.json') }}
      
      # Проверка зависимостей на уязвимости
      - name: Run npm audit
        run: npm audit --audit-level=high
        continue-on-error: true
      
      # SAST сканирование с Semgrep
      - name: Run Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/secrets
            p/nodejs
      
      # Поиск секретов в коде
      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  # ═══════════════════════════════════════════════════════════
  # СТАДИЯ 4: ЮНИТ ТЕСТЫ
  # ═══════════════════════════════════════════════════════════
  unit-tests:
    name: 🧪 Unit Tests
    needs: install
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      
      - name: Restore node_modules
        uses: actions/cache@v4
        with:
          path: node_modules
          key: modules-${{ hashFiles('package-lock.json') }}
      
      - name: Run unit tests
        run: npm run test:unit -- --coverage --reporters=default --reporters=jest-junit
        env:
          JEST_JUNIT_OUTPUT_DIR: ./reports
      
      # Публикация результатов тестов
      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: reports/
      
      # Публикация coverage
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/lcov.info
          fail_ci_if_error: true

  # ═══════════════════════════════════════════════════════════
  # СТАДИЯ 5: ИНТЕГРАЦИОННЫЕ ТЕСТЫ
  # ═══════════════════════════════════════════════════════════
  integration-tests:
    name: 🔗 Integration Tests
    needs: [lint, unit-tests]
    runs-on: ubuntu-latest
    
    # Сервисы для тестов
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      
      redis:
        image: redis:7
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      
      - name: Restore node_modules
        uses: actions/cache@v4
        with:
          path: node_modules
          key: modules-${{ hashFiles('package-lock.json') }}
      
      - name: Run database migrations
        run: npm run db:migrate
        env:
          DATABASE_URL: postgres://test:test@localhost:5432/test
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgres://test:test@localhost:5432/test
          REDIS_URL: redis://localhost:6379

  # ═══════════════════════════════════════════════════════════
  # СТАДИЯ 6: СБОРКА
  # ═══════════════════════════════════════════════════════════
  build:
    name: 🏗️ Build
    needs: [lint, unit-tests]
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      
      - name: Restore node_modules
        uses: actions/cache@v4
        with:
          path: node_modules
          key: modules-${{ hashFiles('package-lock.json') }}
      
      - name: Build application
        run: npm run build
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/
          retention-days: 7

  # ═══════════════════════════════════════════════════════════
  # СТАДИЯ 7: СБОРКА DOCKER ОБРАЗА
  # ═══════════════════════════════════════════════════════════
  build-image:
    name: 🐳 Build Docker Image
    needs: [build, integration-tests, security]
    runs-on: ubuntu-latest
    
    # Только для main и develop веток
    if: github.event_name == 'push'
    
    permissions:
      contents: read
      packages: write
    
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      image-digest: ${{ steps.build.outputs.digest }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: build
          path: dist/
      
      # Настройка Docker Buildx для multi-platform builds
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      # Логин в GitHub Container Registry
      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      # Генерация тегов и labels
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-
            type=raw,value=latest,enable={{is_default_branch}}
      
      # Сборка и push образа
      - name: Build and push Docker image
        id: build
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          platforms: linux/amd64,linux/arm64

  # ═══════════════════════════════════════════════════════════
  # СТАДИЯ 8: СКАНИРОВАНИЕ ОБРАЗА
  # ═══════════════════════════════════════════════════════════
  scan-image:
    name: 🔍 Scan Docker Image
    needs: build-image
    runs-on: ubuntu-latest
    
    steps:
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.ref_name }}-${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
      
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'

  # ═══════════════════════════════════════════════════════════
  # СТАДИЯ 9: ДЕПЛОЙ НА STAGING
  # ═══════════════════════════════════════════════════════════
  deploy-staging:
    name: 🚀 Deploy to Staging
    needs: [build-image, scan-image]
    runs-on: ubuntu-latest
    
    # Только для main ветки
    if: github.ref == 'refs/heads/main'
    
    environment:
      name: staging
      url: https://staging.example.com
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      # Настройка kubectl
      - name: Configure kubectl
        uses: azure/k8s-set-context@v4
        with:
          kubeconfig: ${{ secrets.KUBE_CONFIG_STAGING }}
      
      # Деплой через Helm
      - name: Deploy with Helm
        run: |
          helm upgrade --install myapp ./helm/myapp \
            --namespace staging \
            --create-namespace \
            --set image.repository=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }} \
            --set image.tag=${{ github.ref_name }}-${{ github.sha }} \
            --set environment=staging \
            --wait \
            --timeout 10m
      
      # Проверка деплоя
      - name: Verify deployment
        run: |
          kubectl rollout status deployment/myapp -n staging --timeout=5m
          kubectl get pods -n staging -l app=myapp

  # ═══════════════════════════════════════════════════════════
  # СТАДИЯ 10: E2E ТЕСТЫ НА STAGING
  # ═══════════════════════════════════════════════════════════
  e2e-tests:
    name: 🎭 E2E Tests
    needs: deploy-staging
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      
      - name: Install Playwright
        run: |
          npm ci
          npx playwright install --with-deps chromium
      
      - name: Run E2E tests
        run: npm run test:e2e
        env:
          BASE_URL: https://staging.example.com
      
      - name: Upload E2E test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/

  # ═══════════════════════════════════════════════════════════
  # СТАДИЯ 11: ДЕПЛОЙ НА PRODUCTION
  # ═══════════════════════════════════════════════════════════
  deploy-production:
    name: 🚀 Deploy to Production
    needs: e2e-tests
    runs-on: ubuntu-latest
    
    # Требует ручного подтверждения
    environment:
      name: production
      url: https://example.com
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Configure kubectl
        uses: azure/k8s-set-context@v4
        with:
          kubeconfig: ${{ secrets.KUBE_CONFIG_PRODUCTION }}
      
      # Деплой с Canary стратегией через Argo Rollouts
      - name: Deploy with Helm (Canary)
        run: |
          helm upgrade --install myapp ./helm/myapp \
            --namespace production \
            --set image.repository=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }} \
            --set image.tag=${{ github.ref_name }}-${{ github.sha }} \
            --set environment=production \
            --set rollout.strategy=canary \
            --wait \
            --timeout 15m
      
      - name: Verify deployment
        run: |
          kubectl argo rollouts status myapp -n production --timeout=10m
      
      # Уведомление в Slack
      - name: Notify Slack
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "✅ Deployed to production: ${{ github.sha }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### Полный пример: GitLab CI

YAML

```
# .gitlab-ci.yml
#
# Полный CI/CD pipeline для Python/Django приложения

# Глобальные настройки
default:
  image: python:3.11-slim
  
  # Retry при сетевых ошибках
  retry:
    max: 2
    when:
      - runner_system_failure
      - stuck_or_timeout_failure

# Переменные
variables:
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.pip-cache"
  DOCKER_HOST: tcp://docker:2376
  DOCKER_TLS_CERTDIR: "/certs"
  DOCKER_DRIVER: overlay2
  IMAGE_NAME: $CI_REGISTRY_IMAGE
  POSTGRES_DB: test
  POSTGRES_USER: test
  POSTGRES_PASSWORD: test

# Стадии
stages:
  - install
  - validate
  - test
  - build
  - security
  - deploy-staging
  - integration-tests
  - deploy-production

# Кэширование
.pip_cache: &pip_cache
  cache:
    key:
      files:
        - requirements.txt
        - requirements-dev.txt
    paths:
      - .pip-cache/
      - venv/
    policy: pull-push

# ═══════════════════════════════════════════════════════════
# СТАДИЯ: УСТАНОВКА ЗАВИСИМОСТЕЙ
# ═══════════════════════════════════════════════════════════
install:
  stage: install
  <<: *pip_cache
  script:
    - python -m venv venv
    - source venv/bin/activate
    - pip install --upgrade pip
    - pip install -r requirements.txt -r requirements-dev.txt
  artifacts:
    paths:
      - venv/
    expire_in: 1 hour

# ═══════════════════════════════════════════════════════════
# СТАДИЯ: ВАЛИДАЦИЯ
# ═══════════════════════════════════════════════════════════
lint:
  stage: validate
  <<: *pip_cache
  needs:
    - install
  script:
    - source venv/bin/activate
    - flake8 src/
    - black --check src/
    - isort --check-only src/
    - mypy src/

security-sast:
  stage: validate
  <<: *pip_cache
  needs:
    - install
  script:
    - source venv/bin/activate
    - bandit -r src/ -f json -o bandit-report.json || true
    - safety check -r requirements.txt --json > safety-report.json || true
  artifacts:
    reports:
      sast: bandit-report.json
    paths:
      - bandit-report.json
      - safety-report.json
  allow_failure: true

# ═══════════════════════════════════════════════════════════
# СТАДИЯ: ТЕСТЫ
# ═══════════════════════════════════════════════════════════
unit-tests:
  stage: test
  <<: *pip_cache
  needs:
    - install
  script:
    - source venv/bin/activate
    - pytest tests/unit/ \
        --cov=src \
        --cov-report=xml:coverage.xml \
        --cov-report=html:coverage_html \
        --junitxml=junit.xml \
        -v
  coverage: '/TOTAL.*\s+(\d+%)/'
  artifacts:
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
    paths:
      - coverage_html/
    expire_in: 7 days

integration-tests:
  stage: test
  <<: *pip_cache
  needs:
    - install
    - lint
  services:
    - postgres:15
    - redis:7
  variables:
    DATABASE_URL: postgres://test:test@postgres:5432/test
    REDIS_URL: redis://redis:6379
  script:
    - source venv/bin/activate
    # Ждём пока сервисы поднимутся
    - |
      for i in $(seq 1 30); do
        pg_isready -h postgres -p 5432 -U test && break
        sleep 1
      done
    # Миграции
    - python manage.py migrate
    # Тесты
    - pytest tests/integration/ -v
  artifacts:
    reports:
      junit: integration-junit.xml

# ═══════════════════════════════════════════════════════════
# СТАДИЯ: СБОРКА
# ═══════════════════════════════════════════════════════════
build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  needs:
    - unit-tests
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    # Сборка образа
    - |
      docker build \
        --cache-from $IMAGE_NAME:latest \
        --build-arg BUILDKIT_INLINE_CACHE=1 \
        -t $IMAGE_NAME:$CI_COMMIT_SHORT_SHA \
        -t $IMAGE_NAME:$CI_COMMIT_REF_SLUG \
        -t $IMAGE_NAME:latest \
        .
    # Push
    - docker push $IMAGE_NAME:$CI_COMMIT_SHORT_SHA
    - docker push $IMAGE_NAME:$CI_COMMIT_REF_SLUG
    - |
      if [ "$CI_COMMIT_BRANCH" == "main" ]; then
        docker push $IMAGE_NAME:latest
      fi
  rules:
    - if: $CI_COMMIT_BRANCH == "main" || $CI_COMMIT_BRANCH == "develop"
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

# ═══════════════════════════════════════════════════════════
# СТАДИЯ: БЕЗОПАСНОСТЬ
# ═══════════════════════════════════════════════════════════
container-scan:
  stage: security
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  needs:
    - build
  script:
    - trivy image 
        --exit-code 0 
        --severity HIGH,CRITICAL 
        --format json 
        --output trivy-report.json 
        $IMAGE_NAME:$CI_COMMIT_SHORT_SHA
  artifacts:
    reports:
      container_scanning: trivy-report.json
  rules:
    - if: $CI_COMMIT_BRANCH == "main" || $CI_COMMIT_BRANCH == "develop"

# ═══════════════════════════════════════════════════════════
# СТАДИЯ: ДЕПЛОЙ НА STAGING
# ═══════════════════════════════════════════════════════════
.deploy_template: &deploy_template
  image: 
    name: alpine/helm:3.13
    entrypoint: [""]
  before_script:
    - apk add --no-cache curl
    - curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    - chmod +x kubectl
    - mv kubectl /usr/local/bin/
    - mkdir -p ~/.kube
    - echo "$KUBECONFIG_CONTENT" > ~/.kube/config
  script:
    - |
      helm upgrade --install myapp ./helm/myapp \
        --namespace $ENVIRONMENT \
        --create-namespace \
        -f ./helm/myapp/values-${ENVIRONMENT}.yaml \
        --set image.repository=$IMAGE_NAME \
        --set image.tag=$CI_COMMIT_SHORT_SHA \
        --set ingress.hosts[0].host=${APP_HOST} \
        --wait \
        --timeout 10m
    - kubectl rollout status deployment/myapp -n $ENVIRONMENT --timeout=5m

deploy-staging:
  stage: deploy-staging
  <<: *deploy_template
  variables:
    ENVIRONMENT: staging
    APP_HOST: staging.example.com
    KUBECONFIG_CONTENT: $KUBE_CONFIG_STAGING
  needs:
    - build
    - container-scan
  environment:
    name: staging
    url: https://staging.example.com
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

# ═══════════════════════════════════════════════════════════
# СТАДИЯ: E2E ТЕСТЫ
# ═══════════════════════════════════════════════════════════
e2e-tests:
  stage: integration-tests
  image: mcr.microsoft.com/playwright:v1.40.0-focal
  needs:
    - deploy-staging
  script:
    - npm ci
    - npx playwright test
  variables:
    BASE_URL: https://staging.example.com
  artifacts:
    when: always
    paths:
      - playwright-report/
    expire_in: 7 days
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

performance-tests:
  stage: integration-tests
  image: grafana/k6:latest
  needs:
    - deploy-staging
  script:
    - k6 run 
        --out json=results.json 
        tests/performance/load-test.js
  artifacts:
    paths:
      - results.json
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  allow_failure: true

# ═══════════════════════════════════════════════════════════
# СТАДИЯ: ДЕПЛОЙ НА PRODUCTION
# ═══════════════════════════════════════════════════════════
deploy-production:
  stage: deploy-production
  <<: *deploy_template
  variables:
    ENVIRONMENT: production
    APP_HOST: example.com
    KUBECONFIG_CONTENT: $KUBE_CONFIG_PRODUCTION
  needs:
    - e2e-tests
  environment:
    name: production
    url: https://example.com
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual
  
  # Уведомление в Slack после деплоя
  after_script:
    - |
      curl -X POST $SLACK_WEBHOOK \
        -H "Content-Type: application/json" \
        -d "{
          \"text\": \"✅ Deployed to production\",
          \"attachments\": [{
            \"color\": \"good\",
            \"fields\": [
              {\"title\": \"Version\", \"value\": \"$CI_COMMIT_SHORT_SHA\", \"short\": true},
              {\"title\": \"Author\", \"value\": \"$GITLAB_USER_NAME\", \"short\": true},
              {\"title\": \"Pipeline\", \"value\": \"$CI_PIPELINE_URL\", \"short\": false}
            ]
          }]
        }"

# Откат (ручной)
rollback-production:
  stage: deploy-production
  image: bitnami/kubectl:latest
  script:
    - mkdir -p ~/.kube
    - echo "$KUBE_CONFIG_PRODUCTION" > ~/.kube/config
    - kubectl rollout undo deployment/myapp -n production
    - kubectl rollout status deployment/myapp -n production --timeout=5m
  environment:
    name: production
    action: stop
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual
  allow_failure: true
```

---

## 10. Dockerfile Best Practices

Dockerfile

```
# ═══════════════════════════════════════════════════════════
# MULTI-STAGE BUILD ДЛЯ NODE.JS
# ═══════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────
# Stage 1: Dependencies
# ─────────────────────────────────────────────────────────────
FROM node:20-alpine AS deps

# Рабочая директория
WORKDIR /app

# Копируем только файлы зависимостей (для кэширования)
COPY package.json package-lock.json ./

# Устанавливаем зависимости
# --frozen-lockfile гарантирует точные версии из lock файла
RUN npm ci --frozen-lockfile

# ─────────────────────────────────────────────────────────────
# Stage 2: Builder
# ─────────────────────────────────────────────────────────────
FROM node:20-alpine AS builder

WORKDIR /app

# Копируем зависимости из предыдущего stage
COPY --from=deps /app/node_modules ./node_modules

# Копируем исходный код
COPY . .

# Собираем приложение
RUN npm run build

# Удаляем dev-зависимости
RUN npm prune --production

# ─────────────────────────────────────────────────────────────
# Stage 3: Production
# ─────────────────────────────────────────────────────────────
FROM node:20-alpine AS production

# Метаданные
LABEL maintainer="team@example.com"
LABEL version="1.0.0"
LABEL description="My Node.js Application"

# Безопасность: создаём непривилегированного пользователя
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

WORKDIR /app

# Переменные окружения
ENV NODE_ENV=production
ENV PORT=3000

# Копируем только необходимое для запуска
COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nextjs:nodejs /app/package.json ./

# Переключаемся на непривилегированного пользователя
USER nextjs

# Открываем порт
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# Запуск приложения
CMD ["node", "dist/main.js"]


# ═══════════════════════════════════════════════════════════
# MULTI-STAGE BUILD ДЛЯ PYTHON
# ═══════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────
# Stage 1: Builder
# ─────────────────────────────────────────────────────────────
FROM python:3.11-slim AS builder

# Переменные для Python
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

WORKDIR /app

# Установка системных зависимостей для сборки
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Создаём виртуальное окружение
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Копируем и устанавливаем зависимости
COPY requirements.txt .
RUN pip install --upgrade pip && \
    pip install -r requirements.txt

# ─────────────────────────────────────────────────────────────
# Stage 2: Production
# ─────────────────────────────────────────────────────────────
FROM python:3.11-slim AS production

# Переменные
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PATH="/opt/venv/bin:$PATH"

WORKDIR /app

# Только runtime зависимости
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 \
    curl \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

# Создаём непривилегированного пользователя
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Копируем виртуальное окружение
COPY --from=builder /opt/venv /opt/venv

# Копируем код приложения
COPY --chown=appuser:appuser . .

# Переключаемся на непривилегированного пользователя
USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=10s --start-period=10s --retries=3 \
    CMD curl --fail http://localhost:8000/health || exit 1

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "app.wsgi:application"]


# ═══════════════════════════════════════════════════════════
# MULTI-STAGE BUILD ДЛЯ GO
# ═══════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────
# Stage 1: Builder
# ─────────────────────────────────────────────────────────────
FROM golang:1.21-alpine AS builder

# Зависимости для сборки
RUN apk add --no-cache git ca-certificates tzdata

WORKDIR /app

# Копируем go.mod и go.sum для кэширования зависимостей
COPY go.mod go.sum ./
RUN go mod download

# Копируем исходный код
COPY . .

# Сборка статически линкованного бинарника
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags="-w -s -X main.version=${VERSION}" \
    -o /app/server \
    ./cmd/server

# ─────────────────────────────────────────────────────────────
# Stage 2: Production (scratch — минимальный образ)
# ─────────────────────────────────────────────────────────────
FROM scratch AS production

# Копируем сертификаты для HTTPS
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Копируем timezone data
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo

# Копируем бинарник
COPY --from=builder /app/server /server

# Порт
EXPOSE 8080

# Запуск
ENTRYPOINT ["/server"]
```

---

## 11. Мониторинг CI/CD

### Метрики CI/CD

YAML

```
# ===== КЛЮЧЕВЫЕ МЕТРИКИ CI/CD =====

# 1️⃣  LEAD TIME (время от коммита до production)
# Цель: < 1 час для высокопроизводительных команд
# 
# Как измерить:
# - Время от git push до успешного деплоя на prod
# - Включает: CI + code review + CD

# 2️⃣  DEPLOYMENT FREQUENCY (частота деплоев)
# Цель: несколько раз в день
#
# Как измерить:
# - Количество успешных деплоев на production / день

# 3️⃣  CHANGE FAILURE RATE (процент неудачных изменений)
# Цель: < 15%
#
# Как измерить:
# - (Количество откатов + hotfixes) / Общее количество деплоев

# 4️⃣  MEAN TIME TO RECOVERY (среднее время восстановления)
# Цель: < 1 час
#
# Как измерить:
# - Время от обнаружения проблемы до восстановления

# 5️⃣  BUILD TIME (время сборки)
# Цель: < 10 минут
#
# Как измерить:
# - Средняя продолжительность CI pipeline

# 6️⃣  TEST COVERAGE (покрытие тестами)
# Цель: > 80%
#
# Как измерить:
# - Процент кода, покрытого тестами

# 7️⃣  PIPELINE SUCCESS RATE (успешность pipeline)
# Цель: > 95%
#
# Как измерить:
# - Успешные pipeline / Все pipeline
```

### Prometheus метрики для CI/CD

YAML

```
# prometheus-rules.yml

groups:
  - name: ci-cd-metrics
    rules:
      # Build duration histogram
      - record: ci_build_duration_seconds
        expr: |
          histogram_quantile(0.95, 
            sum(rate(gitlab_ci_pipeline_duration_seconds_bucket[1h])) by (le, project)
          )
      
      # Deployment frequency
      - record: deployment_frequency_daily
        expr: |
          sum(increase(argocd_app_sync_total{phase="Succeeded"}[24h])) by (name)
      
      # Failed deployments
      - record: deployment_failure_rate
        expr: |
          sum(rate(argocd_app_sync_total{phase="Failed"}[24h])) 
          / 
          sum(rate(argocd_app_sync_total[24h]))

  - name: ci-cd-alerts
    rules:
      # Alert: Pipeline слишком долгий
      - alert: CIPipelineTooSlow
        expr: ci_build_duration_seconds > 1800  # 30 минут
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "CI pipeline is too slow"
          description: "Pipeline for {{ $labels.project }} is taking more than 30 minutes"
      
      # Alert: Высокий процент неудачных pipeline
      - alert: HighPipelineFailureRate
        expr: |
          sum(rate(gitlab_ci_pipeline_status{status="failed"}[1h]))
          /
          sum(rate(gitlab_ci_pipeline_status[1h])) > 0.2
        for: 15m
        labels:
          severity: critical
        annotations:
          summary: "High pipeline failure rate"
          description: "More than 20% of pipelines are failing"
      
      # Alert: Деплой не происходит долго
      - alert: NoRecentDeployments
        expr: |
          time() - max(argocd_app_sync_total{phase="Succeeded"}) > 86400
        labels:
          severity: warning
        annotations:
          summary: "No deployments in 24 hours"
```

### Grafana Dashboard для CI/CD

JSON

```
{
  "dashboard": {
    "title": "CI/CD Metrics",
    "panels": [
      {
        "title": "Deployment Frequency",
        "type": "stat",
        "targets": [
          {
            "expr": "sum(increase(argocd_app_sync_total{phase=\"Succeeded\"}[24h]))",
            "legendFormat": "Deployments / day"
          }
        ]
      },
      {
        "title": "Lead Time (95th percentile)",
        "type": "gauge",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, sum(rate(ci_lead_time_seconds_bucket[24h])) by (le))",
            "legendFormat": "Lead Time"
          }
        ]
      },
      {
        "title": "Pipeline Duration",
        "type": "graph",
        "targets": [
          {
            "expr": "avg(gitlab_ci_pipeline_duration_seconds) by (project)",
            "legendFormat": "{{ project }}"
          }
        ]
      },
      {
        "title": "Pipeline Success Rate",
        "type": "gauge",
        "targets": [
          {
            "expr": "sum(rate(gitlab_ci_pipeline_status{status=\"success\"}[24h])) / sum(rate(gitlab_ci_pipeline_status[24h])) * 100",
            "legendFormat": "Success Rate %"
          }
        ]
      }
    ]
  }
}
```

---

## 12. Полная шпаргалка CI/CD

Bash

```
# ═══════════════════════════════════════════════════════════
#                    CI/CD CHEATSHEET
# ═══════════════════════════════════════════════════════════

# ===== CI СТАДИИ =====
# 1. checkout       → Получение кода
# 2. install        → Установка зависимостей
# 3. lint           → Проверка стиля кода
# 4. security       → Анализ безопасности (SAST)
# 5. unit-tests     → Модульные тесты
# 6. build          → Сборка приложения
# 7. integration    → Интеграционные тесты
# 8. package        → Сборка Docker образа
# 9. scan           → Сканирование образа
# 10. publish       → Push в registry

# ===== CD СТАДИИ =====
# 1. deploy-staging  → Деплой на staging
# 2. e2e-tests       → End-to-end тесты
# 3. performance     → Нагрузочные тесты
# 4. approval        → Ручное подтверждение
# 5. deploy-prod     → Деплой на production
# 6. smoke-tests     → Проверка после деплоя
# 7. monitoring      → Мониторинг и алерты


# ===== ИНСТРУМЕНТЫ КАЧЕСТВА КОДА =====
# ESLint        → JavaScript/TypeScript линтер
# Prettier      → Форматирование кода
# Pylint/Flake8 → Python линтер
# Black         → Python форматирование
# golangci-lint → Go мета-линтер
# SonarQube     → Комплексный анализ


# ===== ИНСТРУМЕНТЫ БЕЗОПАСНОСТИ =====
# Semgrep       → SAST (бесплатный, много правил)
# Snyk          → Dependency scanning
# Trivy         → Container scanning
# Gitleaks      → Secret detection
# Bandit        → Python security
# npm audit     → Node.js dependencies


# ===== СТРАТЕГИИ ДЕПЛОЯ =====
# Recreate      → Остановить → Запустить (downtime)
# Rolling       → Постепенная замена (стандарт K8s)
# Blue-Green    → Переключение окружений
# Canary        → Постепенное увеличение трафика
# A/B Testing   → Разные версии для сегментов


# ===== ZERO DOWNTIME ТРЕБОВАНИЯ =====
# • Backward compatible API/DB
# • Health checks (liveness, readiness)
# • Graceful shutdown
# • Минимум 2 реплики
# • PodDisruptionBudget
# • Expand-Contract миграции


# ===== ИНСТРУМЕНТЫ МИГРАЦИЙ БД =====
# Flyway        → SQL, Java-based
# Liquibase     → XML/YAML/SQL, rollback
# Alembic       → Python, SQLAlchemy
# Goose         → Go
# Prisma        → Node.js, ORM
# Knex.js       → Node.js


# ===== КОМАНДЫ FLYWAY =====
flyway info      # Статус миграций
flyway migrate   # Применить миграции
flyway validate  # Проверить миграции
flyway clean     # Очистить БД (осторожно!)
flyway baseline  # Создать baseline


# ===== GITOPS =====
# ArgoCD        → Самый популярный, Web UI
# Flux          → Модульный, CNCF
# 
# Принципы:
# 1. Декларативность
# 2. Версионирование в Git
# 3. Автоматическое применение
# 4. Непрерывная сверка


# ===== СЕКРЕТЫ =====
# CI/CD Variables    → GitLab/GitHub встроенные
# HashiCorp Vault    → Централизованное хранение
# AWS Secrets Manager → AWS native
# Sealed Secrets     → Encrypted в Git
# External Secrets   → Operator для K8s
# SOPS               → Encrypted files


# ===== КЛЮЧЕВЫЕ МЕТРИКИ =====
# Lead Time          → Время до production (< 1 час)
# Deployment Freq    → Частота деплоев (несколько/день)
# Change Failure Rate → Процент откатов (< 15%)
# MTTR               → Время восстановления (< 1 час)
# Build Time         → Время сборки (< 10 мин)
# Test Coverage      → Покрытие тестами (> 80%)


# ===== DOCKER BEST PRACTICES =====
# • Multi-stage builds
# • Минимальный базовый образ (alpine, distroless)
# • Non-root user
# • HEALTHCHECK
# • .dockerignore
# • Фиксированные версии зависимостей
# • Копировать только необходимое
```