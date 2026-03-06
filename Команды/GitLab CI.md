```
# Локальная валидация (требует gitlab-runner)
gitlab-runner exec docker test-job   # Выполнить job локально

# Работа с pipeline (через UI или API)
# Основные переменные окружения в .gitlab-ci.yml:
CI_COMMIT_SHA                         # Хеш коммита
CI_COMMIT_REF_NAME                    # Имя ветки/тега
CI_PIPELINE_ID                        # ID pipeline
CI_JOB_ID                             # ID job
CI_REGISTRY                           # URL registry
CI_REGISTRY_IMAGE                     # Полное имя образа
```