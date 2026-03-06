```
# Инициализация и планирование
terraform init                        # Инициализировать (скачать providers)
terraform init -upgrade               # Обновить providers
terraform plan                        # Посмотреть изменения (dry-run)
terraform plan -out=tfplan            # Сохранить план в файл

# Применение изменений
terraform apply                       # Применить изменения (спросит yes)
terraform apply -auto-approve         # Без подтверждения
terraform apply tfplan                # Применить сохранённый план
terraform apply -target=resource.name # Только конкретный ресурс
terraform apply -var="env=prod"       # Передать переменную
terraform apply -var-file="prod.tfvars"

# Удаление
terraform destroy                     # Удалить все ресурсы
terraform destroy -auto-approve       # Без подтверждения
terraform destroy -target=resource.name

# State (состояние)
terraform state list                  # Список ресурсов в state
terraform state show resource.name    # Информация о ресурсе
terraform state pull                  # Скачать state (JSON)
terraform state rm resource.name      # Удалить из state (не из облака!)
terraform state mv old.name new.name  # Переименовать в state

# Форматирование и валидация
terraform fmt                         # Отформатировать .tf файлы
terraform fmt -recursive              # Рекурсивно
terraform validate                    # Проверить синтаксис
terraform validate -json              # Вывод в JSON

# Вывод outputs
terraform output                      # Все outputs
terraform output vm_ip                # Конкретный output
terraform output -json                # В JSON формате

# Workspaces (окружения)
terraform workspace list              # Список workspaces
terraform workspace new dev           # Создать workspace
terraform workspace select prod       # Переключиться
terraform workspace delete dev        # Удалить

# Граф зависимостей
terraform graph | dot -Tpng > graph.png  # Визуализация

# Import существующих ресурсов
terraform import aws_instance.web i-1234567890abcdef0

# Refresh (обновить state из реального состояния)
terraform refresh

# Консоль (интерактивная Terraform консоль)
terraform console
```