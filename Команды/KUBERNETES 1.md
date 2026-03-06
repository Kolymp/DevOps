```
# Контекст и конфигурация
kubectl config view                   # Посмотреть конфиг
kubectl config get-contexts           # Список контекстов
kubectl config use-context <name>     # Переключить контекст
kubectl config current-context        # Текущий контекст
kubectl cluster-info                  # Информация о кластере

# Namespace
kubectl get namespaces                # Список namespace'ов
kubectl create namespace dev          # Создать namespace
kubectl config set-context --current --namespace=dev  # Переключить namespace
kubectl delete namespace dev          # Удалить namespace

# Pods
kubectl get pods                      # Поды в текущем namespace
kubectl get pods -A                   # Поды во всех namespace
kubectl get pods -o wide              # Расширенная информация
kubectl get pods -w                   # Watch (следить за изменениями)
kubectl describe pod <pod-name>       # Детальная информация
kubectl logs <pod-name>               # Логи пода
kubectl logs -f <pod-name>            # Следить за логами
kubectl logs <pod-name> -c <container> # Логи конкретного контейнера
kubectl exec -it <pod-name> -- bash   # Зайти в под
kubectl delete pod <pod-name>         # Удалить под
kubectl port-forward <pod> 8080:80    # Проброс порта

# Deployments
kubectl get deployments               # Список deployments
kubectl create deployment nginx --image=nginx:latest
kubectl describe deployment <name>    # Информация
kubectl scale deployment <name> --replicas=3
kubectl set image deployment/<name> nginx=nginx:1.21
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>  # Откатить
kubectl delete deployment <name>

# Services
kubectl get services                  # Список сервисов
kubectl get svc                       # Короткая форма
kubectl expose deployment nginx --port=80 --type=LoadBalancer
kubectl describe service <name>
kubectl delete service <name>

# ConfigMaps и Secrets
kubectl get configmaps                # Список ConfigMap
kubectl create configmap app-config --from-file=config.yaml
kubectl get secrets                   # Список Secrets
kubectl create secret generic db-password --from-literal=password=mypass
kubectl describe configmap <name>
kubectl get secret <name> -o yaml     # Просмотр в YAML

# Apply и Delete (декларативно)
kubectl apply -f deployment.yaml      # Создать/обновить из файла
kubectl apply -f ./configs/           # Применить все файлы в директории
kubectl delete -f deployment.yaml     # Удалить ресурсы из файла
kubectl diff -f deployment.yaml       # Показать изменения

# Информация о ресурсах
kubectl get all                       # Все ресурсы
kubectl get all -n kube-system        # В конкретном namespace
kubectl get nodes                     # Ноды кластера
kubectl describe node <node-name>     # Информация о ноде
kubectl top nodes                     # Использование ресурсов нодами
kubectl top pods                      # Использование ресурсов подами

# Labels и Selectors
kubectl get pods -l app=nginx         # Фильтр по label
kubectl label pod <pod> env=prod      # Добавить label
kubectl get pods --show-labels        # Показать labels

# События и отладка
kubectl get events                    # События в namespace
kubectl get events --sort-by='.lastTimestamp'
kubectl describe pod <pod> | grep -A 10 Events

# Копирование файлов
kubectl cp <pod>:/path/file ./file    # Из пода
kubectl cp ./file <pod>:/path/file    # В под

# Работа с манифестами
kubectl create -f file.yaml           # Создать (ошибка если существует)
kubectl apply -f file.yaml            # Создать или обновить
kubectl replace -f file.yaml          # Заменить (должен существовать)
kubectl delete -f file.yaml           # Удалить
kubectl get -f file.yaml              # Показать ресурсы из файла

# Вывод в разных форматах
kubectl get pods -o yaml              # YAML
kubectl get pods -o json              # JSON
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get pods -o wide              # Расширенная таблица

# Shortcuts (короткие формы)
kubectl get po                        # pods
kubectl get svc                       # services
kubectl get deploy                    # deployments
kubectl get rs                        # replicasets
kubectl get cm                        # configmaps
kubectl get ns                        # namespaces
kubectl get no                        # nodes
```