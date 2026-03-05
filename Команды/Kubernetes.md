### Работа с Pod'ами
1. **`kubectl get pods`** — Показать все поды в текущем namespace.
2. **`kubectl get pods -A`** — Показать поды во всех namespace.
3. **`kubectl get pod <pod-name> -o wide`** — Показать под с дополнительной информацией (IP, узел).
4. **`kubectl describe pod <pod-name>`** — Детальная информация о поде (события, ошибки).
5. **`kubectl logs <pod-name>`** — Посмотреть логи пода.
6. **`kubectl logs -f <pod-name>`** — Стримить логи в реальном времени.
7. **`kubectl logs <pod-name> -c <container-name>`** — Логи конкретного контейнера в поде (если их несколько).
8. **`kubectl exec -it <pod-name> -- /bin/bash`** — Зайти внутрь контейнера пода (интерактивный режим).
9. **`kubectl exec <pod-name> -- <command>`** — Выполнить команду внутри пода.
10. **`kubectl port-forward <pod-name> 8080:80`** — Пробросить порт с пода на локальную машину.
11. **`kubectl delete pod <pod-name>`** — Удалить под.
12. **`kubectl run <pod-name> --image=<image>`** — Быстро создать и запустить под (для тестов).
### Работа с Deployment (Деплойменты)
13. **`kubectl get deployments`** — Список деплойментов.
14. **`kubectl describe deployment <name>`** — Информация о деплойменте.
15. **`kubectl create deployment <name> --image=<image>`** — Создать деплоймент.
16. **`kubectl scale deployment <name> --replicas=3`** — Изменить количество реплик.
17. **`kubectl set image deployment/<name> <container>=<new-image>`** — Обновить образ контейнера в деплойменте.
18. **`kubectl rollout status deployment/<name>`** — Проверить статус обновления (rollout).
19. **`kubectl rollout history deployment/<name>`** — История версий деплоймента.
20. **`kubectl rollout undo deployment/<name>`** — Откатить последнее обновление.
21. **`kubectl delete deployment <name>`** — Удалить деплоймент.

### Работа с сервисами (Services)
22. **`kubectl get services`** — Список сервисов.
23. **`kubectl get svc`** — Сокращенная версия (svc = services).
24. **`kubectl expose deployment <name> --port=80 --target-port=8080 --type=NodePort`** — Создать сервис для деплоймента.
25. **`kubectl describe service <name>`** — Информация о сервисе.
26. **`kubectl delete service <name>`** — Удалить сервис.

### Работа с Namespace
27. **`kubectl get namespaces`** — Список namespace.
28. **`kubectl config set-context --current --namespace=<name>`** — Сменить текущий namespace.
29. **`kubectl create namespace <name>`** — Создать namespace.
30. **`kubectl delete namespace <name>`** — Удалить namespace (и всё внутри него!).

### Работа с ConfigMap и Secrets
31. **`kubectl get configmaps`** — Список конфигмапов.
32. **`kubectl create configmap <name> --from-literal=key=value`** — Создать ConfigMap из литерала.
33. **`kubectl create configmap <name> --from-file=<file>`** — Создать ConfigMap из файла.
34. **`kubectl get secrets`** — Список секретов.
35. **`kubectl create secret generic <name> --from-literal=password=123`** — Создать секрет.
36. **`kubectl describe secret <name>`** — Информация о секрете (но не само значение).

### Управление кластером и узлами
37. **`kubectl get nodes`** — Список узлов кластера.
38. **`kubectl describe node <node-name>`** — Информация об узле (ресурсы, поды на нем).
39. **`kubectl top node`** — Показать загрузку узлов (CPU/Memory) — требует metrics-server.
40. **`kubectl top pod`** — Показать загрузку подов.
41. **`kubectl cluster-info`** — Информация о кластере (где мастер, где DNS).
42. **`kubectl api-resources`** — Список всех доступных типов ресурсов в кластере.

### Работа с YAML манифестами
43. **`kubectl apply -f <file.yaml>`** — Применить конфигурацию из файла (создать или обновить).
44. **`kubectl create -f <file.yaml>`** — Создать ресурсы из файла (ошибка, если уже есть).
45. **`kubectl delete -f <file.yaml>`** — Удалить все, что описано в файле.
46. **`kubectl get pod <name> -o yaml`** — Посмотреть YAML описание существующего пода.
47. **`kubectl edit deployment <name>`** — Отредактировать ресурс прямо в редакторе (vim/nano).

### Дополнительные полезные команды
48. **`kubectl get events --sort-by='.lastTimestamp'`** — Посмотреть события кластера (ошибки, старты).
49. **`kubectl cp <pod-name>:/path/to/file ./local-file`** — Скопировать файл из пода на локальную машину.
50. **`kubectl explain pods`** — Документация по структуре ресурса (полезно для написания YAML).
51. **`kubectl get pods --all-namespaces --field-selector status.phase=Failed`** — Найти все поды в статусе `Failed` во всем кластере.  
    _(Помогает найти "мусор" или проблемные поды)._
52. **`kubectl delete pods --all --namespace=<name>`** — Удалить **все** поды в namespace. Если поды управляются Deployment, они будут пересозданы.  
    _(Быстрый способ перезапустить все сервисы в неймспейсе)._
53. **`kubectl logs -f --tail=100 -l app=myapp`** — Смотреть логи всех подов с меткой `app=myapp` (последние 100 строк) в реальном времени.
54. **`kubectl get all -n <namespace>`** — Показать **все** основные ресурсы (поды, сервисы, деплойменты) в namespace.  
    _(Хотя `all` не включает secrets/configmaps, это удобный обзор)._
55. **`kubectl exec -it <pod> -- sh -c 'pg_dump dbname' > backup.sql`** — Создать дамп PostgreSQL из пода и сохранить его локально.
56. **`kubectl port-forward service/<service-name> 8080:80`** — Пробросить порт напрямую через сервис (а не через конкретный под), чтобы балансировка работала локально.
57. **`kubectl run curl --image=curlimages/curl -it --rm -- sh`** — Запустить временный под с curl для отладки сети внутри кластера (с автоудалением после выхода).
58. **`kubectl get pods -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName`** — Вывести список подов в виде красивой таблицы с нужными колонками.
59. **`kubectl delete $(kubectl get all -o name | grep service)`** — Удалить все сервисы (осторожно!). Получаем список всех ресурсов, фильтруем те, у которых в имени есть service, и удаляем их.
60. **`kubectl rollout restart deployment/<name>`** — "Перезапустить" все поды в деплойменте без изменения образа. Стратегия: создает новые поды, потом убивает старые.
61.  **`kubectl top pods -n <namespace> --sort-by=cpu`** — Посмотреть, какие поды жрут больше всего CPU в неймспейсе (сортировка).
62. **`kubectl describe quota -n <namespace>`** — Посмотреть, какие квоты ресурсов установлены на неймспейс и сколько уже использовано.
63. **`kubectl get pods -o json | jq '.items[] | {name: .metadata.name, cpu: .spec.containers[].resources.requests.cpu}'`** — Вывести все поды и их CPU requests
64. **`kubectl run -it --rm debug --image=nicolaka/netshoot -- /bin/bash`** — Запустить мощный отладочный под со всеми сетевыми утилитами (ping, curl, tcpdump, dig) и автоматически удалить после выхода.
65. **`kubectl get endpoints <service-name>`** — Проверить, на какие поды реально направляет трафик сервис (endpoints = реальные IP подов за сервисом).
66. **`kubectl exec -it <pod> -- cat /etc/resolv.conf`** — Посмотреть DNS настройки внутри пода (проверка, какой DNS сервер используется).
67. **`kubectl exec -it <pod> -- nslookup <service-name>.<namespace>.svc.cluster.local`** — Проверить DNS resolution внутри кластера.
68. **`kubectl get networkpolicies`** — Посмотреть политики сети (кто с кем может общаться).
69. **`kubectl get pods -o wide --watch`** — Наблюдать в реальном времени, как меняются поды (их статусы и IP).
70. **`kubectl get nodes -o json | jq '.items[] | {name: .metadata.name, allocatable: .status.allocatable}'`** — Посмотреть, сколько ресурсов реально доступно на каждой ноде.
71. **`kubectl exec -it <pod> -- top`** — Зайти в под и посмотреть, какие процессы внутри него жрут ресурсы.
72. **`kubectl cp /dev/null <pod>:/tmp/restart.txt`** — Триггернуть перечитывание конфига в некоторых приложениях (прикол: скопировать пустой файл, а приложение следит за изменениями).
73. **`kubectl get pv`** — Посмотреть все Persistent Volumes в кластере (и их статусы).
74. **`kubectl cordon <node-name>`** — Запретить планировщику ставить новые поды на ноду (перед обслуживанием ноды).
75. **`kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data`** — Аккуратно вытеснить все поды с ноды (перед выводом ноды из кластера).
76. **`kubectl taint nodes <node-name> key=value:NoSchedule`** — Наложить "taint" на ноду, чтобы определенные поды туда не попадали (или попадали только tolerant).
77. **`kubectl get pod -o wide | grep -v Running`** — Найти все поды, которые не в статусе Running.
78. **`helm list -a --all-namespaces`** — Показать все релизы Helm во всех неймспейсах (включая удаленные).
79. **`helm history <release> -n <namespace>`** — Показать историю изменений Helm-релиза.
80. **`helm rollback <release> <revision> -n <namespace>`** — Откатить Helm-релиз на предыдущую версию.
81. **`helm get values <release> -n <namespace>`** — Посмотреть, с какими values был установлен релиз.
82. **`helm template . -f values.yaml --debug`** — Посмотреть, какие манифесты сгенерирует Helm (без установки в кластер).
### Полезные алиасы для продакшена
Многие админы добавляют в `~/.bashrc` или `~/.zshrc` такие алиасы:
bash
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgsvc='kubectl get services'
alias kga='kubectl get all'
alias kd='kubectl describe'
alias kl='kubectl logs -f'
alias kx='kubectl exec -it'
alias kctx='kubectl config use-context'
alias kns='kubectl config set-context --current --namespace'
alias ktail='kubectl logs -f --tail=100'
alias kprom='kubectl get pods -o wide | grep -v Running

## Пример реального сценария: Инцидент

Представьте, что в проде упал сервис. Алгоритм действий:
1. **`kubectl get pods -n production | grep <service>`** — Найти поды сервиса.
2. **`kubectl describe pod <pod-name> -n production | grep -A 10 Events:`** — Посмотреть события, почему под упал.
3. **`kubectl logs --previous <pod-name> -n production`** — Посмотреть логи предыдущего пода (если текущий перезапустился).
4. **`kubectl rollout undo deployment/<service> -n production`** — Срочно откатить деплой, если проблема в новом образе.
5. **`kubectl scale deployment/<service> -n production --replicas=5`** — Временно увеличить количество реплик, чтобы справиться с нагрузкой.
6. **`kubectl get events -n production --sort-by='.lastTimestamp'`** — Проверить, нет ли других проблем в неймспейсе.
7. **`kubectl top pods -n production | grep <service>`** — Проверить потребление ресурсов проблемным сервисом.
8. **`kubectl exec -it debug-pod -- curl http://<service>:<port>/health`** — Проверить healthcheck сервиса изнутри кластера.
9. **`kubectl get endpoints <service> -n production`** — Убедиться, что endpoints соответствуют живым подам.
10. **`kubectl describe node <node-where-pod-was> | grep -A 5 Conditions`** — Проверить состояние ноды, если под падал из-за проблем с ней.