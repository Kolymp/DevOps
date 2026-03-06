## 1. Infrastructure as Code (IaC)

### Что такое Infrastructure as Code?

**Infrastructure as Code (IaC)** — это подход к управлению и provisioning инфраструктуры через **машинночитаемые конфигурационные файлы**, а не через ручную настройку в веб-консоли или CLI команды.

**Простая аналогия:**

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║  БЕЗ IaC (императивный подход):                                      ║
║  ═══════════════════════════════                                     ║
║                                                                      ║
║  Вы говорите компьютеру КАК сделать (шаг за шагом):                  ║
║                                                                      ║
║  1. Зайди в консоль AWS                                              ║
║  2. Кликни "Create EC2 instance"                                     ║
║  3. Выбери t2.micro                                                  ║
║  4. Выбери Ubuntu 20.04                                              ║
║  5. Создай Security Group                                            ║
║  6. Добавь правило: порт 80 открыт                                   ║
║  7. Кликни "Launch"                                                  ║
║                                                                      ║
║  Проблемы:                                                           ║
║  ❌ Невоспроизводимо (забыли шаг 6 → сервер недоступен)              ║
║  ❌ Нет версионирования (что изменилось? когда? кем?)                ║
║  ❌ Ручная работа → ошибки                                           ║
║  ❌ Сложно масштабировать (создать 100 серверов?)                    ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  С IaC (декларативный подход):                                       ║
║  ══════════════════════════════                                      ║
║                                                                      ║
║  Вы говорите компьютеру ЧТО вы хотите (описываете желаемое состояние): ║
║                                                                      ║
║  # main.tf                                                           ║
║  resource "aws_instance" "web" {                                     ║
║    ami           = "ami-0c55b159cbfafe1f0"                           ║
║    instance_type = "t2.micro"                                        ║
║  }                                                                   ║
║                                                                      ║
║  resource "aws_security_group" "web_sg" {                            ║
║    ingress {                                                         ║
║      from_port = 80                                                  ║
║      to_port   = 80                                                  ║
║      protocol  = "tcp"                                               ║
║    }                                                                 ║
║  }                                                                   ║
║                                                                      ║
║  Преимущества:                                                       ║
║  ✅ Воспроизводимо (запуск = идентичный результат)                   ║
║  ✅ Версионируется в Git (history, rollback, code review)            ║
║  ✅ Автоматизировано (CI/CD pipeline)                                ║
║  ✅ Масштабируется (count = 100 → 100 серверов)                      ║
║  ✅ Документация кодом (конфиг = документация)                       ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### Ключевые принципы IaC

text

```
┌────────────────────────────────────────────────────────────────────┐
│                    ПРИНЦИПЫ IaC                                    │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  1. DECLARATIVE vs IMPERATIVE                                      │
│     ═══════════════════════════                                    │
│                                                                    │
│     Декларативный (Terraform, CloudFormation):                     │
│     • Описываете ЖЕЛАЕМОЕ СОСТОЯНИЕ                                │
│     • Инструмент САМ решает, КАК его достичь                       │
│                                                                    │
│     Императивный (Ansible, Chef):                                  │
│     • Описываете ШАГИ для достижения состояния                     │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  2. IDEMPOTENCY (Идемпотентность)                                  │
│     ══════════════════════════════                                 │
│                                                                    │
│     Запуск N раз = тот же результат, что и 1 раз                   │
│                                                                    │
│     terraform apply (3 раза подряд)                                │
│     → 1-й раз: создаёт ресурсы                                     │
│     → 2-й раз: "No changes" (уже создано)                          │
│     → 3-й раз: "No changes"                                        │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  3. VERSION CONTROL                                                │
│     ═══════════════                                                │
│                                                                    │
│     Конфигурация хранится в Git:                                   │
│     • История изменений (кто, когда, почему)                       │
│     • Code review перед изменением инфраструктуры                  │
│     • Rollback к предыдущей версии                                 │
│     • Branching (dev, staging, prod конфигурации)                  │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  4. REUSABILITY (Переиспользование)                                │
│     ═══════════════════════════                                    │
│                                                                    │
│     • Модули (создать шаблон → использовать многократно)           │
│     • DRY (Don't Repeat Yourself)                                  │
│                                                                    │
│     Пример:                                                        │
│     module "vpc" {                                                 │
│       source = "./modules/vpc"                                     │
│     }                                                              │
│                                                                    │
│     Используем модуль VPC в prod, staging, dev                     │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  5. TESTING                                                        │
│     ════════                                                       │
│                                                                    │
│     • Unit tests (Terratest)                                       │
│     • Integration tests                                            │
│     • Policy as Code (OPA, Sentinel)                               │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

### Инструменты IaC

text

```
┌────────────────────────────────────────────────────────────────────┐
│                   ПОПУЛЯРНЫЕ ИНСТРУМЕНТЫ IaC                       │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  TERRAFORM (HashiCorp)                                             │
│  ══════════════════════                                            │
│  • Cloud-agnostic (AWS, Azure, GCP, Yandex Cloud, 3000+ providers) │
│  • Декларативный (HCL язык)                                        │
│  • Открытый исходный код (Mozilla Public License)                  │
│  • State management (отслеживает текущее состояние)                │
│  • Самый популярный multi-cloud инструмент                         │
│                                                                    │
│  📌 Применение: мультиоблачные инфраструктуры, любые облака        │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  AWS CloudFormation                                                │
│  ═══════════════════                                               │
│  • Только для AWS (vendor lock-in)                                 │
│  • Декларативный (JSON/YAML)                                       │
│  • Бесплатный (часть AWS)                                          │
│  • Нативная интеграция с AWS сервисами                             │
│                                                                    │
│  📌 Применение: если используете ТОЛЬКО AWS                        │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  Pulumi                                                            │
│  ══════                                                            │
│  • Cloud-agnostic                                                  │
│  • Использует НАСТОЯЩИЕ языки программирования:                    │
│    TypeScript, Python, Go, C#, Java                                │
│  • State management                                                │
│  • Более гибкий, чем Terraform (программирование vs конфиг)        │
│                                                                    │
│  📌 Применение: если хотите писать IaC на Python/JS                │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  Ansible                                                           │
│  ═══════                                                           │
│  • Императивный (шаги выполнения)                                  │
│  • Больше для CONFIGURATION MANAGEMENT (настройка ОС, деплой)      │
│  • YAML синтаксис                                                  │
│  • Agentless (работает по SSH)                                     │
│                                                                    │
│  📌 Применение: настройка серверов ПОСЛЕ создания                  │
│     (Terraform создаёт VM → Ansible настраивает)                   │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  Azure Resource Manager (ARM Templates)                            │
│  • Только для Azure                                                │
│  • JSON-конфигурация                                               │
│                                                                    │
│  Google Cloud Deployment Manager                                   │
│  • Только для GCP                                                  │
│  • YAML/Jinja2                                                     │
│                                                                    │
│  Chef, Puppet                                                      │
│  • Configuration management (больше для настройки, чем provisioning)│
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## 2. Terraform — подробное введение

### Что такое Terraform?

**Terraform** — это **open-source инструмент** от HashiCorp для **declarative provisioning** облачной инфраструктуры. Terraform позволяет описать всю инфраструктуру (серверы, сети, БД, DNS) в **конфигурационных файлах** и управлять ей через **CLI команды**.

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                    ЗАЧЕМ НУЖЕН TERRAFORM?                            ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  1. MULTI-CLOUD                                                      ║
║     Одинаковый синтаксис для AWS, Azure, GCP, Yandex Cloud          ║
║     → Знаешь Terraform → можешь работать с любым облаком             ║
║                                                                      ║
║  2. DECLARATIVE                                                      ║
║     Описываете ЖЕЛАЕМОЕ состояние → Terraform делает остальное       ║
║                                                                      ║
║  3. PLAN BEFORE APPLY                                                ║
║     terraform plan — показывает ЧТО изменится                        ║
║     → Предотвращает случайное удаление production ресурсов           ║
║                                                                      ║
║  4. DEPENDENCY MANAGEMENT                                            ║
║     Terraform автоматически определяет порядок создания:             ║
║     1. VPC                                                           ║
║     2. Subnet (зависит от VPC)                                       ║
║     3. VM (зависит от Subnet)                                        ║
║                                                                      ║
║  5. STATE TRACKING                                                   ║
║     Terraform ЗНАЕТ текущее состояние инфраструктуры                 ║
║     → Может обновить/удалить ТОЛЬКО изменённые ресурсы               ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### Какие задачи решает Terraform?

text

```
┌────────────────────────────────────────────────────────────────────┐
│                   ЗАДАЧИ TERRAFORM                                 │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  1. PROVISIONING ИНФРАСТРУКТУРЫ                                    │
│     ════════════════════════════                                   │
│                                                                    │
│     • Создание VM, контейнеров, Kubernetes кластеров               │
│     • Настройка сетей (VPC, подсети, маршруты)                     │
│     • Создание баз данных (RDS, Cloud SQL, Managed PostgreSQL)     │
│     • Настройка балансировщиков нагрузки                           │
│     • DNS записи                                                   │
│     • S3 buckets и хранилища                                       │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  2. УПРАВЛЕНИЕ ИЗМЕНЕНИЯМИ                                         │
│     ══════════════════════                                         │
│                                                                    │
│     • Изменить тип VM (t2.micro → t2.small)                        │
│     • Добавить/удалить правило firewall                            │
│     • Увеличить размер диска                                       │
│     • Terraform ЗНАЕТ что изменилось и применяет только изменения  │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  3. MULTI-CLOUD ORCHESTRATION                                      │
│     ═══════════════════════════                                    │
│                                                                    │
│     • AWS для compute                                              │
│     • GCP для ML                                                   │
│     • Yandex Cloud для хранения данных в РФ                        │
│     • Cloudflare для CDN                                           │
│     • Datadog для мониторинга                                      │
│                                                                    │
│     ВСЁ в одном Terraform конфиге!                                 │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  4. REPRODUCIBILITY (Воспроизводимость)                            │
│     ═══════════════════════════════════════                        │
│                                                                    │
│     • Создать идентичное окружение для staging/dev                 │
│     • Disaster Recovery (пересоздать инфраструктуру за минуты)     │
│     • Compliance (аудит: вся инфраструктура описана в Git)         │
│                                                                    │
│  ────────────────────────────────────────────────────────          │
│                                                                    │
│  5. COST OPTIMIZATION                                              │
│     ═══════════════════                                            │
│                                                                    │
│     • terraform destroy — удалить dev-окружение на выходные        │
│     • Автоматизация: создать окружение утром, удалить вечером      │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## 3. Структура Terraform конфигурации

### Основные блоки Terraform

Terraform конфигурация состоит из **блоков** (blocks), написанных на языке **HCL (HashiCorp Configuration Language)**.

hcl

```
# ════════════════════════════════════════════════════════════
# 1. TERRAFORM BLOCK — настройки Terraform
# ════════════════════════════════════════════════════════════

terraform {
  # Минимальная версия Terraform
  required_version = ">= 1.0"

  # Требуемые провайдеры
  required_providers {
    yandex = {
      source  = "yandex-cloud/yandex"
      version = "~> 0.100"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.5"
    }
  }

  # Backend для хранения state (опционально)
  backend "s3" {
    endpoint   = "storage.yandexcloud.net"
    bucket     = "my-terraform-state"
    key        = "terraform.tfstate"
    region     = "ru-central1"
  }
}


# ════════════════════════════════════════════════════════════
# 2. PROVIDER BLOCK — конфигурация провайдера
# ════════════════════════════════════════════════════════════

provider "yandex" {
  # Credentials (лучше через переменные окружения)
  token     = var.yc_token  # или через YC_TOKEN env var
  cloud_id  = var.cloud_id
  folder_id = var.folder_id
  zone      = "ru-central1-a"
}


# ════════════════════════════════════════════════════════════
# 3. VARIABLE BLOCK — входные переменные
# ════════════════════════════════════════════════════════════

variable "yc_token" {
  description = "Yandex Cloud OAuth token"
  type        = string
  sensitive   = true  # не показывать в логах
}

variable "vm_count" {
  description = "Number of VMs to create"
  type        = number
  default     = 1
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}


# ════════════════════════════════════════════════════════════
# 4. LOCALS BLOCK — локальные переменные (вычисляемые)
# ════════════════════════════════════════════════════════════

locals {
  # Общие теги для всех ресурсов
  common_labels = {
    environment = var.environment
    managed_by  = "terraform"
    project     = "my-project"
  }

  # Динамическое имя VM
  vm_name = "${var.environment}-web-server"
}


# ════════════════════════════════════════════════════════════
# 5. DATA SOURCE — чтение существующих ресурсов
# ════════════════════════════════════════════════════════════

# Получить информацию о существующей сети
data "yandex_vpc_network" "default" {
  name = "default"
}

# Получить последний образ Ubuntu
data "yandex_compute_image" "ubuntu" {
  family = "ubuntu-2004-lts"
}


# ════════════════════════════════════════════════════════════
# 6. RESOURCE BLOCK — создание ресурсов
# ════════════════════════════════════════════════════════════

# Создать подсеть
resource "yandex_vpc_subnet" "subnet-a" {
  name           = "${var.environment}-subnet-a"
  zone           = "ru-central1-a"
  network_id     = data.yandex_vpc_network.default.id
  v4_cidr_blocks = ["10.10.1.0/24"]
  
  labels = local.common_labels
}

# Создать виртуальную машину
resource "yandex_compute_instance" "web" {
  # count — создать несколько идентичных ресурсов
  count = var.vm_count

  name        = "${local.vm_name}-${count.index}"
  platform_id = "standard-v2"
  zone        = "ru-central1-a"

  resources {
    cores  = 2
    memory = 4  # GB
  }

  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.id
      size     = 20  # GB
      type     = "network-ssd"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-a.id
    nat       = true  # Публичный IP
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa.pub")}"
  }

  labels = local.common_labels

  # Жизненный цикл ресурса
  lifecycle {
    # Не пересоздавать VM при изменении тегов
    ignore_changes = [metadata]
    
    # Создать новый ресурс ПЕРЕД удалением старого
    create_before_destroy = true
  }
}


# ════════════════════════════════════════════════════════════
# 7. OUTPUT BLOCK — выходные значения
# ════════════════════════════════════════════════════════════

output "vm_public_ips" {
  description = "Public IP addresses of VMs"
  value = [
    for instance in yandex_compute_instance.web :
    instance.network_interface[0].nat_ip_address
  ]
}

output "vm_ids" {
  description = "IDs of created VMs"
  value       = yandex_compute_instance.web[*].id
}


# ════════════════════════════════════════════════════════════
# 8. MODULE BLOCK — переиспользуемый модуль
# ════════════════════════════════════════════════════════════

module "kubernetes" {
  source = "./modules/k8s-cluster"

  cluster_name = "${var.environment}-k8s"
  subnet_id    = yandex_vpc_subnet.subnet-a.id
  node_count   = 3
}
```

---

### Структура файлов проекта

text

```
my-terraform-project/
│
├── main.tf              # Основная конфигурация (ресурсы)
├── variables.tf         # Определение переменных
├── outputs.tf           # Выходные значения
├── versions.tf          # Версии Terraform и провайдеров
├── terraform.tfvars     # Значения переменных (НЕ коммитить если секреты!)
├── provider.tf          # Конфигурация провайдеров (опционально)
│
├── modules/             # Переиспользуемые модули
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── k8s-cluster/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
│
├── environments/        # Конфигурации для разных окружений
│   ├── dev/
│   │   ├── main.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   │   ├── main.tf
│   │   └── terraform.tfvars
│   └── prod/
│       ├── main.tf
│       └── terraform.tfvars
│
├── .terraform/          # Скачанные провайдеры (генерируется автоматически)
├── .terraform.lock.hcl  # Lock-файл версий провайдеров
├── terraform.tfstate    # State-файл (НЕ коммитить в Git!)
└── .gitignore           # Git ignore правила
```

**Best Practices для организации файлов:**

hcl

```
# ════════════════════════════════════════════════════════════
# versions.tf — Версии и провайдеры
# ════════════════════════════════════════════════════════════

terraform {
  required_version = ">= 1.6"

  required_providers {
    yandex = {
      source  = "yandex-cloud/yandex"
      version = "~> 0.100"
    }
  }
}


# ════════════════════════════════════════════════════════════
# variables.tf — Определение переменных
# ════════════════════════════════════════════════════════════

variable "cloud_id" {
  description = "Yandex Cloud ID"
  type        = string
}

variable "folder_id" {
  description = "Yandex Cloud Folder ID"
  type        = string
}

variable "vm_cores" {
  description = "Number of CPU cores"
  type        = number
  default     = 2
}


# ════════════════════════════════════════════════════════════
# terraform.tfvars — Значения переменных
# ════════════════════════════════════════════════════════════

cloud_id  = "b1g1234567890abcdef"
folder_id = "b1g0987654321fedcba"
vm_cores  = 4


# ════════════════════════════════════════════════════════════
# outputs.tf — Выходные значения
# ════════════════════════════════════════════════════════════

output "vm_ip" {
  description = "Public IP of the VM"
  value       = yandex_compute_instance.web.network_interface[0].nat_ip_address
}


# ════════════════════════════════════════════════════════════
# .gitignore
# ════════════════════════════════════════════════════════════

# State файлы (содержат секреты!)
*.tfstate
*.tfstate.*

# Провайдеры (скачиваются при terraform init)
.terraform/

# Переменные с секретами
terraform.tfvars
*.auto.tfvars

# Crash logs
crash.log

# CLI конфигурация
.terraformrc
terraform.rc
```

---

## 4. Жизненный цикл Terraform

### terraform init — инициализация

**terraform init** — первая команда, которую нужно выполнить в новом Terraform-проекте. Она **подготавливает рабочую директорию**.

Bash

```
# ════════════════════════════════════════════════════════════
# ИНИЦИАЛИЗАЦИЯ
# ════════════════════════════════════════════════════════════

cd my-terraform-project
terraform init

# Что происходит:
# ✅ Скачивает провайдеры (yandex-cloud/yandex)
# ✅ Инициализирует backend (если настроен)
# ✅ Скачивает модули (если используются)
# ✅ Создаёт .terraform/ директорию
# ✅ Создаёт .terraform.lock.hcl (lock-файл версий)
```

**Что делает terraform init:**

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                   ЧТО ДЕЛАЕТ TERRAFORM INIT                          ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  1. СКАЧИВАНИЕ ПРОВАЙДЕРОВ                                           ║
║     ════════════════════════                                         ║
║                                                                      ║
║     terraform {                                                      ║
║       required_providers {                                           ║
║         yandex = {                                                   ║
║           source  = "yandex-cloud/yandex"                            ║
║           version = "~> 0.100"                                       ║
║         }                                                            ║
║       }                                                              ║
║     }                                                                ║
║                                                                      ║
║     → Terraform скачивает плагин провайдера с registry.terraform.io  ║
║     → Сохраняет в .terraform/providers/                              ║
║                                                                      ║
║  ────────────────────────────────────────────────────────            ║
║                                                                      ║
║  2. ИНИЦИАЛИЗАЦИЯ BACKEND                                            ║
║     ═══════════════════════                                          ║
║                                                                      ║
║     backend "s3" {                                                   ║
║       bucket = "my-terraform-state"                                  ║
║       key    = "terraform.tfstate"                                   ║
║     }                                                                ║
║                                                                      ║
║     → Настраивает хранилище для state-файла                          ║
║     → Проверяет доступ к backend                                     ║
║                                                                      ║
║  ────────────────────────────────────────────────────────            ║
║                                                                      ║
║  3. СКАЧИВАНИЕ МОДУЛЕЙ                                               ║
║     ════════════════════                                             ║
║                                                                      ║
║     module "vpc" {                                                   ║
║       source = "terraform-aws-modules/vpc/aws"                       ║
║     }                                                                ║
║                                                                      ║
║     → Скачивает модули из Terraform Registry или Git                 ║
║     → Сохраняет в .terraform/modules/                                ║
║                                                                      ║
║  ────────────────────────────────────────────────────────            ║
║                                                                      ║
║  4. СОЗДАНИЕ LOCK-ФАЙЛА                                              ║
║     ═════════════════════                                            ║
║                                                                      ║
║     .terraform.lock.hcl — фиксирует ТОЧНЫЕ версии провайдеров        ║
║                                                                      ║
║     Зачем: чтобы команда использовала одинаковые версии              ║
║     (как package-lock.json в npm)                                    ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Флаги terraform init:**

Bash

```
# Обновить провайдеры до последних версий
terraform init -upgrade

# Переконфигурировать backend
terraform init -reconfigure

# Миграция state в новый backend
terraform init -migrate-state

# Без интерактивных вопросов
terraform init -input=false
```

---

### State-файл — сердце Terraform

**terraform.tfstate** — это **JSON-файл**, который содержит **текущее состояние** вашей инфраструктуры. Это mapping между **Terraform конфигурацией** и **реальными ресурсами** в облаке.

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                    ЗАЧЕМ НУЖЕН STATE-ФАЙЛ?                           ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  БЕЗ STATE:                                                          ║
║  ═════════                                                           ║
║                                                                      ║
║  Вы запускаете terraform apply несколько раз:                        ║
║                                                                      ║
║  1-й раз: создаёт VM                                                 ║
║  2-й раз: создаёт ЕЩЁ ОДНУ VM (не знает, что уже создано!)          ║
║  3-й раз: создаёт ЕЩЁ ОДНУ VM                                        ║
║                                                                      ║
║  → Дубликаты, хаос, невозможно управлять                             ║
║                                                                      ║
║  ────────────────────────────────────────────────────────            ║
║                                                                      ║
║  С STATE:                                                            ║
║  ════════                                                            ║
║                                                                      ║
║  1-й раз: создаёт VM → записывает в state "VM с ID=abc123"          ║
║  2-й раз: проверяет state → видит "VM уже есть" → No changes         ║
║  3-й раз: No changes                                                 ║
║                                                                      ║
║  → Идемпотентность ✅                                                ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Что хранится в state:**

JSON

```
{
  "version": 4,
  "terraform_version": "1.6.0",
  "resources": [
    {
      "type": "yandex_compute_instance",
      "name": "web",
      "provider": "provider[\"registry.terraform.io/yandex-cloud/yandex\"]",
      "instances": [
        {
          "attributes": {
            "id": "fhm1234567890abcdef",
            "name": "web-server",
            "zone": "ru-central1-a",
            "platform_id": "standard-v2",
            "resources": {
              "cores": 2,
              "memory": 4
            },
            "network_interface": [
              {
                "subnet_id": "e9b12345678901234567",
                "nat_ip_address": "51.250.10.20",
                "ip_address": "10.10.1.5"
              }
            ]
          }
        }
      ]
    }
  ]
}
```

**Почему state важен:**

text

```
1. MAPPING конфигурация → реальные ресурсы
   resource "yandex_compute_instance" "web" {...}
                    ▼
   VM с ID fhm1234567890abcdef в Yandex Cloud

2. METADATA о ресурсах
   • ID
   • IP-адреса
   • Зависимости между ресурсами

3. PERFORMANCE
   • Без state: Terraform должен запрашивать облако для КАЖДОГО ресурса
   • С state: Terraform знает всё локально → быстро

4. COLLABORATION
   • Команда из 10 человек работает с одним state
   • Все видят текущее состояние инфраструктуры
```

**Remote State (удалённое хранение):**

hcl

```
# ⚠️  ПРОБЛЕМА локального state:
# • Файл на вашем компьютере → команда НЕ видит изменений
# • Потеря компьютера = потеря state = катастрофа
# • Конфликты при одновременной работе

# ✅ РЕШЕНИЕ: Remote Backend

terraform {
  backend "s3" {
    endpoint   = "storage.yandexcloud.net"
    bucket     = "my-terraform-state"
    key        = "prod/terraform.tfstate"
    region     = "ru-central1"

    # Credentials
    access_key = var.access_key
    secret_key = var.secret_key

    # State locking (предотвращает одновременное изменение)
    dynamodb_table = "terraform-locks"  # для AWS
    # Для Yandex Cloud используйте Yandex Database для locks
  }
}
```

**State locking:**

text

```
Проблема: два разработчика запускают terraform apply одновременно

БЕЗ LOCKING:
  Dev 1: terraform apply → создаёт ресурсы
  Dev 2: terraform apply → создаёт ресурсы (одновременно!)
  → Конфликт, race condition, непредсказуемое поведение

С LOCKING:
  Dev 1: terraform apply → захватывает lock
  Dev 2: terraform apply → ЖДЁТ (lock занят)
  → Dev 1 завершается → lock освобождается
  → Dev 2 применяет изменения
  → Последовательное выполнение ✅
```

---

### terraform plan — предпросмотр изменений

Bash

```
# ════════════════════════════════════════════════════════════
# ПЛАНИРОВАНИЕ ИЗМЕНЕНИЙ
# ════════════════════════════════════════════════════════════

terraform plan

# Что делает:
# 1. Сравнивает конфигурацию с текущим state
# 2. Запрашивает облако для проверки РЕАЛЬНОГО состояния
# 3. Показывает: что будет создано (+), изменено (~), удалено (-)
# 4. НЕ ПРИМЕНЯЕТ изменения (read-only)
```

**Пример вывода:**

text

```
Terraform will perform the following actions:

  # yandex_compute_instance.web will be created
  + resource "yandex_compute_instance" "web" {
      + created_at    = (known after apply)
      + id            = (known after apply)
      + name          = "web-server"
      + platform_id   = "standard-v2"
      + zone          = "ru-central1-a"

      + resources {
          + cores  = 2
          + memory = 4
        }

      + boot_disk {
          + disk_id = (known after apply)
          
          + initialize_params {
              + image_id = "fd8abcdefg1234567890"
              + size     = 20
              + type     = "network-ssd"
            }
        }
    }

  # yandex_vpc_subnet.subnet-a will be created
  + resource "yandex_vpc_subnet" "subnet-a" {
      + id             = (known after apply)
      + name           = "subnet-a"
      + network_id     = "enpabcdef1234567890"
      + v4_cidr_blocks = ["10.10.1.0/24"]
      + zone           = "ru-central1-a"
    }

Plan: 2 to add, 0 to change, 0 to destroy.
```

**Символы в terraform plan:**

text

```
+ create        — ресурс будет СОЗДАН
~ update        — ресурс будет ИЗМЕНЁН (in-place или with recreation)
- destroy       — ресурс будет УДАЛЁН
-/+ replace     — ресурс будет УДАЛЁН и ПЕРЕСОЗДАН
<= read         — data source будет ПРОЧИТАН
```

**Сохранить план для apply:**

Bash

```
# Сохранить план в файл
terraform plan -out=tfplan

# Применить ТОЧНО этот план (без пересчёта)
terraform apply tfplan
```

---

### terraform apply — создание ресурсов

Bash

```
# ════════════════════════════════════════════════════════════
# ПРИМЕНЕНИЕ ИЗМЕНЕНИЙ
# ════════════════════════════════════════════════════════════

terraform apply

# Интерактивно:
# 1. Показывает план
# 2. Спрашивает: "Do you want to perform these actions? yes/no"
# 3. Ждёт ввода "yes"
# 4. Применяет изменения
# 5. Обновляет state-файл

# Автоматически (для CI/CD):
terraform apply -auto-approve

# С конкретными переменными
terraform apply -var="vm_count=5" -var="environment=prod"

# Применить только конкретный ресурс
terraform apply -target=yandex_compute_instance.web
```

**Процесс apply:**

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                   ЧТО ПРОИСХОДИТ ПРИ APPLY                           ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  1. Refresh State                                                    ║
║     • Terraform запрашивает РЕАЛЬНОЕ состояние ресурсов из облака    ║
║     • Сравнивает с state-файлом                                      ║
║     • Обнаруживает drift (расхождения)                               ║
║                                                                      ║
║  2. Plan                                                             ║
║     • Вычисляет какие изменения нужны                                ║
║     • Строит граф зависимостей                                       ║
║                                                                      ║
║  3. Execution (последовательное выполнение)                          ║
║     • Создаёт ресурсы в правильном порядке:                          ║
║       1. VPC network                                                 ║
║       2. Subnet (зависит от network)                                 ║
║       3. VM (зависит от subnet)                                      ║
║                                                                      ║
║  4. Update State                                                     ║
║     • Записывает ID созданных ресурсов в state                       ║
║     • Обновляет метаданные (IP-адреса, статусы)                      ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

### terraform destroy — удаление ресурсов

Bash

```
# ════════════════════════════════════════════════════════════
# УДАЛЕНИЕ ВСЕХ РЕСУРСОВ
# ════════════════════════════════════════════════════════════

terraform destroy

# Интерактивно спросит подтверждение
# Введите "yes" для удаления

# Автоматически (ОПАСНО!)
terraform destroy -auto-approve

# Удалить только конкретный ресурс
terraform destroy -target=yandex_compute_instance.web

# С переменными
terraform destroy -var-file="prod.tfvars"
```

**Порядок удаления:**

text

```
Terraform удаляет в ОБРАТНОМ порядке зависимостей:

Создание:
  1. Network
  2. Subnet
  3. VM

Удаление:
  1. VM          ← сначала удаляет зависимые
  2. Subnet      ← потом промежуточные
  3. Network     ← в конце базовые
```

---

## 5. Провайдеры Terraform

### Что такое провайдер?

**Provider (Провайдер)** — это плагин Terraform, который **взаимодействует с API** конкретного облачного провайдера или сервиса.

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                   РОЛЬ ПРОВАЙДЕРА                                    ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  Terraform (ядро)                                                    ║
║       ↓                                                              ║
║  Provider Plugin (yandex-cloud/yandex)                               ║
║       ↓                                                              ║
║  Yandex Cloud API                                                    ║
║       ↓                                                              ║
║  Создание/изменение/удаление ресурсов                                ║
║                                                                      ║
║  • Terraform НИЧЕГО не знает про Yandex Cloud                        ║
║  • Провайдер переводит HCL → API-вызовы                              ║
║  • Провайдер возвращает результаты → Terraform обновляет state       ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Популярные провайдеры:**

text

```
┌────────────────────────────────────────────────────────────────────┐
│ AWS          → hashicorp/aws          (Amazon Web Services)        │
│ Azure        → hashicorp/azurerm      (Microsoft Azure)            │
│ GCP          → hashicorp/google       (Google Cloud Platform)      │
│ Yandex Cloud → yandex-cloud/yandex                                 │
│ Kubernetes   → hashicorp/kubernetes                                │
│ Docker       → kreuzwerker/docker                                  │
│ GitHub       → integrations/github                                 │
│ Cloudflare   → cloudflare/cloudflare                               │
│ Datadog      → datadog/datadog                                     │
│ Random       → hashicorp/random       (генератор случайных строк)  │
│ Time         → hashicorp/time         (задержки, таймеры)          │
│ Null         → hashicorp/null         (helper ресурсы)             │
└────────────────────────────────────────────────────────────────────┘

Всего 3000+ провайдеров на registry.terraform.io
```

---

## 6. Создание ресурсов в Yandex Cloud с помощью Terraform

### Полный пример: VM, Network, S3 bucket

hcl

```
# ════════════════════════════════════════════════════════════
# versions.tf
# ════════════════════════════════════════════════════════════

terraform {
  required_version = ">= 1.0"

  required_providers {
    yandex = {
      source  = "yandex-cloud/yandex"
      version = "~> 0.100"
    }
  }
}


# ════════════════════════════════════════════════════════════
# provider.tf
# ════════════════════════════════════════════════════════════

provider "yandex" {
  # Credentials (через переменные окружения безопаснее)
  token     = var.yc_token      # или YC_TOKEN env var
  cloud_id  = var.cloud_id      # или YC_CLOUD_ID
  folder_id = var.folder_id     # или YC_FOLDER_ID
  zone      = "ru-central1-a"   # зона по умолчанию
}


# ════════════════════════════════════════════════════════════
# variables.tf
# ════════════════════════════════════════════════════════════

variable "yc_token" {
  description = "Yandex Cloud OAuth token"
  type        = string
  sensitive   = true
}

variable "cloud_id" {
  description = "Yandex Cloud ID"
  type        = string
}

variable "folder_id" {
  description = "Yandex Cloud Folder ID"
  type        = string
}

variable "ssh_public_key" {
  description = "SSH public key for VM access"
  type        = string
  default     = "~/.ssh/id_rsa.pub"
}


# ════════════════════════════════════════════════════════════
# main.tf — СОЗДАНИЕ СЕТИ
# ════════════════════════════════════════════════════════════

# Создать VPC network
resource "yandex_vpc_network" "main" {
  name        = "main-network"
  description = "Main network for the project"
  
  labels = {
    environment = "production"
    managed_by  = "terraform"
  }
}

# Создать подсеть в зоне ru-central1-a
resource "yandex_vpc_subnet" "subnet-a" {
  name           = "subnet-a"
  description    = "Subnet in zone A"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.main.id
  v4_cidr_blocks = ["10.10.1.0/24"]
  
  labels = {
    environment = "production"
  }
}

# Создать подсеть в зоне ru-central1-b (для HA)
resource "yandex_vpc_subnet" "subnet-b" {
  name           = "subnet-b"
  zone           = "ru-central1-b"
  network_id     = yandex_vpc_network.main.id
  v4_cidr_blocks = ["10.10.2.0/24"]
}


# ════════════════════════════════════════════════════════════
# DATA SOURCES — получение существующих данных
# ════════════════════════════════════════════════════════════

# Получить последний образ Ubuntu 20.04
data "yandex_compute_image" "ubuntu" {
  family = "ubuntu-2004-lts"
}


# ════════════════════════════════════════════════════════════
# СОЗДАНИЕ ВИРТУАЛЬНОЙ МАШИНЫ
# ════════════════════════════════════════════════════════════

resource "yandex_compute_instance" "web" {
  name        = "web-server-1"
  description = "Web server for production"
  platform_id = "standard-v2"
  zone        = "ru-central1-a"

  # Вычислительные ресурсы
  resources {
    cores         = 2      # vCPU
    memory        = 4      # GB RAM
    core_fraction = 100    # 100% CPU (не прерываемая VM)
  }

  # Загрузочный диск
  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.id
      size     = 20   # GB
      type     = "network-ssd"
    }
  }

  # Сетевой интерфейс
  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-a.id
    nat       = true  # Публичный IP
    
    # Можно указать конкретный IP (если зарезервирован)
    # nat_ip_address = yandex_vpc_address.static_ip.external_ipv4_address[0].address
  }

  # Метаданные (cloud-init)
  metadata = {
    # SSH ключ для доступа
    ssh-keys = "ubuntu:${file(var.ssh_public_key)}"
    
    # Cloud-init скрипт
    user-data = <<-EOF
      #cloud-config
      packages:
        - nginx
        - docker.io
      runcmd:
        - systemctl start nginx
        - systemctl enable nginx
    EOF
  }

  # Планирование (прерываемая VM для экономии)
  scheduling_policy {
    preemptible = false  # true = прерываемая (дешевле на 50%)
  }

  labels = {
    environment = "production"
    role        = "web-server"
  }
}


# ════════════════════════════════════════════════════════════
# СОЗДАНИЕ НЕСКОЛЬКИХ VM (через count)
# ════════════════════════════════════════════════════════════

resource "yandex_compute_instance" "worker" {
  count = 3  # Создать 3 идентичные VM

  name        = "worker-${count.index + 1}"
  platform_id = "standard-v2"
  zone        = "ru-central1-a"

  resources {
    cores  = 2
    memory = 4
  }

  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.id
      size     = 20
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-a.id
    nat       = true
  }

  metadata = {
    ssh-keys = "ubuntu:${file(var.ssh_public_key)}"
  }
}


# ════════════════════════════════════════════════════════════
# СОЗДАНИЕ ДОПОЛНИТЕЛЬНОГО ДИСКА
# ════════════════════════════════════════════════════════════

resource "yandex_compute_disk" "data" {
  name = "data-disk"
  type = "network-ssd"
  zone = "ru-central1-a"
  size = 100  # GB

  labels = {
    environment = "production"
  }
}

# Подключить диск к VM
resource "yandex_compute_instance" "db" {
  name = "database-server"
  # ... (базовые настройки)

  # Дополнительный диск
  secondary_disk {
    disk_id     = yandex_compute_disk.data.id
    auto_delete = false  # НЕ удалять диск при удалении VM
  }
}


# ════════════════════════════════════════════════════════════
# СОЗДАНИЕ S3 BUCKET (Object Storage)
# ════════════════════════════════════════════════════════════

# Сначала нужен сервисный аккаунт
resource "yandex_iam_service_account" "storage_sa" {
  name        = "storage-sa"
  description = "Service account for Object Storage"
}

# Назначить роль storage.admin
resource "yandex_resourcemanager_folder_iam_member" "storage_admin" {
  folder_id = var.folder_id
  role      = "storage.admin"
  member    = "serviceAccount:${yandex_iam_service_account.storage_sa.id}"
}

# Создать ключ доступа (access key)
resource "yandex_iam_service_account_static_access_key" "storage_key" {
  service_account_id = yandex_iam_service_account.storage_sa.id
  description        = "Static access key for Object Storage"
}

# Создать S3 bucket
resource "yandex_storage_bucket" "static" {
  bucket     = "my-static-website-${random_string.suffix.result}"
  access_key = yandex_iam_service_account_static_access_key.storage_key.access_key
  secret_key = yandex_iam_service_account_static_access_key.storage_key.secret_key

  # Настройки bucket
  acl = "public-read"  # Публичный доступ на чтение

  # Хостинг статического сайта
  website {
    index_document = "index.html"
    error_document = "error.html"
  }

  # CORS
  cors_rule {
    allowed_headers = ["*"]
    allowed_methods = ["GET", "HEAD"]
    allowed_origins = ["*"]
    max_age_seconds = 3600
  }
}

# Случайный суффикс для уникального имени bucket
resource "random_string" "suffix" {
  length  = 8
  special = false
  upper   = false
}


# ════════════════════════════════════════════════════════════
# СОЗДАНИЕ MANAGED POSTGRESQL
# ════════════════════════════════════════════════════════════

resource "yandex_mdb_postgresql_cluster" "main" {
  name        = "postgres-cluster"
  environment = "PRODUCTION"
  network_id  = yandex_vpc_network.main.id

  config {
    version = "15"
    resources {
      resource_preset_id = "s2.micro"  # 2 vCPU, 8GB RAM
      disk_type_id       = "network-ssd"
      disk_size          = 20  # GB
    }
  }

  # Хосты БД в разных зонах (HA)
  host {
    zone      = "ru-central1-a"
    subnet_id = yandex_vpc_subnet.subnet-a.id
  }

  host {
    zone      = "ru-central1-b"
    subnet_id = yandex_vpc_subnet.subnet-b.id
  }
}

# Создать базу данных
resource "yandex_mdb_postgresql_database" "app_db" {
  cluster_id = yandex_mdb_postgresql_cluster.main.id
  name       = "app_production"
  owner      = yandex_mdb_postgresql_user.app_user.name
}

# Создать пользователя БД
resource "yandex_mdb_postgresql_user" "app_user" {
  cluster_id = yandex_mdb_postgresql_cluster.main.id
  name       = "app_user"
  password   = var.db_password  # Через переменную (НЕ хардкодить!)

  permission {
    database_name = yandex_mdb_postgresql_database.app_db.name
  }
}


# ════════════════════════════════════════════════════════════
# outputs.tf — ВЫХОДНЫЕ ЗНАЧЕНИЯ
# ════════════════════════════════════════════════════════════

output "web_server_public_ip" {
  description = "Public IP of web server"
  value       = yandex_compute_instance.web.network_interface[0].nat_ip_address
}

output "worker_ips" {
  description = "Public IPs of worker VMs"
  value = [
    for instance in yandex_compute_instance.worker :
    instance.network_interface[0].nat_ip_address
  ]
}

output "storage_bucket_url" {
  description = "URL of the storage bucket"
  value       = "https://${yandex_storage_bucket.static.bucket}.storage.yandexcloud.net"
}

output "postgres_connection" {
  description = "PostgreSQL connection info"
  value = {
    host = yandex_mdb_postgresql_cluster.main.host[0].fqdn
    port = 6432
    database = yandex_mdb_postgresql_database.app_db.name
    user = yandex_mdb_postgresql_user.app_user.name
  }
  sensitive = true  # НЕ показывать в логах
}
```

---

### Применение конфигурации

Bash

```
# ════════════════════════════════════════════════════════════
# ШАГ 1: Подготовить переменные
# ════════════════════════════════════════════════════════════

# terraform.tfvars
cat > terraform.tfvars <<EOF
yc_token    = "AQAAAAAA..."  # OAuth token
cloud_id    = "b1g1234567890abcdef"
folder_id   = "b1g0987654321fedcba"
db_password = "SuperSecretPassword123!"
EOF


# ════════════════════════════════════════════════════════════
# ШАГ 2: Инициализация
# ════════════════════════════════════════════════════════════

terraform init

# Вывод:
# Initializing provider plugins...
# - Finding yandex-cloud/yandex versions matching "~> 0.100"...
# - Installing yandex-cloud/yandex v0.100.0...
# Terraform has been successfully initialized!


# ════════════════════════════════════════════════════════════
# ШАГ 3: Проверить план
# ════════════════════════════════════════════════════════════

terraform plan

# Вывод покажет:
# Plan: 15 to add, 0 to change, 0 to destroy.


# ════════════════════════════════════════════════════════════
# ШАГ 4: Применить изменения
# ════════════════════════════════════════════════════════════

terraform apply

# Вводим "yes"
# Terraform создаёт ресурсы (занимает 3-5 минут)

# Вывод:
# Apply complete! Resources: 15 added, 0 changed, 0 destroyed.
#
# Outputs:
# web_server_public_ip = "51.250.10.20"
# worker_ips = ["51.250.10.21", "51.250.10.22", "51.250.10.23"]
# storage_bucket_url = "https://my-static-website-abc123.storage.yandexcloud.net"


# ════════════════════════════════════════════════════════════
# ШАГ 5: Проверить созданные ресурсы
# ════════════════════════════════════════════════════════════

# SSH на web-сервер
ssh ubuntu@51.250.10.20

# Проверить nginx
curl http://51.250.10.20

# Посмотреть outputs
terraform output

# Конкретный output
terraform output web_server_public_ip


# ════════════════════════════════════════════════════════════
# ШАГ 6: Изменить инфраструктуру
# ════════════════════════════════════════════════════════════

# Изменить main.tf (например, увеличить RAM до 8GB)
# resources {
#   cores  = 2
#   memory = 8  # было 4
# }

terraform plan
# Plan: 0 to add, 1 to change, 0 to destroy.

terraform apply
# Terraform пересоздаст VM с новыми параметрами


# ════════════════════════════════════════════════════════════
# ШАГ 7: Удалить всё
# ════════════════════════════════════════════════════════════

terraform destroy

# Вводим "yes"
# Terraform удаляет все ресурсы в правильном порядке
```

---

### Best Practices для Terraform

text

```
╔══════════════════════════════════════════════════════════════════════╗
║                    BEST PRACTICES                                    ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  1. НИКОГДА не коммитьте в Git:                                      ║
║     • terraform.tfstate (содержит секреты и ID ресурсов)             ║
║     • terraform.tfvars (если есть пароли)                            ║
║     • *.auto.tfvars (если есть секреты)                              ║
║     • .terraform/ (скачанные провайдеры)                             ║
║                                                                      ║
║  2. ИСПОЛЬЗУЙТЕ Remote Backend (S3, Terraform Cloud)                 ║
║     • Командная работа                                               ║
║     • State locking                                                  ║
║     • Backup state                                                   ║
║                                                                      ║
║  3. ПЕРЕМЕННЫЕ окружения для секретов:                               ║
║     export TF_VAR_yc_token="AQAAA..."                                ║
║     export TF_VAR_db_password="..."                                  ║
║                                                                      ║
║  4. ИСПОЛЬЗУЙТЕ Modules для переиспользования:                       ║
║     module "vpc" { source = "./modules/vpc" }                        ║
║                                                                      ║
║  5. ИМЕНОВАНИЕ ресурсов:                                             ║
║     resource "yandex_compute_instance" "web_server" {                ║
║       name = "${var.environment}-web-${count.index}"                 ║
║     }                                                                ║
║                                                                      ║
║  6. ВСЕГДА запускайте terraform plan перед apply                     ║
║                                                                      ║
║  7. ИСПОЛЬЗУЙТЕ версионирование провайдеров:                         ║
║     version = "~> 0.100"  (только патч-обновления)                   ║
║                                                                      ║
║  8. ТЕГИ/LABELS на всех ресурсах:                                    ║
║     labels = {                                                       ║
║       environment = "prod"                                           ║
║       managed_by  = "terraform"                                      ║
║       cost_center = "engineering"                                    ║
║     }                                                                ║
║                                                                      ║
║  9. ДОКУМЕНТИРУЙТЕ переменные:                                       ║
║     variable "vm_cores" {                                            ║
║       description = "Number of vCPU cores"                           ║
║       type        = number                                           ║
║       validation { ... }                                             ║
║     }                                                                ║
║                                                                      ║
║  10. ИСПОЛЬЗУЙТЕ terraform fmt для форматирования                    ║
║      terraform validate для проверки синтаксиса                      ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```