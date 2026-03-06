```
# Инициализация
yc init                               # Первоначальная настройка

# Конфигурация
yc config list                        # Текущие настройки
yc config set folder-id <id>          # Установить folder
yc config set cloud-id <id>           # Установить cloud
yc config profile list                # Список профилей
yc config profile create prod         # Создать профиль
yc config profile activate prod       # Активировать профиль

# Compute (VM)
yc compute instance list              # Список VM
yc compute instance get <name>        # Информация о VM
yc compute instance create \
  --name web-server \
  --zone ru-central1-a \
  --cores 2 \
  --memory 4 \
  --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-2004-lts

yc compute instance start <name>      # Запустить VM
yc compute instance stop <name>       # Остановить VM
yc compute instance restart <name>    # Перезапустить VM
yc compute instance delete <name>     # Удалить VM

yc compute image list                 # Список образов
yc compute disk list                  # Список дисков
yc compute snapshot list              # Список снапшотов

# Managed Kubernetes
yc managed-kubernetes cluster list    # Список K8s кластеров
yc managed-kubernetes cluster get <name>
yc managed-kubernetes cluster get-credentials <name> --external
yc managed-kubernetes node-group list --cluster-name=<name>

# Container Registry
yc container registry list            # Список registry
yc container image list --registry-name=<name>
yc container registry configure-docker  # Настроить Docker

# Object Storage (S3)
yc storage bucket list                # Список buckets (требует access key)

# VPC (сети)
yc vpc network list                   # Список сетей
yc vpc subnet list                    # Список подсетей
yc vpc address list                   # Список адресов

# IAM (доступы)
yc iam service-account list           # Список сервисных аккаунтов
yc iam service-account create --name sa-name
yc iam key create --service-account-name sa-name --output key.json
yc iam access-key create --service-account-name sa-name  # Для S3

# Resource Manager
yc resource-manager folder list       # Список каталогов
yc resource-manager cloud list        # Список облаков

# IAM-токен
yc iam create-token                   # Получить IAM-токен (12 часов)

# Форматирование вывода
yc compute instance list --format json
yc compute instance list --format yaml
yc compute instance list --format table
```