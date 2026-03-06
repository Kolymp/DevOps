## 1. Multi-branch Pipeline

text

```
┌────────────────────────────────────────────────────────────┐
│              MULTI-BRANCH PIPELINE                         │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Multi-branch Pipeline — тип Job, который АВТОМАТИЧЕСКИ   │
│  создаёт отдельный Pipeline для КАЖДОЙ ветки репозитория  │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │  Git Repository                                      │ │
│  │  ├── main          ─────────►  Pipeline: main        │ │
│  │  ├── develop       ─────────►  Pipeline: develop     │ │
│  │  ├── feature/login ─────────►  Pipeline: feature/login│ │
│  │  ├── feature/cart  ─────────►  Pipeline: feature/cart│ │
│  │  ├── hotfix/bug123 ─────────►  Pipeline: hotfix/bug123│ │
│  │  └── PR #45        ─────────►  Pipeline: PR-45       │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
│  Jenkins СКАНИРУЕТ репозиторий и:                         │
│  • Находит ветки с Jenkinsfile                            │
│  • Создаёт Pipeline для каждой ветки                      │
│  • Автоматически удаляет при удалении ветки               │
│  • Обрабатывает Pull Request / Merge Request              │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Преимущества Multi-branch

text

```
┌────────────────────────────────────────────────────────────┐
│              ПРЕИМУЩЕСТВА                                  │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ✅ Автоматизация                                          │
│     • Не нужно создавать Job для каждой ветки вручную     │
│     • Новые ветки автоматически подхватываются            │
│     • Удалённые ветки автоматически удаляются             │
│                                                            │
│  ✅ CI для Pull Request                                    │
│     • Автоматическая проверка PR перед merge              │
│     • Статус билда отображается в GitHub/GitLab           │
│     • Можно запретить merge при failed билде              │
│                                                            │
│  ✅ Разные настройки для разных веток                      │
│     • main → deploy to production                         │
│     • develop → deploy to staging                         │
│     • feature/* → только тесты                            │
│                                                            │
│  ✅ Изоляция                                               │
│     • Каждая ветка — отдельный билд                       │
│     • Нет конфликтов между ветками                        │
│     • История билдов для каждой ветки                     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Создание Multi-branch Pipeline

text

```
┌────────────────────────────────────────────────────────────┐
│         СОЗДАНИЕ MULTI-BRANCH PIPELINE                     │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Jenkins → New Item                                       │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ Enter an item name: my-application                   │ │
│  │                                                      │ │
│  │ ○ Freestyle project                                  │ │
│  │ ○ Pipeline                                           │ │
│  │ ● Multibranch Pipeline   ← ВЫБРАТЬ                   │ │
│  │ ○ Folder                                             │ │
│  │ ○ Organization Folder                                │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
│  [OK]                                                     │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ Branch Sources                                       │ │
│  │ ─────────────────                                    │ │
│  │ Add source: ● GitHub                                 │ │
│  │             ○ GitLab                                 │ │
│  │             ○ Bitbucket                              │ │
│  │             ○ Git                                    │ │
│  │                                                      │ │
│  │ Credentials: github-token                            │ │
│  │ Repository URL: https://github.com/user/my-app       │ │
│  │                                                      │ │
│  │ Behaviours:                                          │ │
│  │   ☑ Discover branches                                │ │
│  │   ☑ Discover pull requests from origin               │ │
│  │   ☑ Discover pull requests from forks                │ │
│  │   ☐ Discover tags                                    │ │
│  │                                                      │ │
│  │ ─────────────────────────────────────────────────    │ │
│  │ Build Configuration                                  │ │
│  │ ───────────────────                                  │ │
│  │ Mode: by Jenkinsfile                                 │ │
│  │ Script Path: Jenkinsfile                             │ │
│  │                                                      │ │
│  │ ─────────────────────────────────────────────────    │ │
│  │ Scan Multibranch Pipeline Triggers                   │ │
│  │ ────────────────────────────────                     │ │
│  │ ☑ Periodically if not otherwise run                  │ │
│  │   Interval: 1 hour                                   │ │
│  │                                                      │ │
│  │ ☑ Scan by webhook                                    │ │
│  │   Trigger token: my-scan-token                       │ │
│  │                                                      │ │
│  │ ─────────────────────────────────────────────────    │ │
│  │ Orphaned Item Strategy                               │ │
│  │ ───────────────────────                              │ │
│  │ Days to keep old items: 30                           │ │
│  │ Max # of old items to keep: 10                       │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
│  [Save]                                                   │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Jenkinsfile для Multi-branch

groovy

```
// ═══════════════════════════════════════════════════════════
//          JENKINSFILE ДЛЯ MULTI-BRANCH PIPELINE
// ═══════════════════════════════════════════════════════════

pipeline {
    agent any
    
    environment {
        // Переменные автоматически доступные в Multi-branch:
        // BRANCH_NAME      — имя ветки (main, develop, feature/login)
        // CHANGE_ID        — номер PR (если это PR)
        // CHANGE_TITLE     — заголовок PR
        // CHANGE_AUTHOR    — автор PR
        // CHANGE_TARGET    — целевая ветка PR (main)
        // CHANGE_URL       — URL PR в GitHub
        
        APP_ENV = "${BRANCH_NAME == 'main' ? 'production' : 'staging'}"
    }
    
    stages {
        stage('Info') {
            steps {
                echo "Branch: ${BRANCH_NAME}"
                echo "Environment: ${APP_ENV}"
                
                script {
                    if (env.CHANGE_ID) {
                        echo "This is PR #${CHANGE_ID}"
                        echo "PR Title: ${CHANGE_TITLE}"
                        echo "PR Author: ${CHANGE_AUTHOR}"
                        echo "Target Branch: ${CHANGE_TARGET}"
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm ci && npm run build'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        
        // ───────────────────────────────────────────────────
        // Разные действия для разных веток
        // ───────────────────────────────────────────────────
        
        // Только для main
        stage('Deploy Production') {
            when {
                branch 'main'
            }
            steps {
                sh './deploy.sh production'
            }
        }
        
        // Только для develop
        stage('Deploy Staging') {
            when {
                branch 'develop'
            }
            steps {
                sh './deploy.sh staging'
            }
        }
        
        // Для feature/* веток
        stage('Deploy Preview') {
            when {
                branch pattern: 'feature/*', comparator: 'GLOB'
            }
            steps {
                sh "./deploy-preview.sh ${BRANCH_NAME}"
            }
        }
        
        // Только для Pull Request
        stage('PR Checks') {
            when {
                changeRequest()  // Это PR
            }
            steps {
                echo "Running additional checks for PR #${CHANGE_ID}"
                sh 'npm run lint'
                sh 'npm run security-check'
                
                // Добавить комментарий в PR
                script {
                    // Требует GitHub plugin
                    pullRequest.comment("Build #${BUILD_NUMBER} succeeded! ✅")
                }
            }
        }
        
        // НЕ для PR (только для веток)
        stage('Release') {
            when {
                not { changeRequest() }
                branch 'main'
            }
            steps {
                sh './release.sh'
            }
        }
    }
    
    post {
        success {
            script {
                if (env.CHANGE_ID) {
                    // Обновить статус в GitHub
                    githubNotify status: 'SUCCESS', description: 'Build passed'
                }
            }
        }
        failure {
            script {
                if (env.CHANGE_ID) {
                    githubNotify status: 'FAILURE', description: 'Build failed'
                }
            }
        }
    }
}
```

### when-условия для Multi-branch

groovy

```
// ═══════════════════════════════════════════════════════════
//          WHEN-УСЛОВИЯ ДЛЯ MULTI-BRANCH
// ═══════════════════════════════════════════════════════════

pipeline {
    agent any
    
    stages {
        // Конкретная ветка
        stage('Only Main') {
            when { branch 'main' }
            steps { echo 'Main branch' }
        }
        
        // Паттерн ветки (GLOB)
        stage('Feature Branches') {
            when { branch pattern: 'feature/*', comparator: 'GLOB' }
            steps { echo 'Feature branch' }
        }
        
        // Паттерн ветки (REGEXP)
        stage('Release Branches') {
            when { branch pattern: 'release/\\d+\\.\\d+', comparator: 'REGEXP' }
            steps { echo 'Release branch' }
        }
        
        // Pull Request
        stage('PR Only') {
            when { changeRequest() }
            steps { echo 'This is a Pull Request' }
        }
        
        // PR в конкретную ветку
        stage('PR to Main') {
            when { changeRequest target: 'main' }
            steps { echo 'PR targeting main' }
        }
        
        // НЕ PR
        stage('Not PR') {
            when { not { changeRequest() } }
            steps { echo 'This is NOT a PR' }
        }
        
        // Комбинация
        stage('Main or Develop, but not PR') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
                not { changeRequest() }
            }
            steps { echo 'Main or develop branch (not PR)' }
        }
        
        // Tag
        stage('Release Tag') {
            when { tag pattern: 'v\\d+\\.\\d+\\.\\d+', comparator: 'REGEXP' }
            steps { echo 'Version tag' }
        }
        
        // buildingTag() — любой тег
        stage('Any Tag') {
            when { buildingTag() }
            steps { echo 'Building a tag' }
        }
    }
}
```

---

## 2. Scripted vs Declarative — выбор стиля

text

```
┌────────────────────────────────────────────────────────────┐
│         SCRIPTED vs DECLARATIVE PIPELINE                   │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  DECLARATIVE ⭐ (рекомендуется)                            │
│  ──────────────────────────────                            │
│  pipeline {                                               │
│      agent any                                            │
│      stages {                                             │
│          stage('Build') {                                 │
│              steps { sh 'make' }                          │
│          }                                                │
│      }                                                    │
│  }                                                        │
│                                                            │
│  SCRIPTED                                                 │
│  ─────────                                                 │
│  node {                                                   │
│      stage('Build') {                                     │
│          sh 'make'                                        │
│      }                                                    │
│  }                                                        │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Сравнение

text

```
┌──────────────────────┬────────────────────┬─────────────────────┐
│      Критерий        │    Declarative     │     Scripted        │
├──────────────────────┼────────────────────┼─────────────────────┤
│ Синтаксис            │ Строгий, понятный  │ Свободный Groovy    │
│ Обучение             │ Легче              │ Требует Groovy      │
│ Валидация            │ Есть (до запуска)  │ Нет                 │
│ Blue Ocean UI        │ Полная поддержка   │ Ограниченная        │
│ Читаемость           │ Высокая            │ Зависит от автора   │
│ Гибкость             │ Ограниченная       │ Максимальная        │
│ Циклы/условия        │ Через script {}    │ Напрямую            │
│ Обработка ошибок     │ post { }           │ try/catch/finally   │
│ Повторное исп.       │ Shared Library     │ Shared Library      │
│ Отладка              │ Проще              │ Сложнее             │
│ Документация         │ Больше примеров    │ Меньше              │
└──────────────────────┴────────────────────┴─────────────────────┘
```

### Преимущества Declarative

groovy

```
// ═══════════════════════════════════════════════════════════
//              ПРЕИМУЩЕСТВА DECLARATIVE
// ═══════════════════════════════════════════════════════════

pipeline {
    agent any
    
    // ✅ Встроенные опции — понятно и компактно
    options {
        timeout(time: 1, unit: 'HOURS')
        timestamps()
        disableConcurrentBuilds()
    }
    
    // ✅ Декларативные триггеры
    triggers {
        cron('H 2 * * *')
    }
    
    // ✅ Параметры в одном месте
    parameters {
        string(name: 'VERSION', defaultValue: '1.0')
    }
    
    stages {
        stage('Build') {
            // ✅ when — декларативные условия
            when {
                branch 'main'
                not { changeRequest() }
            }
            steps {
                sh 'make build'
            }
        }
        
        // ✅ parallel — просто и понятно
        stage('Tests') {
            parallel {
                stage('Unit') { steps { sh 'make test-unit' } }
                stage('E2E') { steps { sh 'make test-e2e' } }
            }
        }
    }
    
    // ✅ post — гарантированное выполнение
    post {
        always { cleanWs() }
        success { echo 'Done!' }
        failure { slackSend message: 'Failed!' }
    }
}
```

### Преимущества Scripted

groovy

```
// ═══════════════════════════════════════════════════════════
//              ПРЕИМУЩЕСТВА SCRIPTED
// ═══════════════════════════════════════════════════════════

node {
    // ✅ Полная мощь Groovy
    def services = ['api', 'web', 'worker']
    def environments = ['dev', 'staging', 'prod']
    def results = [:]
    
    stage('Build All') {
        // ✅ Циклы напрямую
        services.each { svc ->
            results[svc] = sh(
                script: "docker build -t ${svc}:${BUILD_NUMBER} ./services/${svc}",
                returnStatus: true
            ) == 0
        }
    }
    
    stage('Dynamic Stages') {
        // ✅ Динамическое создание стадий
        services.findAll { results[it] }.each { svc ->
            stage("Test ${svc}") {
                sh "docker run ${svc}:${BUILD_NUMBER} npm test"
            }
        }
    }
    
    stage('Complex Logic') {
        // ✅ Сложная бизнес-логика
        def config = readJSON file: 'deploy-config.json'
        
        config.deployments.each { deployment ->
            if (shouldDeploy(deployment)) {
                deployTo(deployment.environment, deployment.services)
            }
        }
    }
    
    stage('External API') {
        // ✅ HTTP запросы, внешние API
        def response = httpRequest(
            url: 'https://api.example.com/status',
            authentication: 'api-creds'
        )
        
        def status = readJSON text: response.content
        
        if (status.canDeploy) {
            sh './deploy.sh'
        } else {
            error("Cannot deploy: ${status.reason}")
        }
    }
}

// ✅ Функции вне pipeline
def shouldDeploy(deployment) {
    return deployment.enabled && 
           (BRANCH_NAME == 'main' || deployment.environment != 'prod')
}

def deployTo(env, services) {
    services.each { svc ->
        sh "kubectl set image deployment/${svc} ${svc}=${svc}:${BUILD_NUMBER} -n ${env}"
    }
}
```

### Когда что выбрать

text

```
┌────────────────────────────────────────────────────────────┐
│              КАК ВЫБРАТЬ СТИЛЬ?                            │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ВЫБИРАЙ DECLARATIVE если:                                │
│  ─────────────────────────                                 │
│  ✅ Новый проект                                           │
│  ✅ Команда не знает Groovy                                │
│  ✅ Стандартный CI/CD flow (build → test → deploy)        │
│  ✅ Нужна валидация pipeline                              │
│  ✅ Используете Blue Ocean                                 │
│  ✅ Простые условия (when)                                 │
│  ✅ Хотите поддерживаемый код                              │
│                                                            │
│  ВЫБИРАЙ SCRIPTED если:                                   │
│  ────────────────────────                                  │
│  ⚙️  Очень сложная логика (много условий, циклов)         │
│  ⚙️  Динамическое создание стадий                         │
│  ⚙️  Интеграция с внешними API                            │
│  ⚙️  Legacy pipeline (уже написан на Scripted)            │
│  ⚙️  Нужны возможности, которых нет в Declarative         │
│  ⚙️  Команда хорошо знает Groovy                          │
│                                                            │
│  КОМБИНИРОВАННЫЙ ПОДХОД:                                  │
│  ─────────────────────────                                 │
│  Declarative + script {} блоки для сложных частей         │
│  Это ЛУЧШИЙ выбор для большинства проектов!               │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Комбинированный подход (рекомендуется)

groovy

```
// ═══════════════════════════════════════════════════════════
//          КОМБИНИРОВАННЫЙ ПОДХОД (ЛУЧШИЙ)
// ═══════════════════════════════════════════════════════════

pipeline {
    agent any
    
    stages {
        // Простые стадии — декларативно
        stage('Build') {
            steps {
                sh 'npm ci && npm run build'
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit') { steps { sh 'npm run test:unit' } }
                stage('E2E') { steps { sh 'npm run test:e2e' } }
            }
        }
        
        // Сложная логика — через script {}
        stage('Dynamic Deploy') {
            steps {
                script {
                    // Здесь полноценный Groovy!
                    def config = readJSON file: 'deploy.json'
                    
                    def targets = config.environments.findAll { env ->
                        (BRANCH_NAME == 'main' && env.production) ||
                        (BRANCH_NAME == 'develop' && env.staging)
                    }
                    
                    targets.each { target ->
                        echo "Deploying to ${target.name}"
                        
                        target.services.each { svc ->
                            sh "kubectl set image deployment/${svc} ${svc}=${svc}:${BUILD_NUMBER} -n ${target.namespace}"
                        }
                    }
                }
            }
        }
    }
    
    // post — декларативно (удобнее)
    post {
        always { cleanWs() }
        success { slackSend color: 'good', message: 'Done!' }
    }
}
```

---

## 3. Jenkins Shared Library

text

```
┌────────────────────────────────────────────────────────────┐
│              JENKINS SHARED LIBRARY                        │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Shared Library — переиспользуемый код для Pipeline       │
│                                                            │
│  ПРОБЛЕМА:                                                │
│  • 50 микросервисов                                       │
│  • У каждого свой Jenkinsfile                             │
│  • 90% кода одинакового (build, test, deploy)             │
│  • Нужно исправить баг → менять 50 файлов!                │
│                                                            │
│  РЕШЕНИЕ: Shared Library                                  │
│  • Общий код в отдельном репозитории                      │
│  • Все pipeline используют одну библиотеку                │
│  • Исправление в одном месте → работает везде             │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │  jenkins-shared-library/                             │ │
│  │  ├── vars/                   ← Глобальные функции    │ │
│  │  │   ├── buildApp.groovy                             │ │
│  │  │   ├── deployTo.groovy                             │ │
│  │  │   └── notifySlack.groovy                          │ │
│  │  ├── src/                    ← Классы (опционально)  │ │
│  │  │   └── com/company/                                │ │
│  │  │       └── Pipeline.groovy                         │ │
│  │  └── resources/              ← Файлы (templates)     │ │
│  │      └── config.yaml                                 │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Структура Shared Library

text

```
jenkins-shared-library/
├── vars/                          # Глобальные переменные/функции
│   ├── buildApp.groovy            # Вызов: buildApp()
│   ├── buildApp.txt               # Документация (опционально)
│   ├── deployTo.groovy            # Вызов: deployTo('prod')
│   ├── notifySlack.groovy         # Вызов: notifySlack('message')
│   └── standardPipeline.groovy    # Готовый pipeline
│
├── src/                           # Классы Groovy
│   └── com/
│       └── company/
│           ├── Docker.groovy      # import com.company.Docker
│           ├── Kubernetes.groovy
│           └── Utils.groovy
│
└── resources/                     # Статические файлы
    ├── config/
    │   └── default.yaml
    └── templates/
        └── Dockerfile.template
```

### Создание простой Shared Library

**vars/buildApp.groovy:**

groovy

```
// ═══════════════════════════════════════════════════════════
//          vars/buildApp.groovy
// ═══════════════════════════════════════════════════════════

// Простая функция (вызов: buildApp())
def call() {
    sh 'npm ci'
    sh 'npm run build'
}

// ═══════════════════════════════════════════════════════════
//          vars/buildApp.groovy (с параметрами)
// ═══════════════════════════════════════════════════════════

// Функция с параметрами (вызов: buildApp(type: 'maven'))
def call(Map config = [:]) {
    def type = config.type ?: 'npm'
    def skipTests = config.skipTests ?: false
    
    echo "Building with ${type}..."
    
    switch(type) {
        case 'npm':
            sh 'npm ci'
            sh 'npm run build'
            if (!skipTests) {
                sh 'npm test'
            }
            break
            
        case 'maven':
            def testFlag = skipTests ? '-DskipTests' : ''
            sh "mvn clean package ${testFlag}"
            break
            
        case 'gradle':
            def testFlag = skipTests ? '-x test' : ''
            sh "./gradlew build ${testFlag}"
            break
            
        default:
            error "Unknown build type: ${type}"
    }
}
```

**vars/deployTo.groovy:**

groovy

```
// ═══════════════════════════════════════════════════════════
//          vars/deployTo.groovy
// ═══════════════════════════════════════════════════════════

def call(String environment, Map config = [:]) {
    def namespace = config.namespace ?: environment
    def image = config.image ?: "myapp:${env.BUILD_NUMBER}"
    def deployment = config.deployment ?: 'myapp'
    
    echo "Deploying ${image} to ${environment}..."
    
    // Проверка окружения
    if (environment == 'production') {
        // Требовать подтверждение для продакшна
        timeout(time: 10, unit: 'MINUTES') {
            input message: "Deploy to PRODUCTION?", ok: 'Deploy'
        }
    }
    
    // Деплой
    withCredentials([file(credentialsId: "kubeconfig-${environment}", variable: 'KUBECONFIG')]) {
        sh """
            kubectl --kubeconfig=\$KUBECONFIG \
                set image deployment/${deployment} \
                ${deployment}=${image} \
                -n ${namespace}
            
            kubectl --kubeconfig=\$KUBECONFIG \
                rollout status deployment/${deployment} \
                -n ${namespace} \
                --timeout=5m
        """
    }
    
    echo "✅ Deployed successfully to ${environment}"
}
```

**vars/notifySlack.groovy:**

groovy

```
// ═══════════════════════════════════════════════════════════
//          vars/notifySlack.groovy
// ═══════════════════════════════════════════════════════════

def call(Map config = [:]) {
    def status = config.status ?: currentBuild.result ?: 'SUCCESS'
    def channel = config.channel ?: '#builds'
    def message = config.message ?: ''
    
    def color = [
        'SUCCESS': 'good',
        'UNSTABLE': 'warning',
        'FAILURE': 'danger',
        'ABORTED': '#808080'
    ][status] ?: 'warning'
    
    def icon = [
        'SUCCESS': '✅',
        'UNSTABLE': '⚠️',
        'FAILURE': '❌',
        'ABORTED': '🛑'
    ][status] ?: '❓'
    
    def defaultMessage = """
        ${icon} *${env.JOB_NAME}* #${env.BUILD_NUMBER}
        Status: ${status}
        Branch: ${env.BRANCH_NAME ?: 'N/A'}
        Duration: ${currentBuild.durationString}
        <${env.BUILD_URL}|View Build>
    """
    
    slackSend(
        channel: channel,
        color: color,
        message: message ?: defaultMessage
    )
}
```

**vars/standardPipeline.groovy (готовый pipeline!):**

groovy

```
// ═══════════════════════════════════════════════════════════
//          vars/standardPipeline.groovy
// ═══════════════════════════════════════════════════════════
// Вызов: standardPipeline(type: 'npm', deployTo: ['staging', 'production'])

def call(Map config = [:]) {
    def buildType = config.type ?: 'npm'
    def deployEnvironments = config.deployTo ?: []
    def slackChannel = config.slackChannel ?: '#builds'
    
    pipeline {
        agent any
        
        options {
            timeout(time: 1, unit: 'HOURS')
            timestamps()
            disableConcurrentBuilds()
        }
        
        stages {
            stage('Checkout') {
                steps {
                    checkout scm
                }
            }
            
            stage('Build') {
                steps {
                    // Вызываем другую функцию из библиотеки
                    buildApp(type: buildType)
                }
            }
            
            stage('Test') {
                steps {
                    script {
                        switch(buildType) {
                            case 'npm':
                                sh 'npm test'
                                break
                            case 'maven':
                                sh 'mvn test'
                                break
                        }
                    }
                }
                post {
                    always {
                        junit '**/test-results/**/*.xml'
                    }
                }
            }
            
            stage('Deploy') {
                when {
                    expression { deployEnvironments.size() > 0 }
                    branch 'main'
                }
                steps {
                    script {
                        deployEnvironments.each { env ->
                            stage("Deploy to ${env}") {
                                deployTo(env)
                            }
                        }
                    }
                }
            }
        }
        
        post {
            always {
                notifySlack(channel: slackChannel)
                cleanWs()
            }
        }
    }
}
```

### Подключение Shared Library

**Способ 1: Глобально (для всех pipeline)**

text

```
┌────────────────────────────────────────────────────────────┐
│  Manage Jenkins → System → Global Pipeline Libraries      │
│                                                            │
│  Name: my-shared-library                                  │
│  Default version: main                                    │
│  Load implicitly: ☑ (автоматически доступна везде)        │
│                                                            │
│  Retrieval method: Modern SCM                             │
│  Source Code Management: Git                              │
│  Repository URL: https://github.com/company/jenkins-lib   │
│  Credentials: github-token                                │
└────────────────────────────────────────────────────────────┘
```

**Способ 2: В Jenkinsfile**

groovy

```
// ═══════════════════════════════════════════════════════════
//          ПОДКЛЮЧЕНИЕ В JENKINSFILE
// ═══════════════════════════════════════════════════════════

// Вариант 1: Конкретная версия (рекомендуется)
@Library('my-shared-library@v1.2.0') _

// Вариант 2: Ветка
@Library('my-shared-library@main') _

// Вариант 3: Последняя версия (если Load implicitly включён)
@Library('my-shared-library') _

// Вариант 4: Несколько библиотек
@Library(['my-shared-library@v1.0', 'other-library@main']) _

// Вариант 5: Динамическая загрузка (внутри pipeline)
// library 'my-shared-library@v1.0'

// ВАЖНО: _ (underscore) в конце нужен, если после @Library ничего не импортируется

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                // Теперь можно использовать функции из библиотеки
                buildApp(type: 'npm')
            }
        }
        stage('Deploy') {
            steps {
                deployTo('staging')
            }
        }
    }
    post {
        always {
            notifySlack()
        }
    }
}
```

### Использование готового pipeline

groovy

```
// ═══════════════════════════════════════════════════════════
//          JENKINSFILE (МИНИМАЛЬНЫЙ!)
// ═══════════════════════════════════════════════════════════

@Library('my-shared-library@v1.0') _

// Весь pipeline в одну строку!
standardPipeline(
    type: 'npm',
    deployTo: ['staging', 'production'],
    slackChannel: '#frontend-builds'
)
```

groovy

```
// ═══════════════════════════════════════════════════════════
//          ДРУГОЙ ПРОЕКТ (MAVEN)
// ═══════════════════════════════════════════════════════════

@Library('my-shared-library@v1.0') _

standardPipeline(
    type: 'maven',
    deployTo: ['staging'],
    slackChannel: '#backend-builds'
)
```

---

## 4 Создание Shared Library

### Когда нужна Shared Library

text

```
┌────────────────────────────────────────────────────────────┐
│         КОГДА СОЗДАВАТЬ SHARED LIBRARY?                    │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ✅ СОЗДАВАЙ если:                                         │
│  • 5+ проектов с похожими Jenkinsfile                     │
│  • Копипаст кода между pipeline                           │
│  • Сложная логика (deploy, notifications, integrations)   │
│  • Нужна стандартизация CI/CD в компании                  │
│  • Хотите версионировать CI/CD код                        │
│                                                            │
│  ❌ НЕ НУЖНА если:                                         │
│  • 1-3 простых проекта                                    │
│  • Каждый pipeline уникален                               │
│  • Начинающая команда (лучше сначала освоить basics)      │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Полный пример Shared Library

**Структура:**

text

```
jenkins-shared-library/
├── vars/
│   ├── dockerBuild.groovy
│   ├── dockerPush.groovy
│   ├── k8sDeploy.groovy
│   ├── runTests.groovy
│   ├── notifySlack.groovy
│   └── microservicePipeline.groovy
├── src/
│   └── com/
│       └── company/
│           └── Config.groovy
└── resources/
    └── kubernetes/
        └── deployment.yaml
```

**src/com/company/Config.groovy:**

groovy

```
// ═══════════════════════════════════════════════════════════
//          src/com/company/Config.groovy
// ═══════════════════════════════════════════════════════════

package com.company

class Config {
    static Map environments = [
        dev: [
            namespace: 'dev',
            replicas: 1,
            kubeconfig: 'kubeconfig-dev'
        ],
        staging: [
            namespace: 'staging',
            replicas: 2,
            kubeconfig: 'kubeconfig-staging'
        ],
        production: [
            namespace: 'production',
            replicas: 3,
            kubeconfig: 'kubeconfig-prod',
            requireApproval: true
        ]
    ]
    
    static String dockerRegistry = 'registry.company.com'
    static String slackChannel = '#deployments'
}
```

**vars/dockerBuild.groovy:**

groovy

```
// ═══════════════════════════════════════════════════════════
//          vars/dockerBuild.groovy
// ═══════════════════════════════════════════════════════════

import com.company.Config

def call(Map params = [:]) {
    def imageName = params.name ?: env.JOB_BASE_NAME
    def tag = params.tag ?: env.BUILD_NUMBER
    def dockerfile = params.dockerfile ?: 'Dockerfile'
    def context = params.context ?: '.'
    
    def fullImage = "${Config.dockerRegistry}/${imageName}:${tag}"
    
    echo "🐳 Building Docker image: ${fullImage}"
    
    sh """
        docker build \
            -f ${dockerfile} \
            -t ${fullImage} \
            --build-arg VERSION=${tag} \
            --build-arg BUILD_NUMBER=${env.BUILD_NUMBER} \
            ${context}
    """
    
    // Вернуть имя образа для следующих шагов
    return fullImage
}
```

**vars/k8sDeploy.groovy:**

groovy

```
// ═══════════════════════════════════════════════════════════
//          vars/k8sDeploy.groovy
// ═══════════════════════════════════════════════════════════

import com.company.Config

def call(Map params) {
    def environment = params.environment
    def image = params.image
    def deployment = params.deployment ?: env.JOB_BASE_NAME
    
    def envConfig = Config.environments[environment]
    if (!envConfig) {
        error "Unknown environment: ${environment}"
    }
    
    echo "🚀 Deploying to ${environment}..."
    
    // Подтверждение для production
    if (envConfig.requireApproval) {
        timeout(time: 30, unit: 'MINUTES') {
            input message: "Deploy to ${environment.toUpperCase()}?",
                  ok: 'Deploy',
                  submitter: 'devops,admin'
        }
    }
    
    // Деплой
    withCredentials([file(credentialsId: envConfig.kubeconfig, variable: 'KUBECONFIG')]) {
        sh """
            kubectl set image deployment/${deployment} \
                ${deployment}=${image} \
                -n ${envConfig.namespace}
            
            kubectl rollout status deployment/${deployment} \
                -n ${envConfig.namespace} \
                --timeout=5m
        """
    }
    
    notifySlack(
        message: "✅ Deployed ${deployment} to ${environment}",
        status: 'SUCCESS'
    )
}
```

**vars/microservicePipeline.groovy:**

groovy

```
// ═══════════════════════════════════════════════════════════
//          vars/microservicePipeline.groovy
// ═══════════════════════════════════════════════════════════

import com.company.Config

def call(Map params = [:]) {
    def serviceName = params.name ?: env.JOB_BASE_NAME
    def buildType = params.buildType ?: 'npm'
    def deployEnvs = params.deployTo ?: []
    
    pipeline {
        agent any
        
        environment {
            SERVICE_NAME = "${serviceName}"
            DOCKER_IMAGE = ''
        }
        
        options {
            timeout(time: 30, unit: 'MINUTES')
            timestamps()
            disableConcurrentBuilds()
        }
        
        stages {
            stage('Checkout') {
                steps {
                    checkout scm
                    script {
                        env.GIT_COMMIT_SHORT = sh(
                            script: 'git rev-parse --short HEAD',
                            returnStdout: true
                        ).trim()
                    }
                }
            }
            
            stage('Build & Test') {
                steps {
                    script {
                        buildApp(type: buildType)
                        runTests(type: buildType)
                    }
                }
            }
            
            stage('Docker Build') {
                steps {
                    script {
                        env.DOCKER_IMAGE = dockerBuild(
                            name: serviceName,
                            tag: "${BUILD_NUMBER}-${GIT_COMMIT_SHORT}"
                        )
                    }
                }
            }
            
            stage('Docker Push') {
                steps {
                    dockerPush(image: env.DOCKER_IMAGE)
                }
            }
            
            stage('Deploy') {
                when {
                    expression { deployEnvs.size() > 0 }
                    anyOf {
                        branch 'main'
                        branch 'develop'
                    }
                }
                steps {
                    script {
                        def targetEnvs = []
                        
                        if (env.BRANCH_NAME == 'develop') {
                            targetEnvs = deployEnvs.findAll { it == 'dev' || it == 'staging' }
                        } else if (env.BRANCH_NAME == 'main') {
                            targetEnvs = deployEnvs
                        }
                        
                        targetEnvs.each { environment ->
                            stage("Deploy to ${environment}") {
                                k8sDeploy(
                                    environment: environment,
                                    image: env.DOCKER_IMAGE,
                                    deployment: serviceName
                                )
                            }
                        }
                    }
                }
            }
        }
        
        post {
            success {
                notifySlack(status: 'SUCCESS')
            }
            failure {
                notifySlack(status: 'FAILURE')
            }
            always {
                cleanWs()
            }
        }
    }
}
```

**Использование в проектах:**

groovy

```
// ═══════════════════════════════════════════════════════════
//          Jenkinsfile (в каждом микросервисе)
// ═══════════════════════════════════════════════════════════

@Library('company-shared-library@v2.0') _

microservicePipeline(
    name: 'user-service',
    buildType: 'maven',
    deployTo: ['dev', 'staging', 'production']
)
```

groovy

```
// Другой сервис
@Library('company-shared-library@v2.0') _

microservicePipeline(
    name: 'order-service',
    buildType: 'npm',
    deployTo: ['dev', 'staging']
)
```

---

## 5. Ограничение доступа

text

```
┌────────────────────────────────────────────────────────────┐
│              КОНТРОЛЬ ДОСТУПА В JENKINS                    │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  УРОВНИ БЕЗОПАСНОСТИ:                                     │
│  1. Authentication — КТО может войти                      │
│  2. Authorization  — ЧТО может делать                     │
│                                                            │
│  СТРАТЕГИИ АВТОРИЗАЦИИ:                                   │
│  • Anyone can do anything (небезопасно!)                  │
│  • Legacy mode (устаревшее)                               │
│  • Logged-in users can do anything                        │
│  • Matrix-based security ⭐                                │
│  • Project-based Matrix Authorization ⭐⭐                  │
│  • Role-Based Strategy (плагин) ⭐⭐⭐                      │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Matrix-based Security

text

```
┌────────────────────────────────────────────────────────────┐
│         MATRIX-BASED SECURITY                              │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Manage Jenkins → Security → Authorization                │
│  → Matrix-based security                                  │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │           │Admin│Read│Build│Config│Delete│...       │ │
│  ├───────────┼─────┼────┼─────┼──────┼──────┼──────────┤ │
│  │ admin     │  ✅ │ ✅ │  ✅ │  ✅  │  ✅  │   ✅     │ │
│  │ developer │  ❌ │ ✅ │  ✅ │  ❌  │  ❌  │   ❌     │ │
│  │ viewer    │  ❌ │ ✅ │  ❌ │  ❌  │  ❌  │   ❌     │ │
│  │ deployer  │  ❌ │ ✅ │  ✅ │  ❌  │  ❌  │   ✅     │ │
│  │ anonymous │  ❌ │ ❌ │  ❌ │  ❌  │  ❌  │   ❌     │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
│  ПРАВА:                                                   │
│  • Overall: Administer, Read, RunScripts                  │
│  • Job: Build, Cancel, Configure, Create, Delete, Read    │
│  • View: Configure, Create, Delete, Read                  │
│  • Agent: Build, Configure, Connect, Create, Delete       │
│  • Credentials: Create, Delete, ManageDomains, Update     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Project-based Matrix Authorization

text

```
┌────────────────────────────────────────────────────────────┐
│         PROJECT-BASED MATRIX                               │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Позволяет настраивать права ДЛЯ КАЖДОГО JOB отдельно    │
│                                                            │
│  Manage Jenkins → Security → Authorization                │
│  → Project-based Matrix Authorization Strategy            │
│                                                            │
│  ШАГ 1: Настроить глобальные права (как в Matrix)         │
│                                                            │
│  ШАГ 2: В каждом Job → Configure → Enable project-based   │
│         security                                          │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ Job: production-deploy                               │ │
│  │                                                      │ │
│  │ ☑ Enable project-based security                      │ │
│  │                                                      │ │
│  │ Inheritance Strategy:                                │ │
│  │   ● Inherit permissions from parent                  │ │
│  │   ○ Do not inherit permissions                       │ │
│  │                                                      │ │
│  │ User/group permissions:                              │ │
│  │           │Read│Build│Configure│Delete│              │ │
│  │ ──────────┼────┼─────┼─────────┼──────┤              │ │
│  │ devops    │ ✅ │  ✅ │   ✅    │  ❌  │              │ │
│  │ developer │ ✅ │  ❌ │   ❌    │  ❌  │ ← Только read │ │
│  │ lead      │ ✅ │  ✅ │   ❌    │  ❌  │              │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
│  ПРИМЕР СЦЕНАРИЯ:                                         │
│  • production-deploy: только devops может запускать       │
│  • staging-deploy: developers + devops                    │
│  • dev-deploy: все могут запускать                        │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Role-Based Strategy (плагин)

text

```
┌────────────────────────────────────────────────────────────┐
│         ROLE-BASED AUTHORIZATION STRATEGY                  │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Требует: Role-based Authorization Strategy Plugin        │
│                                                            │
│  ПРЕИМУЩЕСТВА:                                            │
│  • Роли вместо отдельных пользователей                    │
│  • Паттерны для Job (regex)                               │
│  • Проще управлять большим количеством пользователей      │
│                                                            │
│  ТИПЫ РОЛЕЙ:                                              │
│  • Global roles  — права на весь Jenkins                  │
│  • Item roles    — права на Job/Folder (по паттерну)      │
│  • Agent roles   — права на агенты                        │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ GLOBAL ROLES:                                        │ │
│  │ ────────────────────────                              │ │
│  │ admin:     Overall/Administer ✅                      │ │
│  │ developer: Overall/Read ✅, Job/Read ✅               │ │
│  │ viewer:    Overall/Read ✅                            │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ ITEM ROLES (по паттерну):                            │ │
│  │ ────────────────────────────                          │ │
│  │ frontend-dev:                                        │ │
│  │   Pattern: frontend-.*                               │ │
│  │   Permissions: Build ✅, Read ✅, Cancel ✅           │ │
│  │                                                      │ │
│  │ backend-dev:                                         │ │
│  │   Pattern: backend-.*|api-.*                         │ │
│  │   Permissions: Build ✅, Read ✅                      │ │
│  │                                                      │ │
│  │ prod-deployer:                                       │ │
│  │   Pattern: .*-production                             │ │
│  │   Permissions: Build ✅, Read ✅                      │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ ASSIGN ROLES:                                        │ │
│  │ ───────────────                                       │ │
│  │ user: john                                           │ │
│  │   Global: developer                                  │ │
│  │   Item: frontend-dev                                 │ │
│  │                                                      │ │
│  │ user: jane                                           │ │
│  │   Global: developer                                  │ │
│  │   Item: backend-dev, prod-deployer                   │ │
│  │                                                      │ │
│  │ group: frontend-team                                 │ │
│  │   Global: developer                                  │ │
│  │   Item: frontend-dev                                 │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Ограничение на уровне Folder

text

```
┌────────────────────────────────────────────────────────────┐
│         FOLDER-BASED SECURITY                              │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Структура:                                               │
│  ├── Frontend/                  ← Team: frontend-team     │
│  │   ├── web-app                                          │
│  │   ├── mobile-app                                       │
│  │   └── admin-panel                                      │
│  ├── Backend/                   ← Team: backend-team      │
│  │   ├── api-service                                      │
│  │   └── auth-service                                     │
│  └── DevOps/                    ← Team: devops-team       │
│      ├── infrastructure                                   │
│      └── monitoring                                       │
│                                                            │
│  Настройка Folder:                                        │
│  Folder → Configure → Properties → Enable Folder Auth     │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ Frontend Folder:                                     │ │
│  │                                                      │ │
│  │ ☑ Enable Folder-based authorization                  │ │
│  │                                                      │ │
│  │           │Read│Build│Configure│Create│Delete│       │ │
│  │ ──────────┼────┼─────┼─────────┼──────┼──────┤       │ │
│  │ frontend- │ ✅ │  ✅ │   ✅    │  ✅  │  ✅  │       │ │
│  │ team      │    │     │         │      │      │       │ │
│  │ devops    │ ✅ │  ✅ │   ✅    │  ✅  │  ✅  │       │ │
│  │ developer │ ✅ │  ✅ │   ❌    │  ❌  │  ❌  │       │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
│  Преимущества:                                            │
│  • Команды изолированы друг от друга                      │
│  • Свои credentials в каждой папке                        │
│  • Легко управлять доступом                               │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## 6. Jenkins Sandbox

text

```
┌────────────────────────────────────────────────────────────┐
│              JENKINS SANDBOX (Script Security)             │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Sandbox — ограниченное окружение для выполнения Groovy   │
│  кода в Pipeline. Защищает Jenkins от вредоносных скриптов│
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │                                                      │ │
│  │  Jenkinsfile (Groovy код)                            │ │
│  │         │                                            │ │
│  │         ▼                                            │ │
│  │  ┌─────────────────────┐                             │ │
│  │  │      SANDBOX        │                             │ │
│  │  │  ─────────────────  │                             │ │
│  │  │  • Ограниченный API │                             │ │
│  │  │  • Whitelist методов│                             │ │
│  │  │  • Блокировка опасных│                            │ │
│  │  │    операций         │                             │ │
│  │  └─────────────────────┘                             │ │
│  │         │                                            │ │
│  │         ▼                                            │ │
│  │  Безопасное выполнение                               │ │
│  │                                                      │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
│  ЗАЧЕМ НУЖЕН SANDBOX?                                     │
│  • Любой может создать PR с вредоносным Jenkinsfile       │
│  • Jenkinsfile имеет доступ к Jenkins API                 │
│  • Без sandbox можно украсть credentials, изменить config │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Как работает Sandbox

text

```
┌────────────────────────────────────────────────────────────┐
│              КАК РАБОТАЕТ SANDBOX                          │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ✅ РАЗРЕШЕНО в Sandbox (по умолчанию):                    │
│  • Базовые Groovy операции (циклы, условия, строки)       │
│  • Pipeline DSL (sh, echo, stage, parallel)               │
│  • Стандартные шаги (checkout, build, junit)              │
│  • Работа с env, params, currentBuild                     │
│                                                            │
│  ❌ ЗАБЛОКИРОВАНО в Sandbox:                               │
│  • System.exit() — может убить Jenkins                    │
│  • Runtime.exec() — выполнение произвольных команд        │
│  • File операции вне workspace                            │
│  • Доступ к Jenkins internal API                          │
│  • Reflection (Class.forName, Method.invoke)              │
│  • Сетевые операции (Socket, URL без одобрения)           │
│                                                            │
│  КОГДА SANDBOX АКТИВЕН:                                   │
│  • Pipeline из SCM (Jenkinsfile в репозитории) ✅         │
│  • Pipeline script из Job configuration ✅                │
│  • Replay (повторный запуск с изменениями) ✅             │
│                                                            │
│  КОГДА SANDBOX НЕ АКТИВЕН:                                │
│  • Shared Library (загружена глобально) ❌                │
│  • Script Console (Manage Jenkins) ❌                     │
│  • System Groovy Build Step ❌                            │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Одобрение скриптов

text

```
┌────────────────────────────────────────────────────────────┐
│              SCRIPT APPROVAL                               │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Когда pipeline использует неодобренный метод:            │
│                                                            │
│  1. Билд падает с ошибкой:                                │
│     "Scripts not permitted to use method X"               │
│                                                            │
│  2. Админ должен одобрить:                                │
│     Manage Jenkins → In-process Script Approval           │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ Pending Script Approvals:                            │ │
│  │                                                      │ │
│  │ ⚠️  method java.lang.String toUpperCase              │ │
│  │    [Approve] [Deny]                                  │ │
│  │                                                      │ │
│  │ ⚠️  method groovy.json.JsonSlurper parseText         │ │
│  │    java.lang.String                                  │ │
│  │    [Approve] [Deny]                                  │ │
│  │                                                      │ │
│  │ ⚠️  staticMethod java.lang.System getenv             │ │
│  │    java.lang.String                                  │ │
│  │    [Approve] [Deny]                                  │ │
│  │                                                      │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
│  ПОСЛЕ ОДОБРЕНИЯ:                                         │
│  • Метод добавляется в whitelist                          │
│  • Все pipeline могут его использовать                    │
│  • Одобрение сохраняется в scriptApproval.xml             │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Примеры проблем с Sandbox

groovy

```
// ═══════════════════════════════════════════════════════════
//          ПРИМЕРЫ: ЧТО БЛОКИРУЕТСЯ
// ═══════════════════════════════════════════════════════════

pipeline {
    agent any
    stages {
        stage('Examples') {
            steps {
                script {
                    // ❌ ЗАБЛОКИРОВАНО: new File()
                    // def file = new File('/etc/passwd')
                    // ERROR: Scripts not permitted to use new java.io.File
                    
                    // ✅ ВМЕСТО ЭТОГО: readFile (Pipeline step)
                    def content = readFile 'config.txt'
                    
                    // ───────────────────────────────────────
                    
                    // ❌ ЗАБЛОКИРОВАНО: URL без одобрения
                    // def url = new URL('https://api.example.com')
                    // def text = url.getText()
                    
                    // ✅ ВМЕСТО ЭТОГО: httpRequest (плагин)
                    def response = httpRequest 'https://api.example.com'
                    
                    // ───────────────────────────────────────
                    
                    // ❌ ЗАБЛОКИРОВАНО: System.getenv()
                    // def path = System.getenv('PATH')
                    
                    // ✅ ВМЕСТО ЭТОГО: env
                    def path = env.PATH
                    
                    // ───────────────────────────────────────
                    
                    // ❌ ЗАБЛОКИРОВАНО: execute()
                    // def result = "ls -la".execute().text
                    
                    // ✅ ВМЕСТО ЭТОГО: sh step
                    def result = sh(script: 'ls -la', returnStdout: true)
                    
                    // ───────────────────────────────────────
                    
                    // ⚠️  ТРЕБУЕТ ОДОБРЕНИЯ: JsonSlurper
                    // def json = new groovy.json.JsonSlurper()
                    // def data = json.parseText('{"key":"value"}')
                    
                    // ✅ ВМЕСТО ЭТОГО: readJSON (Pipeline step)
                    writeFile file: 'temp.json', text: '{"key":"value"}'
                    def data = readJSON file: 'temp.json'
                }
            }
        }
    }
}
```

### Отключение Sandbox (не рекомендуется!)

text

```
┌────────────────────────────────────────────────────────────┐
│         ⚠️  ОТКЛЮЧЕНИЕ SANDBOX                             │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Можно отключить sandbox для конкретного pipeline:        │
│                                                            │
│  Job → Configure → Pipeline                               │
│  Definition: Pipeline script                              │
│  ☐ Use Groovy Sandbox   ← СНЯТЬ ГАЛОЧКУ                   │
│                                                            │
│  ⚠️  ОПАСНО! Это требует:                                  │
│  • Права на Overall/RunScripts у пользователя             │
│  • Полное доверие к коду pipeline                         │
│  • НЕ использовать для Pipeline из SCM                    │
│                                                            │
│  Shared Library — АЛЬТЕРНАТИВА:                           │
│  • Код в Shared Library выполняется БЕЗ sandbox           │
│  • Но Shared Library контролируется админами              │
│  • Безопаснее, чем отключение sandbox в каждом job        │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Best Practices безопасности

text

```
┌────────────────────────────────────────────────────────────┐
│         BEST PRACTICES БЕЗОПАСНОСТИ                        │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  1. ОСТАВИТЬ SANDBOX ВКЛЮЧЁННЫМ                           │
│     • Это защита от вредоносного кода                     │
│     • Одобрять методы по необходимости                    │
│                                                            │
│  2. ИСПОЛЬЗОВАТЬ SHARED LIBRARY                           │
│     • Сложный код — в Shared Library                      │
│     • Library контролируется админами                     │
│     • Pipeline остаётся простым                           │
│                                                            │
│  3. МИНИМУМ ПРАВ                                          │
│     • Developers: Build, Read                             │
│     • DevOps: + Configure, Credentials                    │
│     • Admin: всё                                          │
│                                                            │
│  4. FOLDER ISOLATION                                      │
│     • Команды в отдельных Folder                          │
│     • Свои credentials в каждой папке                     │
│                                                            │
│  5. АУДИТ                                                 │
│     • Audit Trail Plugin — логировать действия            │
│     • Кто одобрил скрипт? Кто запустил деплой?           │
│                                                            │
│  6. CREDENTIALS                                           │
│     • Минимальный scope (Folder, не Global)               │
│     • Ротация секретов                                    │
│     • Не логировать значения                              │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Итоговая шпаргалка

text

```
┌────────────────────────────────────────────────────────────┐
│              ДОПОЛНИТЕЛЬНЫЕ ТЕМЫ — ШПАРГАЛКА               │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  MULTI-BRANCH:                                            │
│    • Авто-создание Pipeline для каждой ветки              │
│    • BRANCH_NAME, CHANGE_ID — переменные окружения        │
│    • when { branch 'main' } — условия для веток           │
│    • when { changeRequest() } — условия для PR            │
│                                                            │
│  SCRIPTED vs DECLARATIVE:                                 │
│    • Declarative — для большинства случаев ⭐              │
│    • Scripted — для сложной логики                        │
│    • Комбинация: Declarative + script {} — лучший выбор   │
│                                                            │
│  SHARED LIBRARY:                                          │
│    • vars/ — глобальные функции (buildApp.groovy)         │
│    • src/ — классы Groovy                                 │
│    • @Library('name@version') _ — подключение             │
│                                                            │
│  БЕЗОПАСНОСТЬ:                                            │
│    • Matrix-based security — права по пользователям       │
│    • Project-based — права на каждый Job                  │
│    • Role-based (плагин) — роли + паттерны ⭐              │
│    • Folder — изоляция команд                             │
│                                                            │
│  SANDBOX:                                                 │
│    • Ограничивает Groovy в Pipeline                       │
│    • Script Approval — одобрение методов                  │
│    • Shared Library — выполняется без sandbox             │
│                                                            │
└────────────────────────────────────────────────────────────┘
```