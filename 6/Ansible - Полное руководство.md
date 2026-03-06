## 1. Требования для установки Ansible

text

```
┌────────────────────────────────────────────────────────────┐
│         ТРЕБОВАНИЯ К CONTROL NODE (Control Host)           │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Control Node — машина, с которой ЗАПУСКАЕТСЯ Ansible     │
│                                                            │
│  ОПЕРАЦИОННАЯ СИСТЕМА:                                    │
│  ✅ Linux (любой дистрибутив)                             │
│  ✅ macOS                                                  │
│  ✅ WSL (Windows Subsystem for Linux)                     │
│  ❌ Windows (нельзя использовать как control node!)       │
│                                                            │
│  СОФТ:                                                    │
│  • Python 3.8+ (обязательно)                              │
│  • SSH client                                             │
│  • pip (для установки Ansible)                           │
│                                                            │
│  УСТАНОВКА:                                               │
│  # Ubuntu/Debian                                          │
│  sudo apt update                                          │
│  sudo apt install python3 python3-pip                     │
│  pip3 install ansible                                     │
│                                                            │
│  # CentOS/RHEL                                            │
│  sudo yum install python3 python3-pip                     │
│  pip3 install ansible                                     │
│                                                            │
│  # macOS                                                  │
│  brew install ansible                                     │
│                                                            │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│         ТРЕБОВАНИЯ К MANAGED NODES (Целевые хосты)         │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Managed Node — машина, КОТОРОЙ управляет Ansible         │
│                                                            │
│  ОПЕРАЦИОННАЯ СИСТЕМА:                                    │
│  ✅ Linux (любой)                                         │
│  ✅ Unix (BSD, Solaris)                                   │
│  ✅ macOS                                                  │
│  ✅ Windows (через WinRM, не SSH)                         │
│                                                            │
│  СОФТ:                                                    │
│  • Python 2.7+ или 3.5+ (НЕ обязательно Ansible!)        │
│  • SSH server (openssh-server)                           │
│  • sudo (если нужна эскалация привилегий)                │
│                                                            │
│  НАСТРОЙКА SSH:                                           │
│  # На managed node:                                       │
│  sudo apt install openssh-server                          │
│  sudo systemctl enable ssh                                │
│  sudo systemctl start ssh                                 │
│                                                            │
│  # Разрешить SSH ключ от control node:                   │
│  ssh-copy-id user@managed-node                            │
│                                                            │
│  ВАЖНО:                                                   │
│  • Ansible НЕ требует установки на managed nodes          │
│  • Работает через SSH (agentless)                         │
│  • Python нужен для выполнения модулей                    │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## 2. Какими хостами можно управлять

text

```
┌────────────────────────────────────────────────────────────┐
│         ТИПЫ MANAGED NODES                                 │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  LINUX/UNIX СЕРВЕРЫ (через SSH):                          │
│  ✅ Ubuntu, Debian, CentOS, RHEL, Fedora                  │
│  ✅ Alpine Linux                                          │
│  ✅ FreeBSD, OpenBSD                                      │
│  ✅ Solaris                                               │
│  ✅ macOS                                                  │
│                                                            │
│  WINDOWS СЕРВЕРЫ (через WinRM):                           │
│  ✅ Windows Server 2012+                                  │
│  ✅ Windows 10+                                           │
│  • Требуется настройка WinRM                              │
│  • Ansible модули для Windows отличаются                  │
│                                                            │
│  СЕТЕВОЕ ОБОРУДОВАНИЕ (через специальные модули):         │
│  ✅ Cisco IOS, IOS-XR, NX-OS                              │
│  ✅ Juniper Junos                                         │
│  ✅ Arista EOS                                            │
│  ✅ F5 BIG-IP                                             │
│                                                            │
│  ОБЛАЧНЫЕ ПЛАТФОРМЫ (через API):                          │
│  ✅ AWS EC2                                               │
│  ✅ Azure VMs                                             │
│  ✅ Google Cloud Compute                                  │
│  ✅ OpenStack                                             │
│  ✅ VMware vSphere                                        │
│                                                            │
│  КОНТЕЙНЕРЫ:                                              │
│  ✅ Docker containers                                     │
│  ✅ LXC/LXD                                               │
│                                                            │
│  БАЗЫ ДАННЫХ:                                             │
│  ✅ MySQL, PostgreSQL, MongoDB                            │
│  ✅ Redis, Elasticsearch                                  │
│                                                            │
│  KUBERNETES:                                              │
│  ✅ Управление ресурсами K8s (через модули k8s)           │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## 3. Файл ansible.cfg

text

```
┌────────────────────────────────────────────────────────────┐
│         ФАЙЛ ansible.cfg                                   │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ansible.cfg — конфигурационный файл Ansible               │
│  Определяет настройки по умолчанию                        │
│                                                            │
│  ПРИОРИТЕТ ПОИСКА (от высшего к низшему):                 │
│  1. ANSIBLE_CONFIG (переменная окружения)                 │
│  2. ./ansible.cfg (в текущей директории)                  │
│  3. ~/.ansible.cfg (в домашней директории)                │
│  4. /etc/ansible/ansible.cfg (глобальный)                 │
│                                                            │
│  ПРОВЕРИТЬ ИСПОЛЬЗУЕМУЮ КОНФИГУРАЦИЮ:                     │
│  ansible --version                                        │
│    config file = /path/to/ansible.cfg                     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Пример ansible.cfg:**

ini

```
# ═══════════════════════════════════════════════════════════
#          ansible.cfg
# ═══════════════════════════════════════════════════════════

[defaults]
# ───────────────────────────────────────────────────────────
# Путь к inventory файлу
# ───────────────────────────────────────────────────────────
inventory = ./inventory/hosts.ini

# ───────────────────────────────────────────────────────────
# Удалённый пользователь (по умолчанию)
# ───────────────────────────────────────────────────────────
remote_user = ansible

# ───────────────────────────────────────────────────────────
# Путь к SSH ключу
# ───────────────────────────────────────────────────────────
private_key_file = ~/.ssh/ansible_key

# ───────────────────────────────────────────────────────────
# Количество параллельных подключений
# ───────────────────────────────────────────────────────────
forks = 10

# ───────────────────────────────────────────────────────────
# Таймаут SSH подключения (секунды)
# ───────────────────────────────────────────────────────────
timeout = 30

# ───────────────────────────────────────────────────────────
# Проверка SSH ключей хоста
# ───────────────────────────────────────────────────────────
host_key_checking = False  # Отключить (для тестирования)

# ───────────────────────────────────────────────────────────
# Логирование
# ───────────────────────────────────────────────────────────
log_path = ./ansible.log

# ───────────────────────────────────────────────────────────
# Формат вывода
# ───────────────────────────────────────────────────────────
stdout_callback = yaml  # yaml, json, debug, minimal

# ───────────────────────────────────────────────────────────
# Путь к ролям
# ───────────────────────────────────────────────────────────
roles_path = ./roles:~/.ansible/roles:/usr/share/ansible/roles

# ───────────────────────────────────────────────────────────
# Не собирать facts (ускоряет выполнение)
# ───────────────────────────────────────────────────────────
gathering = smart  # smart, explicit, implicit

# ───────────────────────────────────────────────────────────
# Retry файлы (сохранять список failed хостов)
# ───────────────────────────────────────────────────────────
retry_files_enabled = True
retry_files_save_path = ./retry

# ───────────────────────────────────────────────────────────
# Цвета в выводе
# ───────────────────────────────────────────────────────────
nocolor = False

# ───────────────────────────────────────────────────────────
# Interpreter Python на managed nodes
# ───────────────────────────────────────────────────────────
interpreter_python = auto  # auto, /usr/bin/python3

[privilege_escalation]
# ───────────────────────────────────────────────────────────
# Эскалация привилегий (sudo)
# ───────────────────────────────────────────────────────────
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
# ───────────────────────────────────────────────────────────
# SSH настройки
# ───────────────────────────────────────────────────────────
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
pipelining = True  # Ускорение (требует sudo без tty)
control_path = /tmp/ansible-ssh-%%h-%%p-%%r
```

---

## 4. Inventory файл

text

```
┌────────────────────────────────────────────────────────────┐
│         INVENTORY FILE                                     │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Inventory — список managed nodes (хостов)                │
│                                                            │
│  ФОРМАТЫ:                                                 │
│  • INI (простой, классический)                            │
│  • YAML (современный, рекомендуется)                      │
│  • JSON (программный)                                     │
│  • Dynamic inventory (скрипт/плагин)                      │
│                                                            │
│  ГДЕ ХРАНИТЬ:                                             │
│  • Любое имя: inventory, hosts, hosts.ini, hosts.yml      │
│  • В ansible.cfg: inventory = ./inventory/hosts.ini       │
│  • Или параметр: ansible-playbook -i hosts.ini play.yml   │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### INI формат

ini

```
# ═══════════════════════════════════════════════════════════
#          INVENTORY — INI формат (hosts.ini)
# ═══════════════════════════════════════════════════════════

# ───────────────────────────────────────────────────────────
# Ungrouped хосты (без группы)
# ───────────────────────────────────────────────────────────
standalone-server.example.com

# ───────────────────────────────────────────────────────────
# Группа: webservers
# ───────────────────────────────────────────────────────────
[webservers]
web1.example.com
web2.example.com
web3.example.com

# С параметрами
web4.example.com ansible_host=192.168.1.10 ansible_port=2222

# Alias (удобное имя)
web-prod ansible_host=web.example.com ansible_user=deploy

# ───────────────────────────────────────────────────────────
# Группа: databases
# ───────────────────────────────────────────────────────────
[databases]
db1.example.com
db2.example.com

# С переменными
db-master.example.com ansible_host=10.0.0.5 db_role=master
db-slave.example.com ansible_host=10.0.0.6 db_role=slave

# ───────────────────────────────────────────────────────────
# Паттерны (диапазоны)
# ───────────────────────────────────────────────────────────
[workers]
worker[01:10].example.com  # worker01, worker02, ..., worker10
worker[a:f].example.com    # workera, workerb, ..., workerf

# ───────────────────────────────────────────────────────────
# Переменные для группы
# ───────────────────────────────────────────────────────────
[webservers:vars]
ansible_user=www-data
ansible_python_interpreter=/usr/bin/python3
http_port=80
max_clients=200

[databases:vars]
ansible_user=postgres
db_port=5432

# ───────────────────────────────────────────────────────────
# Вложенные группы (группа групп)
# ───────────────────────────────────────────────────────────
[production:children]
webservers
databases

[staging:children]
webservers-staging
databases-staging

# Переменные для вложенной группы
[production:vars]
environment=production
monitoring=enabled

# ───────────────────────────────────────────────────────────
# Localhost (без SSH)
# ───────────────────────────────────────────────────────────
[local]
localhost ansible_connection=local

# ───────────────────────────────────────────────────────────
# Общие переменные для ВСЕХ хостов
# ───────────────────────────────────────────────────────────
[all:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

### YAML формат

YAML

```
# ═══════════════════════════════════════════════════════════
#          INVENTORY — YAML формат (hosts.yml)
# ═══════════════════════════════════════════════════════════

all:
  # ─────────────────────────────────────────────────────────
  # Переменные для всех хостов
  # ─────────────────────────────────────────────────────────
  vars:
    ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
    ansible_python_interpreter: /usr/bin/python3
  
  # ─────────────────────────────────────────────────────────
  # Ungrouped хосты
  # ─────────────────────────────────────────────────────────
  hosts:
    standalone-server.example.com:
  
  # ─────────────────────────────────────────────────────────
  # Группы хостов
  # ─────────────────────────────────────────────────────────
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
        web3.example.com:
        
        # С параметрами
        web4.example.com:
          ansible_host: 192.168.1.10
          ansible_port: 2222
          ansible_user: deploy
        
        # Alias
        web-prod:
          ansible_host: web.example.com
          ansible_user: deploy
          ansible_ssh_private_key_file: ~/.ssh/deploy_key
      
      # Переменные группы
      vars:
        http_port: 80
        max_clients: 200
    
    databases:
      hosts:
        db1.example.com:
        db2.example.com:
        
        db-master.example.com:
          ansible_host: 10.0.0.5
          db_role: master
        
        db-slave.example.com:
          ansible_host: 10.0.0.6
          db_role: slave
      
      vars:
        ansible_user: postgres
        db_port: 5432
    
    # ───────────────────────────────────────────────────────
    # Вложенные группы
    # ───────────────────────────────────────────────────────
    production:
      children:
        webservers:
        databases:
      
      vars:
        environment: production
        monitoring: enabled
    
    staging:
      children:
        webservers-staging:
        databases-staging:
      
      vars:
        environment: staging
    
    # ───────────────────────────────────────────────────────
    # Localhost
    # ───────────────────────────────────────────────────────
    local:
      hosts:
        localhost:
          ansible_connection: local
```

### Dynamic Inventory

Python

```
#!/usr/bin/env python3
# ═══════════════════════════════════════════════════════════
#          DYNAMIC INVENTORY — inventory.py
# ═══════════════════════════════════════════════════════════

import json
import sys

def get_inventory():
    """Получить inventory из внешнего источника (API, БД, файл)"""
    
    # Пример: получить список EC2 инстансов из AWS
    # В реальности используется boto3
    
    inventory = {
        'webservers': {
            'hosts': ['web1.example.com', 'web2.example.com'],
            'vars': {
                'http_port': 80,
                'ansible_user': 'ubuntu'
            }
        },
        'databases': {
            'hosts': ['db1.example.com'],
            'vars': {
                'db_port': 5432
            }
        },
        '_meta': {
            'hostvars': {
                'web1.example.com': {
                    'ansible_host': '192.168.1.10'
                },
                'web2.example.com': {
                    'ansible_host': '192.168.1.11'
                }
            }
        }
    }
    
    return inventory

if __name__ == '__main__':
    if len(sys.argv) == 2 and sys.argv[1] == '--list':
        print(json.dumps(get_inventory(), indent=2))
    elif len(sys.argv) == 3 and sys.argv[1] == '--host':
        # Вернуть переменные конкретного хоста
        print(json.dumps({}))
    else:
        print("Usage: inventory.py --list")
        sys.exit(1)
```

Bash

```
# Сделать исполняемым
chmod +x inventory.py

# Использовать
ansible-playbook -i inventory.py playbook.yml

# Или в ansible.cfg:
# inventory = ./inventory.py
```

---

## 5. Модули vs Плагины

text

```
┌────────────────────────────────────────────────────────────┐
│         МОДУЛИ vs ПЛАГИНЫ                                  │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  МОДУЛИ (Modules):                                        │
│  ──────────────────                                        │
│  • Выполняются НА MANAGED NODES                           │
│  • Делают конкретные действия (install, copy, user)       │
│  • Используются в tasks                                   │
│  • Примеры: apt, yum, copy, file, user, service           │
│                                                            │
│  ПЛАГИНЫ (Plugins):                                       │
│  ──────────────────                                        │
│  • Выполняются НА CONTROL NODE                            │
│  • Расширяют функциональность Ansible                     │
│  • НЕ используются в tasks напрямую                       │
│  • Типы:                                                  │
│    - Connection plugins (ssh, winrm, docker)              │
│    - Lookup plugins (file, env, password)                 │
│    - Filter plugins (to_json, regex_replace)              │
│    - Callback plugins (yaml, json вывод)                  │
│    - Inventory plugins (aws_ec2, azure_rm)                │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Сравнение:**

text

```
┌──────────────────┬────────────────────┬─────────────────────┐
│   Критерий       │     МОДУЛЬ         │      ПЛАГИН         │
├──────────────────┼────────────────────┼─────────────────────┤
│ Где выполняется  │ Managed node       │ Control node        │
│ Использование    │ В tasks            │ Автоматически       │
│ Примеры          │ apt, copy, user    │ ssh, lookup, json   │
│ Количество       │ 3000+              │ 100+                │
└──────────────────┴────────────────────┴─────────────────────┘
```

---

## 6. Playbook, Play, Task

text

```
┌────────────────────────────────────────────────────────────┐
│         PLAYBOOK → PLAY → TASK                             │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  PLAYBOOK (плейбук):                                      │
│  • YAML файл с описанием автоматизации                    │
│  • Состоит из одного или нескольких PLAY                  │
│  • Файл: playbook.yml, site.yml, deploy.yml              │
│                                                            │
│  PLAY (плей):                                             │
│  • Набор задач (tasks) для группы хостов                  │
│  • Playbook может содержать несколько play                │
│  • Каждый play начинается с "- name:" и "hosts:"          │
│                                                            │
│  TASK (задача):                                           │
│  • Вызов МОДУЛЯ с параметрами                             │
│  • Минимальная единица работы                             │
│  • Play состоит из нескольких tasks                       │
│                                                            │
│  СТРУКТУРА:                                               │
│  ┌────────────────────────────────────────────────────┐   │
│  │ PLAYBOOK: deploy.yml                               │   │
│  │ ├─ PLAY 1: Setup webservers                        │   │
│  │ │  ├─ TASK 1: Install nginx                        │   │
│  │ │  ├─ TASK 2: Copy config                          │   │
│  │ │  └─ TASK 3: Start service                        │   │
│  │ └─ PLAY 2: Setup databases                         │   │
│  │    ├─ TASK 1: Install postgresql                   │   │
│  │    └─ TASK 2: Create database                      │   │
│  └────────────────────────────────────────────────────┘   │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Пример:**

YAML

```
# ═══════════════════════════════════════════════════════════
#          PLAYBOOK с несколькими PLAY
# ═══════════════════════════════════════════════════════════

---
# ───────────────────────────────────────────────────────────
# PLAY 1: Настройка веб-серверов
# ───────────────────────────────────────────────────────────
- name: Setup webservers  # Это PLAY
  hosts: webservers
  become: yes
  
  tasks:
    # TASK 1
    - name: Install Nginx
      apt:
        name: nginx
        state: present
    
    # TASK 2
    - name: Copy Nginx config
      copy:
        src: files/nginx.conf
        dest: /etc/nginx/nginx.conf
    
    # TASK 3
    - name: Start Nginx
      service:
        name: nginx
        state: started

# ───────────────────────────────────────────────────────────
# PLAY 2: Настройка баз данных
# ───────────────────────────────────────────────────────────
- name: Setup databases  # Это второй PLAY
  hosts: databases
  become: yes
  
  tasks:
    # TASK 1
    - name: Install PostgreSQL
      apt:
        name: postgresql
        state: present
    
    # TASK 2
    - name: Create database
      postgresql_db:
        name: myapp
        state: present
```

---

## 7. Ansible Role

text

```
┌────────────────────────────────────────────────────────────┐
│         ANSIBLE ROLE                                       │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Role — переиспользуемый набор конфигурации               │
│                                                            │
│  ЗАЧЕМ НУЖНЫ РОЛИ:                                        │
│  • Организация кода (не всё в одном playbook)             │
│  • Переиспользование (одна роль в разных проектах)        │
│  • Модульность (nginx роль, postgres роль)                │
│  • Ansible Galaxy (готовые роли от сообщества)            │
│                                                            │
│  СТРУКТУРА РОЛИ:                                          │
│  roles/nginx/                                             │
│  ├── tasks/          # Задачи                             │
│  │   └── main.yml                                         │
│  ├── handlers/       # Обработчики событий                │
│  │   └── main.yml                                         │
│  ├── templates/      # Jinja2 шаблоны                     │
│  │   └── nginx.conf.j2                                    │
│  ├── files/          # Статические файлы                  │
│  │   └── index.html                                       │
│  ├── vars/           # Переменные                         │
│  │   └── main.yml                                         │
│  ├── defaults/       # Переменные по умолчанию            │
│  │   └── main.yml                                         │
│  ├── meta/           # Метаданные роли                    │
│  │   └── main.yml                                         │
│  └── README.md       # Документация                       │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Создание роли:**

Bash

```
# Создать структуру роли
ansible-galaxy init nginx

# Или вручную
mkdir -p roles/nginx/{tasks,handlers,templates,files,vars,defaults,meta}
```

**Пример роли:**

YAML

```
# roles/nginx/tasks/main.yml
# ═══════════════════════════════════════════════════════════

---
- name: Install Nginx
  apt:
    name: nginx
    state: present

- name: Copy Nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart Nginx

- name: Start Nginx
  service:
    name: nginx
    state: started
    enabled: yes
```

YAML

```
# roles/nginx/handlers/main.yml
# ═══════════════════════════════════════════════════════════

---
- name: Restart Nginx
  service:
    name: nginx
    state: restarted
```

YAML

```
# roles/nginx/defaults/main.yml
# ═══════════════════════════════════════════════════════════

---
nginx_port: 80
nginx_user: www-data
```

**Использование роли:**

YAML

```
# ═══════════════════════════════════════════════════════════
#          PLAYBOOK с ролями
# ═══════════════════════════════════════════════════════════

---
- name: Setup webservers
  hosts: webservers
  become: yes
  
  roles:
    - nginx
    - { role: php, php_version: '8.1' }
    - role: mysql
      vars:
        mysql_root_password: secret
```

---

## 8. Задать пользователя для подключения

YAML

```
# ═══════════════════════════════════════════════════════════
#          СПОСОБЫ ЗАДАТЬ ПОЛЬЗОВАТЕЛЯ
# ═══════════════════════════════════════════════════════════

# ───────────────────────────────────────────────────────────
# 1. В playbook (для всех хостов в play)
# ───────────────────────────────────────────────────────────
- name: Deploy application
  hosts: webservers
  remote_user: deploy  # ← ИМЯ ПОЛЬЗОВАТЕЛЯ
  
  tasks:
    - name: Task
      ...

# ───────────────────────────────────────────────────────────
# 2. Для конкретного task
# ───────────────────────────────────────────────────────────
- name: Deploy application
  hosts: webservers
  
  tasks:
    - name: Copy as deploy user
      copy:
        src: app.tar.gz
        dest: /tmp/
      remote_user: deploy  # ← Только для этого task

# ───────────────────────────────────────────────────────────
# 3. В inventory (для конкретного хоста)
# ───────────────────────────────────────────────────────────
# hosts.ini:
[webservers]
web1.example.com ansible_user=deploy

# hosts.yml:
webservers:
  hosts:
    web1.example.com:
      ansible_user: deploy

# ───────────────────────────────────────────────────────────
# 4. В inventory (для группы хостов)
# ───────────────────────────────────────────────────────────
# hosts.ini:
[webservers:vars]
ansible_user=deploy

# hosts.yml:
webservers:
  vars:
    ansible_user: deploy

# ───────────────────────────────────────────────────────────
# 5. В ansible.cfg (глобально)
# ───────────────────────────────────────────────────────────
# ansible.cfg:
[defaults]
remote_user = deploy

# ───────────────────────────────────────────────────────────
# 6. Через командную строку
# ───────────────────────────────────────────────────────────
ansible-playbook playbook.yml -u deploy
# или
ansible-playbook playbook.yml --user=deploy
```

---

## 9. Задать путь к SSH ключу

YAML

```
# ═══════════════════════════════════════════════════════════
#          СПОСОБЫ ЗАДАТЬ SSH КЛЮЧ
# ═══════════════════════════════════════════════════════════

# ───────────────────────────────────────────────────────────
# 1. В playbook
# ───────────────────────────────────────────────────────────
- name: Deploy
  hosts: webservers
  vars:
    ansible_ssh_private_key_file: ~/.ssh/deploy_key
  
  tasks:
    - name: Task
      ...

# ───────────────────────────────────────────────────────────
# 2. В inventory (для хоста)
# ───────────────────────────────────────────────────────────
# hosts.ini:
web1.example.com ansible_ssh_private_key_file=~/.ssh/deploy_key

# hosts.yml:
webservers:
  hosts:
    web1.example.com:
      ansible_ssh_private_key_file: ~/.ssh/deploy_key

# ───────────────────────────────────────────────────────────
# 3. В inventory (для группы)
# ───────────────────────────────────────────────────────────
# hosts.ini:
[webservers:vars]
ansible_ssh_private_key_file=~/.ssh/deploy_key

# hosts.yml:
webservers:
  vars:
    ansible_ssh_private_key_file: ~/.ssh/deploy_key

# ───────────────────────────────────────────────────────────
# 4. В ansible.cfg (глобально)
# ───────────────────────────────────────────────────────────
[defaults]
private_key_file = ~/.ssh/deploy_key

# ───────────────────────────────────────────────────────────
# 5. Через командную строку
# ───────────────────────────────────────────────────────────
ansible-playbook playbook.yml --private-key=~/.ssh/deploy_key
```

---

## 10. Скопировать файл

YAML

```
# ═══════════════════════════════════════════════════════════
#          КОПИРОВАНИЕ ФАЙЛОВ
# ═══════════════════════════════════════════════════════════

---
- name: Copy files example
  hosts: webservers
  become: yes
  
  tasks:
    # ───────────────────────────────────────────────────────
    # 1. COPY модуль — копировать файл
    # ───────────────────────────────────────────────────────
    - name: Copy file from control node to managed node
      copy:
        src: /local/path/file.txt         # На control node
        dest: /remote/path/file.txt       # На managed node
        owner: www-data
        group: www-data
        mode: '0644'
        backup: yes  # Создать backup если файл существует
    
    # ───────────────────────────────────────────────────────
    # 2. Копировать директорию
    # ───────────────────────────────────────────────────────
    - name: Copy directory
      copy:
        src: /local/app/        # Trailing slash важен!
        dest: /var/www/app/
        owner: www-data
        mode: '0755'
    
    # ───────────────────────────────────────────────────────
    # 3. Копировать с содержимым (inline)
    # ───────────────────────────────────────────────────────
    - name: Create file with content
      copy:
        content: |
          server {
              listen 80;
              server_name example.com;
          }
        dest: /etc/nginx/sites-available/example.com
        mode: '0644'
    
    # ───────────────────────────────────────────────────────
    # 4. FETCH модуль — скачать с managed node на control
    # ───────────────────────────────────────────────────────
    - name: Fetch log file from remote
      fetch:
        src: /var/log/app.log               # На managed node
        dest: /tmp/logs/{{ inventory_hostname }}/  # Control node
        flat: yes  # Без сохранения структуры директорий
    
    # ───────────────────────────────────────────────────────
    # 5. SYNCHRONIZE — rsync (быстрее для больших файлов)
    # ───────────────────────────────────────────────────────
    - name: Sync directory with rsync
      synchronize:
        src: /local/data/
        dest: /remote/data/
        delete: yes  # Удалить файлы, которых нет в src
        rsync_opts:
          - "--exclude=*.log"
          - "--exclude=.git"
```
## 11. Создать пользователя в Linux

YAML

```
# ═══════════════════════════════════════════════════════════
#          СОЗДАНИЕ ПОЛЬЗОВАТЕЛЯ (USER модуль)
# ═══════════════════════════════════════════════════════════

---
- name: User management examples
  hosts: all
  become: yes
  
  tasks:
    # ───────────────────────────────────────────────────────
    # 1. Простое создание пользователя
    # ───────────────────────────────────────────────────────
    - name: Create user 'john'
      user:
        name: john
        state: present  # present = создать, absent = удалить
    
    # ───────────────────────────────────────────────────────
    # 2. Создание с дополнительными параметрами
    # ───────────────────────────────────────────────────────
    - name: Create user with full options
      user:
        name: deploy
        uid: 1500                          # User ID
        group: deploy                      # Primary group
        groups: sudo,www-data              # Additional groups
        shell: /bin/bash                   # Shell
        home: /home/deploy                 # Home directory
        create_home: yes                   # Создать home dir
        comment: "Deployment user"         # GECOS field
        password: "{{ 'P@ssw0rd' | password_hash('sha512') }}"  # Хеш пароля
        update_password: on_create         # Обновлять пароль только при создании
        state: present
    
    # ───────────────────────────────────────────────────────
    # 3. Создание системного пользователя
    # ───────────────────────────────────────────────────────
    - name: Create system user for application
      user:
        name: appuser
        system: yes                        # Системный пользователь (UID < 1000)
        shell: /bin/false                  # Нельзя логиниться
        create_home: no                    # Без home directory
        state: present
    
    # ───────────────────────────────────────────────────────
    # 4. Добавить SSH ключ пользователю
    # ───────────────────────────────────────────────────────
    - name: Create user with SSH key
      user:
        name: developer
        state: present
    
    - name: Add SSH authorized key
      authorized_key:
        user: developer
        key: "{{ lookup('file', '/path/to/public_key.pub') }}"
        state: present
    
    # Или несколько ключей
    - name: Add multiple SSH keys
      authorized_key:
        user: developer
        key: "{{ item }}"
      loop:
        - "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ... user1@host"
        - "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ... user2@host"
    
    # ───────────────────────────────────────────────────────
    # 5. Установить срок действия пароля
    # ───────────────────────────────────────────────────────
    - name: Create user with password expiration
      user:
        name: tempuser
        password: "{{ 'TempPass123' | password_hash('sha512') }}"
        expires: 1735689600  # Unix timestamp (2025-01-01)
        state: present
    
    # ───────────────────────────────────────────────────────
    # 6. Дать sudo права (без пароля)
    # ───────────────────────────────────────────────────────
    - name: Create user with sudo rights
      user:
        name: admin
        groups: sudo
        append: yes  # Добавить к группе, не заменять
        state: present
    
    - name: Allow sudo without password
      lineinfile:
        path: /etc/sudoers.d/admin
        line: 'admin ALL=(ALL) NOPASSWD:ALL'
        create: yes
        mode: '0440'
        validate: 'visudo -cf %s'  # Валидация перед применением
    
    # ───────────────────────────────────────────────────────
    # 7. Удалить пользователя
    # ───────────────────────────────────────────────────────
    - name: Remove user
      user:
        name: olduser
        state: absent
        remove: yes  # Удалить home directory и mail spool
        force: yes   # Убить процессы пользователя перед удалением
    
    # ───────────────────────────────────────────────────────
    # 8. Создать группу перед созданием пользователя
    # ───────────────────────────────────────────────────────
    - name: Create group
      group:
        name: developers
        gid: 2000
        state: present
    
    - name: Create user in group
      user:
        name: dev1
        group: developers
        state: present
    
    # ───────────────────────────────────────────────────────
    # 9. Заблокировать/разблокировать пользователя
    # ───────────────────────────────────────────────────────
    - name: Lock user account
      user:
        name: john
        password_lock: yes  # Заблокировать
    
    - name: Unlock user account
      user:
        name: john
        password_lock: no   # Разблокировать
    
    # ───────────────────────────────────────────────────────
    # 10. Пример с переменными
    # ───────────────────────────────────────────────────────
    - name: Create multiple users from list
      user:
        name: "{{ item.name }}"
        groups: "{{ item.groups }}"
        shell: "{{ item.shell | default('/bin/bash') }}"
        state: present
      loop:
        - { name: 'alice', groups: 'developers', shell: '/bin/bash' }
        - { name: 'bob', groups: 'operators', shell: '/bin/zsh' }
        - { name: 'charlie', groups: 'developers,sudo', shell: '/bin/bash' }
```

---

## 12. Template модуль

text

```
┌────────────────────────────────────────────────────────────┐
│         TEMPLATE МОДУЛЬ                                    │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ЗАЧЕМ НУЖЕН:                                             │
│  • Создание конфигурационных файлов с динамическими       │
│    значениями                                             │
│  • Использование переменных Ansible                       │
│  • Условная логика (if/else)                              │
│  • Циклы (for)                                            │
│  • Фильтры (to_json, upper, lower, etc.)                  │
│                                                            │
│  ОТЛИЧИЕ ОТ COPY:                                         │
│  • COPY — статический файл (как есть)                     │
│  • TEMPLATE — Jinja2 шаблон (с переменными)               │
│                                                            │
│  СИНТАКСИС JINJA2:                                        │
│  {{ variable }}        — вывод переменной                 │
│  {% if condition %}    — условие                          │
│  {% for item in list %}— цикл                             │
│  {{ var | filter }}    — фильтр                           │
│  {# comment #}         — комментарий                      │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Пример использования:**

YAML

```
# ═══════════════════════════════════════════════════════════
#          PLAYBOOK с template
# ═══════════════════════════════════════════════════════════

---
- name: Configure Nginx
  hosts: webservers
  become: yes
  
  vars:
    nginx_port: 80
    server_name: example.com
    max_clients: 100
    enable_ssl: true
    ssl_cert: /etc/ssl/certs/example.com.crt
    ssl_key: /etc/ssl/private/example.com.key
    upstream_servers:
      - 192.168.1.10:8080
      - 192.168.1.11:8080
      - 192.168.1.12:8080
  
  tasks:
    # ───────────────────────────────────────────────────────
    # Использование template
    # ───────────────────────────────────────────────────────
    - name: Configure Nginx from template
      template:
        src: templates/nginx.conf.j2      # Jinja2 шаблон
        dest: /etc/nginx/nginx.conf       # Целевой файл
        owner: root
        group: root
        mode: '0644'
        backup: yes                       # Backup перед изменением
        validate: 'nginx -t -c %s'        # Валидация перед применением
      notify: Restart Nginx
  
  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

**Шаблон Jinja2:**

jinja2

```
{# ═══════════════════════════════════════════════════════════
   templates/nginx.conf.j2
   ═══════════════════════════════════════════════════════════ #}

{# Комментарий — не попадёт в результат #}

user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections {{ max_clients }};  {# Переменная #}
}

http {
    sendfile on;
    tcp_nopush on;
    
    {# ───────────────────────────────────────────────────── #}
    {# Условие — SSL включён или нет #}
    {# ───────────────────────────────────────────────────── #}
    {% if enable_ssl %}
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    {% endif %}
    
    {# ───────────────────────────────────────────────────── #}
    {# Upstream серверы (цикл) #}
    {# ───────────────────────────────────────────────────── #}
    upstream backend {
        {% for server in upstream_servers %}
        server {{ server }};
        {% endfor %}
    }
    
    server {
        listen {{ nginx_port }};
        server_name {{ server_name }};
        
        {# ─────────────────────────────────────────────────── #}
        {# SSL конфигурация (условно) #}
        {# ─────────────────────────────────────────────────── #}
        {% if enable_ssl %}
        listen 443 ssl http2;
        ssl_certificate {{ ssl_cert }};
        ssl_certificate_key {{ ssl_key }};
        {% endif %}
        
        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            
            {# Ansible facts (системные переменные) #}
            add_header X-Served-By {{ ansible_hostname }};
        }
    }
}

{# ───────────────────────────────────────────────────────── #}
{# Автоматический комментарий (опционально) #}
{# ───────────────────────────────────────────────────────── #}
# This file is managed by Ansible
# Host: {{ inventory_hostname }}
# Date: {{ ansible_date_time.iso8601 }}
```

**Результат (после применения template):**

nginx

```
# /etc/nginx/nginx.conf

user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 100;
}

http {
    sendfile on;
    tcp_nopush on;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    
    upstream backend {
        server 192.168.1.10:8080;
        server 192.168.1.11:8080;
        server 192.168.1.12:8080;
    }
    
    server {
        listen 80;
        server_name example.com;
        
        listen 443 ssl http2;
        ssl_certificate /etc/ssl/certs/example.com.crt;
        ssl_certificate_key /etc/ssl/private/example.com.key;
        
        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            
            add_header X-Served-By web1;
        }
    }
}

# This file is managed by Ansible
# Host: web1.example.com
# Date: 2024-01-15T10:30:45Z
```

**Дополнительные примеры шаблонов:**

jinja2

```
{# ═══════════════════════════════════════════════════════════
   ФИЛЬТРЫ Jinja2
   ═══════════════════════════════════════════════════════════ #}

{# Верхний/нижний регистр #}
{{ server_name | upper }}        → EXAMPLE.COM
{{ server_name | lower }}        → example.com

{# Значение по умолчанию #}
{{ custom_var | default('default_value') }}

{# JSON #}
{{ my_dict | to_json }}
{{ my_dict | to_nice_json }}     → с отступами

{# YAML #}
{{ my_dict | to_yaml }}

{# Математика #}
{{ (max_clients * 2) }}

{# Замена строк #}
{{ server_name | replace('.', '-') }}  → example-com

{# Регулярные выражения #}
{{ server_name | regex_replace('^www\.', '') }}

{# ═══════════════════════════════════════════════════════════
   УСЛОВИЯ
   ═══════════════════════════════════════════════════════════ #}

{% if ansible_distribution == "Ubuntu" %}
apt-get install nginx
{% elif ansible_distribution == "CentOS" %}
yum install nginx
{% else %}
echo "Unknown distribution"
{% endif %}

{# ═══════════════════════════════════════════════════════════
   ЦИКЛЫ
   ═══════════════════════════════════════════════════════════ #}

{% for user in users %}
User {{ loop.index }}: {{ user.name }} ({{ user.email }})
{% endfor %}

{# loop.index — номер итерации (с 1) #}
{# loop.index0 — номер итерации (с 0) #}
{# loop.first — первая итерация #}
{# loop.last — последняя итерация #}

{# ═══════════════════════════════════════════════════════════
   ANSIBLE FACTS (встроенные переменные)
   ═══════════════════════════════════════════════════════════ #}

Hostname: {{ ansible_hostname }}
FQDN: {{ ansible_fqdn }}
OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
Architecture: {{ ansible_architecture }}
CPU cores: {{ ansible_processor_vcpus }}
RAM: {{ ansible_memtotal_mb }} MB
IP: {{ ansible_default_ipv4.address }}
```

---

## 13. Выполнение от привилегированного пользователя

YAML

```
# ═══════════════════════════════════════════════════════════
#          PRIVILEGE ESCALATION (sudo)
# ═══════════════════════════════════════════════════════════

---
# ───────────────────────────────────────────────────────────
# 1. Для всего play (все tasks с sudo)
# ───────────────────────────────────────────────────────────
- name: Install packages
  hosts: webservers
  become: yes              # Включить sudo для всех tasks
  become_method: sudo      # Метод эскалации (по умолчанию)
  become_user: root        # От имени какого пользователя (по умолчанию root)
  
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
      # Выполнится с sudo

# ───────────────────────────────────────────────────────────
# 2. Для конкретного task
# ───────────────────────────────────────────────────────────
- name: Mixed privileges
  hosts: webservers
  
  tasks:
    - name: Check home directory (без sudo)
      command: ls ~
    
    - name: Install package (с sudo)
      apt:
        name: htop
        state: present
      become: yes           # sudo только для этого task
    
    - name: Create file as specific user
      file:
        path: /tmp/file.txt
        state: touch
      become: yes
      become_user: www-data # От имени www-data (не root!)

# ───────────────────────────────────────────────────────────
# 3. В ansible.cfg (глобально)
# ───────────────────────────────────────────────────────────
# ansible.cfg:
[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False  # Не запрашивать пароль sudo

# ───────────────────────────────────────────────────────────
# 4. В inventory
# ───────────────────────────────────────────────────────────
# hosts.ini:
[webservers]
web1.example.com ansible_become=yes ansible_become_method=sudo

# hosts.yml:
webservers:
  hosts:
    web1.example.com:
      ansible_become: yes
      ansible_become_user: root

# ───────────────────────────────────────────────────────────
# 5. Через командную строку
# ───────────────────────────────────────────────────────────
ansible-playbook playbook.yml --become
ansible-playbook playbook.yml -b                    # Короткая форма
ansible-playbook playbook.yml --become-user=root
ansible-playbook playbook.yml --ask-become-pass     # Запросить sudo пароль
ansible-playbook playbook.yml -K                    # Короткая форма

# ───────────────────────────────────────────────────────────
# 6. Блок tasks с become
# ───────────────────────────────────────────────────────────
- name: Example with block
  hosts: webservers
  
  tasks:
    - name: Non-privileged task
      debug:
        msg: "No sudo needed"
    
    - name: Privileged block
      block:
        - name: Install package
          apt:
            name: nginx
            state: present
        
        - name: Start service
          service:
            name: nginx
            state: started
      become: yes  # sudo для всех tasks в блоке
```

---

## 14. Способы эскалации привилегий (become methods)

YAML

```
# ═══════════════════════════════════════════════════════════
#          BECOME METHODS
# ═══════════════════════════════════════════════════════════

---
- name: Privilege escalation methods
  hosts: all
  
  tasks:
    # ───────────────────────────────────────────────────────
    # 1. SUDO (по умолчанию, самый популярный)
    # ───────────────────────────────────────────────────────
    - name: Using sudo
      apt:
        name: nginx
        state: present
      become: yes
      become_method: sudo
      become_user: root
    
    # ───────────────────────────────────────────────────────
    # 2. SU (переключение пользователя)
    # ───────────────────────────────────────────────────────
    - name: Using su
      command: whoami
      become: yes
      become_method: su
      become_user: root
      # Требует пароля root (become_pass или --ask-become-pass)
    
    # ───────────────────────────────────────────────────────
    # 3. PBRUN (PowerBroker)
    # ───────────────────────────────────────────────────────
    - name: Using pbrun
      command: ls /root
      become: yes
      become_method: pbrun
      become_user: root
    
    # ───────────────────────────────────────────────────────
    # 4. PFEXEC (Solaris)
    # ───────────────────────────────────────────────────────
    - name: Using pfexec
      command: ls /etc
      become: yes
      become_method: pfexec
      become_user: root
    
    # ───────────────────────────────────────────────────────
    # 5. DOAS (OpenBSD альтернатива sudo)
    # ───────────────────────────────────────────────────────
    - name: Using doas
      command: whoami
      become: yes
      become_method: doas
      become_user: root
    
    # ───────────────────────────────────────────────────────
    # 6. DZDO (CA Privileged Access Manager)
    # ───────────────────────────────────────────────────────
    - name: Using dzdo
      command: ls /root
      become: yes
      become_method: dzdo
      become_user: root
    
    # ───────────────────────────────────────────────────────
    # 7. KSU (Kerberos su)
    # ───────────────────────────────────────────────────────
    - name: Using ksu
      command: whoami
      become: yes
      become_method: ksu
      become_user: root
    
    # ───────────────────────────────────────────────────────
    # 8. RUNAS (специфично для некоторых систем)
    # ───────────────────────────────────────────────────────
    - name: Using runas
      command: ls /root
      become: yes
      become_method: runas
      become_user: root
```

**Сравнение методов:**

text

```
┌──────────────┬───────────────────┬─────────────────────────┐
│   Метод      │   Система         │   Описание              │
├──────────────┼───────────────────┼─────────────────────────┤
│ sudo         │ Linux (все)       │ Стандарт, рекомендуется │
│ su           │ Unix/Linux        │ Требует root пароль     │
│ pbrun        │ PowerBroker       │ Enterprise             │
│ pfexec       │ Solaris           │ Solaris-specific       │
│ doas         │ OpenBSD           │ Замена sudo            │
│ dzdo         │ CA PAM            │ Enterprise             │
│ ksu          │ Kerberos          │ Kerberos auth          │
│ runas        │ Varies            │ Редко используется     │
└──────────────┴───────────────────┴─────────────────────────┘
```

---

## 15. Handlers

text

```
┌────────────────────────────────────────────────────────────┐
│         HANDLERS                                           │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Handler — специальный task, который выполняется:         │
│  • ТОЛЬКО если был вызван (notify)                        │
│  • ТОЛЬКО ОДИН РАЗ (даже если вызван многократно)         │
│  • В КОНЦЕ play (после всех tasks)                        │
│                                                            │
│  ЗАЧЕМ НУЖНЫ:                                             │
│  • Перезапуск сервисов только если конфиг изменился       │
│  • Избежать лишних перезапусков                           │
│  • Группировка действий (один handler для нескольких tasks)│
│                                                            │
│  КОГДА HANDLER НЕ ВЫПОЛНИТСЯ:                             │
│  • Task не сделал изменений (changed: false)              │
│  • Task упал с ошибкой                                    │
│  • Play был прерван до конца                              │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Примеры:**

YAML

```
# ═══════════════════════════════════════════════════════════
#          HANDLERS — ПРИМЕРЫ
# ═══════════════════════════════════════════════════════════

---
- name: Configure Nginx
  hosts: webservers
  become: yes
  
  tasks:
    # ───────────────────────────────────────────────────────
    # Task 1: Изменить конфиг
    # ───────────────────────────────────────────────────────
    - name: Copy Nginx config
      copy:
        src: files/nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx  # ← Вызвать handler
      # Если файл изменился → handler будет вызван
      # Если файл не изменился → handler НЕ вызывается
    
    # ───────────────────────────────────────────────────────
    # Task 2: Изменить site конфиг
    # ───────────────────────────────────────────────────────
    - name: Copy site config
      template:
        src: templates/site.conf.j2
        dest: /etc/nginx/sites-available/mysite
      notify: Restart Nginx  # ← Тот же handler
      # Если этот task тоже изменил файл,
      # handler всё равно выполнится ОДИН РАЗ
    
    # ───────────────────────────────────────────────────────
    # Task 3: Вызов нескольких handlers
    # ───────────────────────────────────────────────────────
    - name: Update application
      copy:
        src: app.tar.gz
        dest: /var/www/app.tar.gz
      notify:
        - Extract application
        - Restart App
        - Notify Slack
      # Все три handlers будут вызваны
  
  # ─────────────────────────────────────────────────────────
  # HANDLERS (выполняются в конце)
  # ─────────────────────────────────────────────────────────
  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
    
    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded  # Reload вместо restart (быстрее)
    
    - name: Extract application
      unarchive:
        src: /var/www/app.tar.gz
        dest: /var/www/
        remote_src: yes
    
    - name: Restart App
      service:
        name: myapp
        state: restarted
    
    - name: Notify Slack
      slack:
        token: "{{ slack_token }}"
        msg: "Application updated on {{ inventory_hostname }}"
```

**Принудительный вызов handler:**

YAML

```
# ═══════════════════════════════════════════════════════════
#          FLUSH HANDLERS — принудительный вызов
# ═══════════════════════════════════════════════════════════

---
- name: Example with flush
  hosts: webservers
  become: yes
  
  tasks:
    - name: Update config
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx
    
    # ───────────────────────────────────────────────────────
    # meta: flush_handlers — выполнить handlers СЕЙЧАС
    # ───────────────────────────────────────────────────────
    - name: Flush handlers
      meta: flush_handlers
      # Nginx перезапустится ЗДЕСЬ, а не в конце play
    
    - name: Check Nginx is running
      uri:
        url: http://localhost
        status_code: 200
      # Этот task выполнится ПОСЛЕ перезапуска Nginx
  
  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

**Listen — группировка handlers:**

YAML

```
# ═══════════════════════════════════════════════════════════
#          LISTEN — альтернативный синтаксис
# ═══════════════════════════════════════════════════════════

---
- name: Example with listen
  hosts: webservers
  become: yes
  
  tasks:
    - name: Update Nginx config
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: "reload web services"  # ← Не имя handler, а topic!
    
    - name: Update Apache config
      copy:
        src: apache.conf
        dest: /etc/apache2/apache2.conf
      notify: "reload web services"  # ← Тот же topic
  
  handlers:
    # ───────────────────────────────────────────────────────
    # Несколько handlers слушают один topic
    # ───────────────────────────────────────────────────────
    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
      listen: "reload web services"  # ← Слушает topic
    
    - name: Reload Apache
      service:
        name: apache2
        state: reloaded
      listen: "reload web services"  # ← Тот же topic
    
    - name: Clear cache
      file:
        path: /var/cache/web
        state: absent
      listen: "reload web services"  # ← Тоже выполнится
```

---

## 16. Gathering Facts

text

```
┌────────────────────────────────────────────────────────────┐
│         GATHERING FACTS                                    │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Gathering Facts — сбор информации о managed nodes        │
│                                                            │
│  ЧТО СОБИРАЕТСЯ:                                          │
│  • Операционная система и версия                          │
│  • Архитектура (x86_64, arm64)                            │
│  • Hostname, FQDN, domain                                 │
│  • Сетевые интерфейсы и IP адреса                         │
│  • CPU (количество ядер, модель)                          │
│  • RAM (total, free, swap)                                │
│  • Диски и файловые системы                               │
│  • Переменные окружения                                   │
│  • И многое другое (~100+ переменных)                     │
│                                                            │
│  КОГДА ВЫПОЛНЯЕТСЯ:                                       │
│  • В НАЧАЛЕ каждого play (по умолчанию)                   │
│  • Перед первым task                                      │
│  • Занимает время (~2-5 сек на хост)                      │
│                                                            │
│  КАК ИСПОЛЬЗУЕТСЯ:                                        │
│  • Переменные ansible_* доступны в tasks и templates      │
│  • Условия when (например, только для Ubuntu)             │
│  • Динамическая конфигурация                              │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Вывод при выполнении playbook:**

Bash

```
ansible-playbook playbook.yml

# PLAY [Configure webservers] *********************************************
# 
# TASK [Gathering Facts] **************************************************
# ok: [web1.example.com]
# ok: [web2.example.com]
# ↑ Эта стадия — сбор facts
# 
# TASK [Install Nginx] ****************************************************
# changed: [web1.example.com]
# ...
```
## 17. Использование Facts

YAML

```
# ═══════════════════════════════════════════════════════════
#          ИСПОЛЬЗОВАНИЕ ANSIBLE FACTS
# ═══════════════════════════════════════════════════════════

---
- name: Examples using facts
  hosts: all
  become: yes
  
  tasks:
    # ───────────────────────────────────────────────────────
    # 1. Узнать имя и версию дистрибутива
    # ───────────────────────────────────────────────────────
    - name: Display OS information
      debug:
        msg: |
          Distribution: {{ ansible_distribution }}
          Version: {{ ansible_distribution_version }}
          Release: {{ ansible_distribution_release }}
          OS Family: {{ ansible_os_family }}
      # Примеры значений:
      # ansible_distribution: Ubuntu, CentOS, RedHat, Debian
      # ansible_distribution_version: 22.04, 8.5, 11
      # ansible_distribution_release: jammy, focal
      # ansible_os_family: Debian, RedHat
    
    # ───────────────────────────────────────────────────────
    # 2. Условная установка пакетов по дистрибутиву
    # ───────────────────────────────────────────────────────
    - name: Install web server on Ubuntu/Debian
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian"
    
    - name: Install web server on CentOS/RHEL
      yum:
        name: nginx
        state: present
      when: ansible_os_family == "RedHat"
    
    # ───────────────────────────────────────────────────────
    # 3. Hostname и FQDN
    # ───────────────────────────────────────────────────────
    - name: Display hostname info
      debug:
        msg: |
          Hostname: {{ ansible_hostname }}
          FQDN: {{ ansible_fqdn }}
          Domain: {{ ansible_domain }}
      # ansible_hostname: web1
      # ansible_fqdn: web1.example.com
      # ansible_domain: example.com
    
    # ───────────────────────────────────────────────────────
    # 4. Архитектура системы
    # ───────────────────────────────────────────────────────
    - name: Display architecture
      debug:
        msg: |
          Architecture: {{ ansible_architecture }}
          Machine: {{ ansible_machine }}
      # ansible_architecture: x86_64, aarch64, armv7l
      # ansible_machine: x86_64
    
    # ───────────────────────────────────────────────────────
    # 5. CPU информация
    # ───────────────────────────────────────────────────────
    - name: Display CPU info
      debug:
        msg: |
          CPU Cores: {{ ansible_processor_vcpus }}
          CPU Count: {{ ansible_processor_count }}
          CPU Model: {{ ansible_processor[2] }}
      # ansible_processor_vcpus: 4 (виртуальные ядра)
      # ansible_processor_count: 2 (физические CPU)
      # ansible_processor: список с информацией о CPU
    
    # ───────────────────────────────────────────────────────
    # 6. Память (RAM)
    # ───────────────────────────────────────────────────────
    - name: Display memory info
      debug:
        msg: |
          Total RAM: {{ ansible_memtotal_mb }} MB
          Free RAM: {{ ansible_memfree_mb }} MB
          Swap Total: {{ ansible_swaptotal_mb }} MB
    
    # ───────────────────────────────────────────────────────
    # 7. Сетевые интерфейсы и IP адреса
    # ───────────────────────────────────────────────────────
    - name: Display network info
      debug:
        msg: |
          Default IPv4: {{ ansible_default_ipv4.address }}
          Gateway: {{ ansible_default_ipv4.gateway }}
          Interface: {{ ansible_default_ipv4.interface }}
          All IPs: {{ ansible_all_ipv4_addresses }}
      # ansible_default_ipv4.address: 192.168.1.10
      # ansible_default_ipv4.gateway: 192.168.1.1
      # ansible_default_ipv4.interface: eth0
      # ansible_all_ipv4_addresses: ["192.168.1.10", "10.0.0.5"]
    
    # ───────────────────────────────────────────────────────
    # 8. Диски и файловые системы
    # ───────────────────────────────────────────────────────
    - name: Display disk info
      debug:
        msg: |
          Devices: {{ ansible_devices.keys() | list }}
          Mounts: {{ ansible_mounts | map(attribute='mount') | list }}
      # ansible_devices: sda, sdb, nvme0n1
      # ansible_mounts: список примонтированных ФС
    
    # ───────────────────────────────────────────────────────
    # 9. Дата и время
    # ───────────────────────────────────────────────────────
    - name: Display date/time
      debug:
        msg: |
          Date: {{ ansible_date_time.date }}
          Time: {{ ansible_date_time.time }}
          ISO8601: {{ ansible_date_time.iso8601 }}
          Timezone: {{ ansible_date_time.tz }}
      # ansible_date_time.date: 2024-01-15
      # ansible_date_time.time: 10:30:45
      # ansible_date_time.iso8601: 2024-01-15T10:30:45Z
      # ansible_date_time.tz: UTC
    
    # ───────────────────────────────────────────────────────
    # 10. Переменные окружения
    # ───────────────────────────────────────────────────────
    - name: Display environment
      debug:
        msg: |
          Home: {{ ansible_env.HOME }}
          User: {{ ansible_env.USER }}
          Path: {{ ansible_env.PATH }}
    
    # ───────────────────────────────────────────────────────
    # 11. Пользователь, от которого выполняется
    # ───────────────────────────────────────────────────────
    - name: Display user info
      debug:
        msg: |
          User ID: {{ ansible_user_id }}
          User: {{ ansible_user_dir }}
          Real User: {{ ansible_real_user_id }}
    
    # ───────────────────────────────────────────────────────
    # 12. Kernel и система
    # ───────────────────────────────────────────────────────
    - name: Display kernel info
      debug:
        msg: |
          Kernel: {{ ansible_kernel }}
          System: {{ ansible_system }}
      # ansible_kernel: 5.15.0-91-generic
      # ansible_system: Linux
    
    # ───────────────────────────────────────────────────────
    # 13. Виртуализация
    # ───────────────────────────────────────────────────────
    - name: Display virtualization
      debug:
        msg: |
          Virtualization Type: {{ ansible_virtualization_type }}
          Virtualization Role: {{ ansible_virtualization_role }}
      # ansible_virtualization_type: kvm, docker, vmware, xen
      # ansible_virtualization_role: guest, host
    
    # ───────────────────────────────────────────────────────
    # 14. Python информация
    # ───────────────────────────────────────────────────────
    - name: Display Python info
      debug:
        msg: |
          Python Version: {{ ansible_python_version }}
          Python Executable: {{ ansible_python.executable }}
      # ansible_python_version: 3.10.12
      # ansible_python.executable: /usr/bin/python3
```

**Просмотр ВСЕХ facts:**

Bash

```
# ───────────────────────────────────────────────────────────
# Вывести ВСЕ facts конкретного хоста
# ───────────────────────────────────────────────────────────
ansible web1.example.com -m setup

# С фильтром (только IPv4)
ansible web1.example.com -m setup -a "filter=ansible_all_ipv4_addresses"

# Только сетевые интерфейсы
ansible web1.example.com -m setup -a "filter=ansible_*_ipv4"

# Сохранить в файл
ansible web1.example.com -m setup --tree /tmp/facts/
```

---

## 18. Пропуск Gathering Facts

YAML

```
# ═══════════════════════════════════════════════════════════
#          ПРОПУСК GATHERING FACTS
# ═══════════════════════════════════════════════════════════

---
# ───────────────────────────────────────────────────────────
# 1. Отключить для конкретного play
# ───────────────────────────────────────────────────────────
- name: Quick tasks without facts
  hosts: webservers
  gather_facts: no  # ← Не собирать facts
  
  tasks:
    - name: Ping
      ping:

# ───────────────────────────────────────────────────────────
# 2. Собрать facts вручную (когда нужно)
# ───────────────────────────────────────────────────────────
- name: Manual fact gathering
  hosts: all
  gather_facts: no
  
  tasks:
    - name: Do something without facts
      debug:
        msg: "No facts yet"
    
    # Собрать facts вручную
    - name: Gather facts now
      setup:
    
    - name: Now facts are available
      debug:
        msg: "OS: {{ ansible_distribution }}"

# ───────────────────────────────────────────────────────────
# 3. Собрать только подмножество facts (быстрее)
# ───────────────────────────────────────────────────────────
- name: Gather only network facts
  hosts: all
  gather_facts: no
  
  tasks:
    - name: Gather only network facts
      setup:
        gather_subset:
          - '!all'        # Отключить всё
          - '!min'        # Отключить минимальный набор
          - network       # Только сеть
      # Опции gather_subset:
      # - all: всё (по умолчанию)
      # - min: минимальный набор
      # - hardware: CPU, RAM, диски
      # - network: сетевые интерфейсы, IP
      # - virtual: виртуализация
      # - ohai: Ohai facts (если установлен)
      # - facter: Facter facts (если установлен)
      # Префикс ! = исключить

# ───────────────────────────────────────────────────────────
# 4. В ansible.cfg (глобально)
# ───────────────────────────────────────────────────────────
# ansible.cfg:
[defaults]
gathering = smart  # smart, explicit, implicit
# smart — собирать только если нет в кэше (по умолчанию)
# explicit — собирать только если gather_facts: yes
# implicit — всегда собирать

# Время кэша facts
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 86400  # 24 часа

# ───────────────────────────────────────────────────────────
# 5. Пример: ускорение для большого количества хостов
# ───────────────────────────────────────────────────────────
- name: Fast deployment
  hosts: all
  gather_facts: no  # Экономия ~3 сек на хост
  
  tasks:
    - name: Copy file
      copy:
        src: app.tar.gz
        dest: /tmp/
      # Facts не нужны — не собираем
```

**Когда стоит отключать facts:**

text

```
┌────────────────────────────────────────────────────────────┐
│         КОГДА ОТКЛЮЧАТЬ GATHERING FACTS?                   │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ✅ ОТКЛЮЧИТЬ если:                                        │
│  • Facts не используются в playbook                       │
│  • Простые задачи (ping, copy файла)                      │
│  • Большое количество хостов (1000+)                      │
│  • Критична скорость выполнения                           │
│  • Ad-hoc команды                                         │
│                                                            │
│  ❌ НЕ ОТКЛЮЧАТЬ если:                                     │
│  • Используете when с условиями по OS                     │
│  • Шаблоны (template) с переменными ansible_*             │
│  • Динамическая конфигурация по характеристикам хоста     │
│  • Условная установка пакетов (apt vs yum)                │
│                                                            │
│  ЭКОНОМИЯ ВРЕМЕНИ:                                        │
│  • 1 хост: ~2-3 секунды                                   │
│  • 100 хостов: ~200-300 секунд (5 минут)                  │
│  • 1000 хостов: ~2000-3000 секунд (50 минут!)             │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## 19. Поиск и замена в файле (регулярные выражения)

YAML

```
# ═══════════════════════════════════════════════════════════
#          ПОИСК И ЗАМЕНА В ФАЙЛЕ
# ═══════════════════════════════════════════════════════════

---
- name: Find and replace examples
  hosts: all
  become: yes
  
  tasks:
    # ───────────────────────────────────────────────────────
    # 1. LINEINFILE — изменить/добавить строку
    # ───────────────────────────────────────────────────────
    
    # Простая замена (без regex)
    - name: Replace exact line
      lineinfile:
        path: /etc/hosts
        regexp: '^127\.0\.0\.1'
        line: '127.0.0.1 localhost myhost'
        state: present
      # Если строка с таким regexp существует → заменить
      # Если нет → добавить в конец
    
    # С regex группами
    - name: Replace with backreferences
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?Port\s+(\d+)'
        line: 'Port 2222'
        backrefs: yes
      # backrefs: yes — использовать группы \1, \2...
      # Если строка НЕ найдена → НЕ добавлять (в отличие от обычного режима)
    
    # Удалить строку
    - name: Remove line
      lineinfile:
        path: /etc/hosts
        regexp: '^192\.168\.1\.100'
        state: absent
    
    # Добавить после определённой строки
    - name: Add line after match
      lineinfile:
        path: /etc/fstab
        line: '/dev/sdb1 /data ext4 defaults 0 0'
        insertafter: '^# /data'
    
    # Добавить перед определённой строкой
    - name: Add line before match
      lineinfile:
        path: /etc/hosts
        line: '10.0.0.5 db.example.com'
        insertbefore: '^127\.0\.0\.1'
    
    # Добавить в начало файла
    - name: Add to beginning
      lineinfile:
        path: /etc/resolv.conf
        line: 'nameserver 8.8.8.8'
        insertbefore: BOF  # Beginning Of File
    
    # Добавить в конец файла
    - name: Add to end
      lineinfile:
        path: /etc/hosts
        line: '192.168.1.200 extra.host'
        insertafter: EOF  # End Of File
    
    # ───────────────────────────────────────────────────────
    # 2. REPLACE — замена ВСЕХ вхождений в файле
    # ───────────────────────────────────────────────────────
    
    # Простая замена
    - name: Replace all occurrences
      replace:
        path: /etc/nginx/nginx.conf
        regexp: 'worker_connections\s+\d+'
        replace: 'worker_connections 2048'
      # Заменит ВСЕ вхождения (не только первое)
    
    # С группами захвата
    - name: Replace with backreferences
      replace:
        path: /var/www/html/config.php
        regexp: "define\\('DB_HOST', '([^']+)'\\)"
        replace: "define('DB_HOST', 'localhost')"
        backup: yes  # Создать backup перед изменением
    
    # Замена только в определённой секции
    - name: Replace in section
      replace:
        path: /etc/nginx/nginx.conf
        regexp: '^(\s+)listen\s+80;'
        replace: '\1listen 8080;'
        after: 'server {'        # Только после этой строки
        before: '}'              # И до этой строки
    
    # Многострочная замена
    - name: Multiline replace
      replace:
        path: /etc/config.txt
        regexp: 'old_value_(\d+)'
        replace: 'new_value_\1'
    
    # ───────────────────────────────────────────────────────
    # 3. REGEX в TEMPLATE (для генерации файлов)
    # ───────────────────────────────────────────────────────
    
    - name: Generate config with regex filter
      template:
        src: config.j2
        dest: /etc/myapp/config.txt
      vars:
        old_value: "server: old-server.com"
        # В шаблоне:
        # {{ old_value | regex_replace('old-server', 'new-server') }}
        # Результат: "server: new-server.com"
    
    # ───────────────────────────────────────────────────────
    # 4. Сложные примеры
    # ───────────────────────────────────────────────────────
    
    # Изменить несколько параметров
    - name: Update multiple SSH settings
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "{{ item.regexp }}"
        line: "{{ item.line }}"
        state: present
      loop:
        - { regexp: '^#?Port\s', line: 'Port 2222' }
        - { regexp: '^#?PermitRootLogin\s', line: 'PermitRootLogin no' }
        - { regexp: '^#?PasswordAuthentication\s', line: 'PasswordAuthentication no' }
      notify: Restart SSH
    
    # Закомментировать строку
    - name: Comment out line
      replace:
        path: /etc/default/grub
        regexp: '^(GRUB_TIMEOUT=\d+)$'
        replace: '# \1'
    
    # Раскомментировать строку
    - name: Uncomment line
      replace:
        path: /etc/sysctl.conf
        regexp: '^#\s*(net\.ipv4\.ip_forward=1)'
        replace: '\1'
    
    # Удалить пустые строки
    - name: Remove empty lines
      replace:
        path: /etc/hosts
        regexp: '^\s*$\n'
        replace: ''
    
    # Удалить trailing whitespace
    - name: Remove trailing whitespace
      replace:
        path: /etc/config
        regexp: '\s+$'
        replace: ''
  
  handlers:
    - name: Restart SSH
      service:
        name: sshd
        state: restarted
```

**Примеры regex паттернов:**

YAML

```
# ═══════════════════════════════════════════════════════════
#          ПОЛЕЗНЫЕ REGEX ПАТТЕРНЫ
# ═══════════════════════════════════════════════════════════

# IP адрес
regexp: '\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b'

# Email
regexp: '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'

# URL
regexp: 'https?://[^\s]+'

# Начало строки (^) и конец ($)
regexp: '^ServerName\s+.*$'

# Любой символ (.) и ноль или более (*)
regexp: 'listen\s+\d+.*'

# Один или более (+)
regexp: 'worker_processes\s+\d+'

# Ноль или один (?)
regexp: '^#?Port\s+\d+'  # Строка может начинаться с # или нет

# Группы захвата ()
regexp: '^(ServerName)\s+(.*)$'
replace: '\1 new.server.com'  # \1 = ServerName, \2 = старое значение

# Классы символов
regexp: '[0-9]+' # Цифры
regexp: '[a-zA-Z]+' # Буквы
regexp: '[^abc]' # Всё КРОМЕ a, b, c

# Escape спецсимволов
regexp: '\.'  # Точка (литерал)
regexp: '\$'  # Доллар
regexp: '\('  # Скобка
regexp: '\\'  # Обратный слэш
```

---

## 20. Регистрация переменной в runtime

YAML

```
# ═══════════════════════════════════════════════════════════
#          REGISTER — сохранение результата task
# ═══════════════════════════════════════════════════════════

---
- name: Register variable examples
  hosts: all
  become: yes
  
  tasks:
    # ───────────────────────────────────────────────────────
    # 1. Сохранить результат команды
    # ───────────────────────────────────────────────────────
    - name: Check if file exists
      stat:
        path: /etc/myapp/config.txt
      register: config_file
    
    - name: Display result
      debug:
        msg: |
          File exists: {{ config_file.stat.exists }}
          Size: {{ config_file.stat.size | default(0) }}
          Owner: {{ config_file.stat.pw_name | default('unknown') }}
    
    # Использовать в условии
    - name: Create file if not exists
      file:
        path: /etc/myapp/config.txt
        state: touch
      when: not config_file.stat.exists
    
    # ───────────────────────────────────────────────────────
    # 2. Сохранить вывод shell команды
    # ───────────────────────────────────────────────────────
    - name: Get disk usage
      shell: df -h / | tail -1 | awk '{print $5}'
      register: disk_usage
      changed_when: false  # Не помечать как changed
    
    - name: Show disk usage
      debug:
        msg: "Disk usage: {{ disk_usage.stdout }}"
      # disk_usage.stdout — вывод команды
      # disk_usage.stderr — ошибки
      # disk_usage.rc — exit code
    
    # Условие на основе вывода
    - name: Warn if disk full
      debug:
        msg: "WARNING: Disk is more than 80% full!"
      when: disk_usage.stdout | replace('%', '') | int > 80
    
    # ───────────────────────────────────────────────────────
    # 3. Сохранить результат с exit code
    # ───────────────────────────────────────────────────────
    - name: Check if service exists
      command: systemctl status nginx
      register: nginx_status
      failed_when: false  # Не падать если команда вернула != 0
      changed_when: false
    
    - name: Service is running
      debug:
        msg: "Nginx is running"
      when: nginx_status.rc == 0
    
    - name: Service is not running
      debug:
        msg: "Nginx is NOT running"
      when: nginx_status.rc != 0
    
    # ───────────────────────────────────────────────────────
    # 4. Сохранить список (с loop)
    # ───────────────────────────────────────────────────────
    - name: Find all log files
      find:
        paths: /var/log
        patterns: '*.log'
        recurse: no
      register: log_files
    
    - name: Display found files
      debug:
        msg: "{{ log_files.files | map(attribute='path') | list }}"
    
    - name: Delete old log files
      file:
        path: "{{ item.path }}"
        state: absent
      loop: "{{ log_files.files }}"
      when: item.size > 100000000  # > 100 MB
    
    # ───────────────────────────────────────────────────────
    # 5. Сохранить результат с loop
    # ───────────────────────────────────────────────────────
    - name: Check multiple services
      service_facts:
      register: services
    
    - name: Ping multiple hosts
      command: ping -c 1 {{ item }}
      loop:
        - google.com
        - github.com
        - gitlab.com
      register: ping_results
      failed_when: false
      changed_when: false
    
    # ping_results.results — список результатов
    - name: Show ping results
      debug:
        msg: "{{ item.item }}: {{ 'UP' if item.rc == 0 else 'DOWN' }}"
      loop: "{{ ping_results.results }}"
    
    # ───────────────────────────────────────────────────────
    # 6. Комплексный пример с set_fact
    # ───────────────────────────────────────────────────────
    - name: Get IP addresses
      shell: hostname -I | awk '{print $1}'
      register: ip_output
      changed_when: false
    
    # Сохранить в переменную для использования дальше
    - name: Set fact
      set_fact:
        primary_ip: "{{ ip_output.stdout }}"
    
    - name: Use registered fact
      debug:
        msg: "Primary IP: {{ primary_ip }}"
    
    # ───────────────────────────────────────────────────────
    # 7. Проверка успешности предыдущих tasks
    # ───────────────────────────────────────────────────────
    - name: Task that might fail
      command: /opt/script.sh
      register: script_result
      ignore_errors: yes  # Продолжить даже при ошибке
    
    - name: Rollback if failed
      command: /opt/rollback.sh
      when: script_result is failed
    
    - name: Success actions
      command: /opt/cleanup.sh
      when: script_result is succeeded
    
    # ───────────────────────────────────────────────────────
    # 8. JSON парсинг
    # ───────────────────────────────────────────────────────
    - name: Get JSON from API
      uri:
        url: https://api.github.com/users/ansible
        return_content: yes
      register: api_response
    
    - name: Parse JSON
      debug:
        msg: |
          Name: {{ api_response.json.name }}
          Repos: {{ api_response.json.public_repos }}
      # api_response.json — автоматически распарсенный JSON
    
    # ───────────────────────────────────────────────────────
    # 9. Передача между tasks
    # ───────────────────────────────────────────────────────
    - name: Generate password
      command: openssl rand -base64 32
      register: generated_password
      changed_when: false
    
    - name: Create user with generated password
      user:
        name: newuser
        password: "{{ generated_password.stdout | password_hash('sha512') }}"
        state: present
    
    # ───────────────────────────────────────────────────────
    # 10. Сохранить в файл для следующего run
    # ───────────────────────────────────────────────────────
    - name: Get current timestamp
      command: date +%s
      register: timestamp
      changed_when: false
    
    - name: Save to file
      copy:
        content: "{{ timestamp.stdout }}"
        dest: /tmp/last_run.txt
    
    - name: Read from file
      slurp:
        src: /tmp/last_run.txt
      register: last_run
      ignore_errors: yes
    
    - name: Compare timestamps
      debug:
        msg: "Seconds since last run: {{ timestamp.stdout | int - (last_run.content | b64decode | trim | int) }}"
      when: last_run is succeeded
```

**Структура registered переменной:**

YAML

```
# Пример registered переменной:
registered_var:
  changed: true              # Был ли изменён хост
  failed: false              # Упал ли task
  skipped: false             # Был ли пропущен
  rc: 0                      # Return code (для command/shell)
  stdout: "output text"      # Стандартный вывод
  stdout_lines: ["line1", "line2"]  # Вывод построчно
  stderr: ""                 # Ошибки
  stderr_lines: []
  cmd: "echo test"           # Выполненная команда
  start: "2024-01-15 10:00"  # Время начала
  end: "2024-01-15 10:00"    # Время окончания
  delta: "0:00:00.123"       # Длительность
  
  # Для модулей типа stat, find:
  stat:
    exists: true
    path: "/etc/hosts"
    size: 1234
    mode: "0644"
    uid: 0
    gid: 0
  
  # Для uri модуля:
  json:                      # Распарсенный JSON
    key: "value"
  status: 200                # HTTP status code
  content: "..."             # Сырой ответ
```
## 21. Ansible Galaxy

text

```
┌────────────────────────────────────────────────────────────┐
│         ANSIBLE GALAXY                                     │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Ansible Galaxy — репозиторий готовых ролей и коллекций   │
│                                                            │
│  ЧТО ТАКОЕ:                                               │
│  • https://galaxy.ansible.com                             │
│  • Публичный репозиторий (как Docker Hub для Ansible)     │
│  • Готовые роли от сообщества и компаний                  │
│  • Можно использовать бесплатно                           │
│                                                            │
│  ЧТО МОЖНО НАЙТИ:                                         │
│  • Роли (nginx, postgresql, docker, kubernetes)           │
│  • Коллекции (модули, плагины, роли)                      │
│  • От официальных вендоров (AWS, Azure, Kubernetes)       │
│  • От сообщества                                          │
│                                                            │
│  ЗАЧЕМ ИСПОЛЬЗОВАТЬ:                                      │
│  • Не изобретать велосипед                                │
│  • Протестированный код                                   │
│  • Best practices                                         │
│  • Экономия времени                                       │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Установка ролей

Bash

```
# ═══════════════════════════════════════════════════════════
#          УСТАНОВКА РОЛЕЙ ИЗ GALAXY
# ═══════════════════════════════════════════════════════════

# ───────────────────────────────────────────────────────────
# Поиск роли
# ───────────────────────────────────────────────────────────
ansible-galaxy search nginx

# Результат:
# Found 234 roles matching your search:
# 
# Name                         Description
# ----                         -----------
# geerlingguy.nginx            Nginx installation for Linux, FreeBSD and OpenBSD
# jdauphant.nginx              Install and configure nginx

# ───────────────────────────────────────────────────────────
# Информация о роли
# ───────────────────────────────────────────────────────────
ansible-galaxy info geerlingguy.nginx

# Результат:
# Role: geerlingguy.nginx
# Description: Nginx installation for Linux, FreeBSD and OpenBSD
# Author: geerlingguy
# Stars: 2542
# Downloads: 15M
# ...

# ───────────────────────────────────────────────────────────
# Установка роли
# ───────────────────────────────────────────────────────────
ansible-galaxy install geerlingguy.nginx

# В определённую директорию
ansible-galaxy install geerlingguy.nginx -p ./roles/

# Конкретная версия
ansible-galaxy install geerlingguy.nginx,3.1.4

# ───────────────────────────────────────────────────────────
# Установка из requirements.yml (рекомендуется)
# ───────────────────────────────────────────────────────────
ansible-galaxy install -r requirements.yml

# ───────────────────────────────────────────────────────────
# Обновить все роли
# ───────────────────────────────────────────────────────────
ansible-galaxy install -r requirements.yml --force

# ───────────────────────────────────────────────────────────
# Список установленных ролей
# ───────────────────────────────────────────────────────────
ansible-galaxy list

# Результат:
# - geerlingguy.nginx, 3.1.4
# - geerlingguy.postgresql, 3.4.3
# - geerlingguy.docker, 6.1.0

# ───────────────────────────────────────────────────────────
# Удалить роль
# ───────────────────────────────────────────────────────────
ansible-galaxy remove geerlingguy.nginx
```

### requirements.yml

YAML

```
# ═══════════════════════════════════════════════════════════
#          requirements.yml
# ═══════════════════════════════════════════════════════════

---
# ───────────────────────────────────────────────────────────
# Роли из Ansible Galaxy
# ───────────────────────────────────────────────────────────
roles:
  # Простая установка (последняя версия)
  - name: geerlingguy.nginx
  
  # С конкретной версией
  - name: geerlingguy.postgresql
    version: 3.4.3
  
  # Из Git репозитория
  - name: my-custom-role
    src: https://github.com/user/ansible-role-custom.git
    version: main
  
  # Из Git с переименованием
  - name: custom_nginx
    src: https://github.com/company/nginx-role.git
    version: v1.0.0
  
  # Из локальной директории
  - name: local-role
    src: /path/to/local/role
  
  # С переменными по умолчанию
  - name: geerlingguy.docker
    version: 6.1.0

# ───────────────────────────────────────────────────────────
# Коллекции из Ansible Galaxy
# ───────────────────────────────────────────────────────────
collections:
  # Официальная коллекция AWS
  - name: amazon.aws
    version: 5.5.0
  
  # Kubernetes
  - name: kubernetes.core
    version: 2.4.0
  
  # Community General (огромная коллекция модулей)
  - name: community.general
    version: 7.3.0
  
  # Из Git
  - name: my_namespace.my_collection
    source: https://github.com/company/ansible-collection.git
    type: git
    version: main
```

**Установка из requirements.yml:**

Bash

```
# Установить роли
ansible-galaxy install -r requirements.yml

# Установить коллекции
ansible-galaxy collection install -r requirements.yml

# Установить всё (роли + коллекции)
ansible-galaxy install -r requirements.yml
ansible-galaxy collection install -r requirements.yml
```

### Использование установленной роли

YAML

```
# ═══════════════════════════════════════════════════════════
#          ИСПОЛЬЗОВАНИЕ GALAXY РОЛЕЙ
# ═══════════════════════════════════════════════════════════

---
- name: Setup web servers
  hosts: webservers
  become: yes
  
  # ───────────────────────────────────────────────────────
  # Вариант 1: Простое использование
  # ───────────────────────────────────────────────────────
  roles:
    - geerlingguy.nginx
  
  # ───────────────────────────────────────────────────────
  # Вариант 2: С переменными
  # ───────────────────────────────────────────────────────
  roles:
    - role: geerlingguy.nginx
      vars:
        nginx_vhosts:
          - listen: "80"
            server_name: "example.com"
            root: "/var/www/example.com"
            index: "index.html"
        nginx_remove_default_vhost: true

# ═══════════════════════════════════════════════════════════
#          КОМБИНИРОВАНИЕ НЕСКОЛЬКИХ РОЛЕЙ
# ═══════════════════════════════════════════════════════════

---
- name: Full stack setup
  hosts: webservers
  become: yes
  
  vars:
    # PostgreSQL переменные
    postgresql_databases:
      - name: myapp_db
    postgresql_users:
      - name: myapp_user
        password: secret
    
    # Docker переменные
    docker_users:
      - deploy
  
  roles:
    # 1. Установить Docker
    - geerlingguy.docker
    
    # 2. Установить PostgreSQL
    - geerlingguy.postgresql
    
    # 3. Установить Nginx
    - geerlingguy.nginx
    
    # 4. Настроить firewall
    - geerlingguy.firewall
  
  tasks:
    # Дополнительные tasks после ролей
    - name: Deploy application
      docker_container:
        name: myapp
        image: myapp:latest
        ports:
          - "8080:8080"
```

### Коллекции

text

```
┌────────────────────────────────────────────────────────────┐
│         ANSIBLE COLLECTIONS                                │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Collection — пакет, содержащий:                          │
│  • Модули                                                 │
│  • Плагины                                                │
│  • Роли                                                   │
│  • Playbooks                                              │
│  • Документацию                                           │
│                                                            │
│  ОТЛИЧИЕ ОТ РОЛИ:                                         │
│  • Роль — один компонент (nginx, postgresql)              │
│  • Коллекция — набор связанных компонентов (AWS, K8s)     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

Bash

```
# ═══════════════════════════════════════════════════════════
#          РАБОТА С КОЛЛЕКЦИЯМИ
# ═══════════════════════════════════════════════════════════

# ───────────────────────────────────────────────────────────
# Установка коллекции
# ───────────────────────────────────────────────────────────
ansible-galaxy collection install amazon.aws

# Конкретная версия
ansible-galaxy collection install amazon.aws:5.5.0

# Из requirements.yml
ansible-galaxy collection install -r requirements.yml

# ───────────────────────────────────────────────────────────
# Список установленных коллекций
# ───────────────────────────────────────────────────────────
ansible-galaxy collection list

# Результат:
# Collection        Version
# ----------------- -------
# amazon.aws        5.5.0
# kubernetes.core   2.4.0
# community.general 7.3.0

# ───────────────────────────────────────────────────────────
# Обновить коллекцию
# ───────────────────────────────────────────────────────────
ansible-galaxy collection install amazon.aws --force

# ───────────────────────────────────────────────────────────
# Где хранятся коллекции
# ───────────────────────────────────────────────────────────
ansible-galaxy collection list

# По умолчанию:
# ~/.ansible/collections/ansible_collections/
# /usr/share/ansible/collections/ansible_collections/
```

**Использование модулей из коллекций:**

YAML

```
# ═══════════════════════════════════════════════════════════
#          ИСПОЛЬЗОВАНИЕ МОДУЛЕЙ ИЗ КОЛЛЕКЦИЙ
# ═══════════════════════════════════════════════════════════

---
- name: AWS example
  hosts: localhost
  gather_facts: no
  
  collections:
    # Объявить используемые коллекции
    - amazon.aws
  
  tasks:
    # ───────────────────────────────────────────────────────
    # Вариант 1: С префиксом коллекции
    # ───────────────────────────────────────────────────────
    - name: Create EC2 instance (full name)
      amazon.aws.ec2_instance:
        name: web-server
        instance_type: t2.micro
        image_id: ami-0c55b159cbfafe1f0
        region: us-east-1
        state: present
    
    # ───────────────────────────────────────────────────────
    # Вариант 2: Без префикса (после collections:)
    # ───────────────────────────────────────────────────────
    - name: Create EC2 instance (short name)
      ec2_instance:  # Без amazon.aws.
        name: web-server
        instance_type: t2.micro
        image_id: ami-0c55b159cbfafe1f0
        region: us-east-1
        state: present
```

**Популярные коллекции:**

YAML

```
# ═══════════════════════════════════════════════════════════
#          ПОПУЛЯРНЫЕ КОЛЛЕКЦИИ
# ═══════════════════════════════════════════════════════════

collections:
  # ─────────────────────────────────────────────────────────
  # ОБЛАЧНЫЕ ПРОВАЙДЕРЫ
  # ─────────────────────────────────────────────────────────
  - name: amazon.aws           # AWS (EC2, S3, RDS, etc.)
  - name: azure.azcollection   # Microsoft Azure
  - name: google.cloud         # Google Cloud Platform
  - name: openstack.cloud      # OpenStack
  
  # ─────────────────────────────────────────────────────────
  # ОРКЕСТРАЦИЯ И КОНТЕЙНЕРЫ
  # ─────────────────────────────────────────────────────────
  - name: kubernetes.core      # Kubernetes
  - name: containers.podman    # Podman
  - name: community.docker     # Docker
  
  # ─────────────────────────────────────────────────────────
  # СЕТЕВОЕ ОБОРУДОВАНИЕ
  # ─────────────────────────────────────────────────────────
  - name: cisco.ios            # Cisco IOS
  - name: cisco.nxos           # Cisco NX-OS
  - name: junipernetworks.junos # Juniper Junos
  - name: arista.eos           # Arista EOS
  
  # ─────────────────────────────────────────────────────────
  # БАЗЫ ДАННЫХ
  # ─────────────────────────────────────────────────────────
  - name: community.mysql      # MySQL/MariaDB
  - name: community.postgresql # PostgreSQL
  - name: community.mongodb    # MongoDB
  
  # ─────────────────────────────────────────────────────────
  # МОНИТОРИНГ И ЛОГИРОВАНИЕ
  # ─────────────────────────────────────────────────────────
  - name: community.grafana    # Grafana
  - name: prometheus.prometheus # Prometheus
  
  # ─────────────────────────────────────────────────────────
  # ОБЩЕГО НАЗНАЧЕНИЯ
  # ─────────────────────────────────────────────────────────
  - name: community.general    # Огромная коллекция доп. модулей
  - name: ansible.posix        # POSIX утилиты
  - name: ansible.windows      # Windows модули
```

---

## 22. Поиск файлов и сохранение в переменную

YAML

```
# ═══════════════════════════════════════════════════════════
#          ПОИСК ФАЙЛОВ И СОХРАНЕНИЕ В ПЕРЕМЕННУЮ
# ═══════════════════════════════════════════════════════════

---
- name: Find and process files
  hosts: all
  become: yes
  
  tasks:
    # ───────────────────────────────────────────────────────
    # 1. Найти все .log файлы
    # ───────────────────────────────────────────────────────
    - name: Find all .log files
      find:
        paths: /var/log
        patterns: '*.log'
        recurse: yes              # Рекурсивно
        file_type: file           # Только файлы (не директории)
      register: log_files
    
    # Структура log_files:
    # log_files:
    #   matched: 15              # Количество найденных
    #   files:
    #     - path: /var/log/syslog
    #       size: 1234567
    #       mode: "0644"
    #       uid: 0
    #       ...
    
    - name: Display found files count
      debug:
        msg: "Found {{ log_files.matched }} log files"
    
    # ───────────────────────────────────────────────────────
    # 2. Сохранить только пути в переменную
    # ───────────────────────────────────────────────────────
    - name: Extract paths to variable
      set_fact:
        log_file_paths: "{{ log_files.files | map(attribute='path') | list }}"
    
    - name: Display paths
      debug:
        var: log_file_paths
      # Результат:
      # log_file_paths:
      #   - /var/log/syslog
      #   - /var/log/auth.log
      #   - /var/log/nginx/access.log
      #   - /var/log/nginx/error.log
    
    # ───────────────────────────────────────────────────────
    # 3. Найти файлы по сложным критериям
    # ───────────────────────────────────────────────────────
    - name: Find large old log files
      find:
        paths:
          - /var/log
          - /opt/app/logs
        patterns:
          - '*.log'
          - '*.log.*'
        excludes:
          - 'important.log'       # Исключить
        recurse: yes
        age: 7d                   # Старше 7 дней
        size: 100m                # Больше 100 MB
        file_type: file
      register: old_large_logs
    
    - name: Save paths of large old logs
      set_fact:
        cleanup_files: "{{ old_large_logs.files | map(attribute='path') | list }}"
    
    # ───────────────────────────────────────────────────────
    # 4. Найти файлы и обработать каждый
    # ───────────────────────────────────────────────────────
    - name: Archive each log file
      archive:
        path: "{{ item.path }}"
        dest: "{{ item.path }}.gz"
        remove: yes
      loop: "{{ log_files.files }}"
      when: item.size > 10485760  # > 10 MB
    
    # ───────────────────────────────────────────────────────
    # 5. Найти файлы с определённым содержимым
    # ───────────────────────────────────────────────────────
    - name: Find all nginx config files
      find:
        paths: /etc/nginx
        patterns: '*.conf'
        recurse: yes
      register: nginx_configs
    
    - name: Search for errors in configs
      shell: grep -l "error" {{ item.path }}
      loop: "{{ nginx_configs.files }}"
      register: configs_with_errors
      failed_when: false
      changed_when: false
    
    - name: Extract configs with errors
      set_fact:
        error_configs: >-
          {{
            configs_with_errors.results
            | selectattr('rc', 'equalto', 0)
            | map(attribute='item.path')
            | list
          }}
    
    - name: Display configs with errors
      debug:
        var: error_configs
    
    # ───────────────────────────────────────────────────────
    # 6. Найти и удалить старые файлы
    # ───────────────────────────────────────────────────────
    - name: Find old temp files
      find:
        paths: /tmp
        patterns: 'tmp*'
        age: 30d
        file_type: file
      register: old_temp_files
    
    - name: Delete old temp files
      file:
        path: "{{ item.path }}"
        state: absent
      loop: "{{ old_temp_files.files }}"
    
    # ───────────────────────────────────────────────────────
    # 7. Подсчёт общего размера
    # ───────────────────────────────────────────────────────
    - name: Calculate total size of logs
      set_fact:
        total_log_size: "{{ log_files.files | map(attribute='size') | sum }}"
    
    - name: Display total size
      debug:
        msg: "Total size: {{ (total_log_size | int / 1024 / 1024) | round(2) }} MB"
    
    # ───────────────────────────────────────────────────────
    # 8. Группировка по директориям
    # ───────────────────────────────────────────────────────
    - name: Group files by directory
      set_fact:
        files_by_dir: >-
          {{
            log_files.files
            | map('dirname')
            | unique
            | list
          }}
    
    - name: Display directories
      debug:
        msg: "Log directories: {{ files_by_dir }}"
    
    # ───────────────────────────────────────────────────────
    # 9. Найти самый большой файл
    # ───────────────────────────────────────────────────────
    - name: Find largest file
      set_fact:
        largest_file: "{{ log_files.files | sort(attribute='size', reverse=true) | first }}"
      when: log_files.matched > 0
    
    - name: Display largest file
      debug:
        msg: >-
          Largest file: {{ largest_file.path }}
          ({{ (largest_file.size / 1024 / 1024) | round(2) }} MB)
      when: largest_file is defined
    
    # ───────────────────────────────────────────────────────
    # 10. Сложная фильтрация
    # ───────────────────────────────────────────────────────
    - name: Find specific files
      find:
        paths: /var/log
        patterns:
          - 'access.log*'
          - 'error.log*'
        use_regex: no
        recurse: yes
        age: 1d
        age_stamp: mtime         # mtime, atime, ctime
      register: recent_logs
    
    - name: Filter and transform
      set_fact:
        processed_logs: >-
          {{
            recent_logs.files
            | selectattr('size', 'gt', 1048576)    # > 1 MB
            | map(attribute='path')
            | map('regex_replace', '^/var/log/', '')
            | list
          }}
    
    - name: Display processed
      debug:
        var: processed_logs
```

**Дополнительные параметры find:**

YAML

```
# ═══════════════════════════════════════════════════════════
#          ПАРАМЕТРЫ МОДУЛЯ FIND
# ═══════════════════════════════════════════════════════════

- name: Find examples
  find:
    # ───────────────────────────────────────────────────────
    # ОСНОВНЫЕ ПАРАМЕТРЫ
    # ───────────────────────────────────────────────────────
    paths:
      - /var/log
      - /opt/app/logs
    patterns:
      - '*.log'
      - '*.txt'
    excludes:
      - 'debug.log'
      - 'temp*'
    
    # ───────────────────────────────────────────────────────
    # РЕКУРСИЯ
    # ───────────────────────────────────────────────────────
    recurse: yes                 # Искать в поддиректориях
    depth: 2                     # Максимальная глубина рекурсии
    
    # ───────────────────────────────────────────────────────
    # ТИП ФАЙЛА
    # ───────────────────────────────────────────────────────
    file_type: file              # file, directory, link, any
    
    # ───────────────────────────────────────────────────────
    # РАЗМЕР
    # ───────────────────────────────────────────────────────
    size: 10m                    # Точно 10 MB
    size: +10m                   # Больше 10 MB
    size: -10m                   # Меньше 10 MB
    # Единицы: b, k, m, g, t
    
    # ───────────────────────────────────────────────────────
    # ВОЗРАСТ
    # ───────────────────────────────────────────────────────
    age: 7d                      # Точно 7 дней
    age: +7d                     # Старше 7 дней
    age: -7d                     # Моложе 7 дней
    age: 1w                      # 1 неделя
    age: 2h                      # 2 часа
    # Единицы: s, m, h, d, w
    
    age_stamp: mtime             # mtime (modified), atime (accessed), ctime (created)
    
    # ───────────────────────────────────────────────────────
    # ПРАВА ДОСТУПА
    # ───────────────────────────────────────────────────────
    mode: '0644'                 # Точно 0644
    
    # ───────────────────────────────────────────────────────
    # REGEX
    # ───────────────────────────────────────────────────────
    use_regex: yes               # Использовать regex в patterns
    patterns:
      - '.*\.log$'
      - '^access.*'
    
    # ───────────────────────────────────────────────────────
    # СОДЕРЖИМОЕ
    # ───────────────────────────────────────────────────────
    contains: 'ERROR'            # Содержит строку
    
    # ───────────────────────────────────────────────────────
    # СКРЫТЫЕ ФАЙЛЫ
    # ───────────────────────────────────────────────────────
    hidden: yes                  # Включить скрытые файлы
    
    # ───────────────────────────────────────────────────────
    # СЛЕДОВАТЬ СИМЛИНКАМ
    # ───────────────────────────────────────────────────────
    follow: yes                  # Следовать за симлинками
    
    # ───────────────────────────────────────────────────────
    # ПРОИЗВОДИТЕЛЬНОСТЬ
    # ───────────────────────────────────────────────────────
    get_checksum: no             # Не вычислять checksum (быстрее)
```

---

## 23. Lookup плагин

text

```
┌────────────────────────────────────────────────────────────┐
│         LOOKUP ПЛАГИНЫ                                     │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Lookup — получение данных ИЗ ВНЕШНИХ ИСТОЧНИКОВ          │
│                                                            │
│  КОГДА ВЫПОЛНЯЕТСЯ:                                       │
│  • На CONTROL NODE (не на managed node)                   │
│  • Во время выполнения playbook                           │
│                                                            │
│  ИСТОЧНИКИ:                                               │
│  • Файлы                                                  │
│  • Переменные окружения                                   │
│  • DNS                                                    │
│  • Базы данных                                            │
│  • Хранилища секретов (Vault, AWS Secrets Manager)        │
│  • URL                                                    │
│  • Консольные команды                                     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

YAML

```
# ═══════════════════════════════════════════════════════════
#          LOOKUP ПЛАГИНЫ — ПРИМЕРЫ
# ═══════════════════════════════════════════════════════════

---
- name: Lookup examples
  hosts: localhost
  gather_facts: no
  
  vars:
    # ───────────────────────────────────────────────────────
    # 1. FILE — прочитать файл
    # ───────────────────────────────────────────────────────
    ssh_key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
    nginx_config: "{{ lookup('file', '/etc/nginx/nginx.conf') }}"
    
    # ───────────────────────────────────────────────────────
    # 2. ENV — переменная окружения
    # ───────────────────────────────────────────────────────
    current_user: "{{ lookup('env', 'USER') }}"
    home_dir: "{{ lookup('env', 'HOME') }}"
    path: "{{ lookup('env', 'PATH') }}"
    
    # С default значением если не найдено
    custom_var: "{{ lookup('env', 'CUSTOM_VAR') | default('default_value') }}"
    
    # ───────────────────────────────────────────────────────
    # 3. PIPE — вывод shell команды
    # ───────────────────────────────────────────────────────
    current_date: "{{ lookup('pipe', 'date +%Y-%m-%d') }}"
    hostname: "{{ lookup('pipe', 'hostname') }}"
    git_commit: "{{ lookup('pipe', 'git rev-parse --short HEAD') }}"
    
    # ───────────────────────────────────────────────────────
    # 4. TEMPLATE — обработать Jinja2 шаблон
    # ───────────────────────────────────────────────────────
    rendered_config: "{{ lookup('template', 'templates/config.j2') }}"
    
    # ───────────────────────────────────────────────────────
    # 5. PASSWORD — генерация паролей
    # ───────────────────────────────────────────────────────
    # Генерирует случайный пароль и сохраняет в файл
    # При повторном вызове возвращает ТОТ ЖЕ пароль
    db_password: "{{ lookup('password', '/tmp/db_password length=32') }}"
    
    # С определёнными символами
    admin_password: "{{ lookup('password', '/tmp/admin_pass length=16 chars=ascii_letters,digits') }}"
    
    # ───────────────────────────────────────────────────────
    # 6. FILEGLOB — список файлов по паттерну
    # ───────────────────────────────────────────────────────
    log_files: "{{ lookup('fileglob', '/var/log/*.log', wantlist=True) }}"
    
    # ───────────────────────────────────────────────────────
    # 7. LINES — построчное чтение
    # ───────────────────────────────────────────────────────
    users_list: "{{ lookup('lines', 'cat /etc/passwd | cut -d: -f1', wantlist=True) }}"
    
    # ───────────────────────────────────────────────────────
    # 8. URL — HTTP запрос
    # ───────────────────────────────────────────────────────
    github_api: "{{ lookup('url', 'https://api.github.com/users/ansible') }}"
    
    # ───────────────────────────────────────────────────────
    # 9. DNS — DNS lookup
    # ───────────────────────────────────────────────────────
    google_ips: "{{ lookup('dig', 'google.com', wantlist=True) }}"
    mx_records: "{{ lookup('dig', 'gmail.com', 'qtype=MX', wantlist=True) }}"
    
    # ───────────────────────────────────────────────────────
    # 10. INI — чтение INI файлов
    # ───────────────────────────────────────────────────────
    db_host: "{{ lookup('ini', 'host section=database file=/etc/app.ini') }}"
    db_port: "{{ lookup('ini', 'port section=database file=/etc/app.ini') }}"
    
    # ───────────────────────────────────────────────────────
    # 11. CSVFILE — чтение CSV
    # ───────────────────────────────────────────────────────
    user_email: "{{ lookup('csvfile', 'john file=users.csv delimiter=, col=2') }}"
    
    # ───────────────────────────────────────────────────────
    # 12. SEQUENCE — генерация последовательности
    # ───────────────────────────────────────────────────────
    range_10: "{{ lookup('sequence', 'start=1 end=10', wantlist=True) }}"
    # Результат: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    
    # С форматом
    servers: "{{ lookup('sequence', 'start=1 end=5 format=server-%02d', wantlist=True) }}"
    # Результат: ['server-01', 'server-02', 'server-03', 'server-04', 'server-05']
  
  tasks:
    # ───────────────────────────────────────────────────────
    # Использование lookup в tasks
    # ───────────────────────────────────────────────────────
    - name: Display lookups
      debug:
        msg: |
          SSH Key (first 50 chars): {{ ssh_key[:50] }}
          Current User: {{ current_user }}
          Date: {{ current_date }}
          GitHub API (name): {{ (github_api | from_json).name }}
          Google IPs: {{ google_ips }}
    
    # ───────────────────────────────────────────────────────
    # Query (альтернативный синтаксис — всегда список)
    # ───────────────────────────────────────────────────────
    - name: Using query (always returns list)
      debug:
        msg: "{{ item }}"
      loop: "{{ query('fileglob', '/var/log/*.log') }}"
    
    # ───────────────────────────────────────────────────────
    # Практический пример: создание пользователей из CSV
    # ───────────────────────────────────────────────────────
    - name: Create users from CSV
      user:
        name: "{{ item.split(',')[0] }}"
        comment: "{{ item.split(',')[1] }}"
        shell: "{{ item.split(',')[2] }}"
        state: present
      loop: "{{ lookup('file', 'users.csv').splitlines() }}"
      when: not item.startswith('#')  # Пропустить комментарии
```

**Продвинутые lookup:**

YAML

```
# ═══════════════════════════════════════════════════════════
#          ПРОДВИНУТЫЕ LOOKUP ПЛАГИНЫ
# ═══════════════════════════════════════════════════════════

---
- name: Advanced lookups
  hosts: localhost
  
  vars:
    # ───────────────────────────────────────────────────────
    # AWS Secrets Manager
    # ───────────────────────────────────────────────────────
    db_password: >-
      {{
        lookup('amazon.aws.aws_secret',
               'prod/database/password',
               region='us-east-1')
      }}
    
    # ───────────────────────────────────────────────────────
    # HashiCorp Vault
    # ───────────────────────────────────────────────────────
    api_key: >-
      {{
        lookup('community.hashi_vault.hashi_vault_read',
               'secret/data/api',
               url='https://vault.example.com')
      }}
    
    # ───────────────────────────────────────────────────────
    # Redis
    # ───────────────────────────────────────────────────────
    cache_value: >-
      {{
        lookup('community.general.redis',
               'my_key',
               host='localhost')
      }}
    
    # ───────────────────────────────────────────────────────
    # MongoDB
    # ───────────────────────────────────────────────────────
    mongo_data: >-
      {{
        lookup('community.mongodb.mongodb',
               'mydb mycollection',
               connection_string='mongodb://localhost:27017')
      }}
    
    # ───────────────────────────────────────────────────────
    # Kubernetes ConfigMap
    # ───────────────────────────────────────────────────────
    k8s_config: >-
      {{
        lookup('kubernetes.core.k8s',
               'ConfigMap',
               namespace='default',
               resource_name='app-config')
      }}
    
    # ───────────────────────────────────────────────────────
    # Комбинирование lookup
    # ───────────────────────────────────────────────────────
    combined: >-
      {{
        lookup('template', 'config.j2')
        | from_yaml
      }}
    
    # ───────────────────────────────────────────────────────
    # Вложенные lookup
    # ───────────────────────────────────────────────────────
    secret_file_content: >-
      {{
        lookup('file',
               lookup('env', 'SECRET_FILE_PATH'))
      }}
  
  tasks:
    - name: Use secrets
      debug:
        msg: "DB Password is set"
      no_log: true  # Не показывать в логах
```

---

## 24. Ускорение выполнения playbook

YAML

```
# ═══════════════════════════════════════════════════════════
#          УСКОРЕНИЕ ANSIBLE PLAYBOOK
# ═══════════════════════════════════════════════════════════

# ───────────────────────────────────────────────────────────
# 1. УВЕЛИЧИТЬ FORKS (параллельные подключения)
# ───────────────────────────────────────────────────────────
# ansible.cfg:
[defaults]
forks = 50              # По умолчанию: 5
# Или в команде:
# ansible-playbook -f 50 playbook.yml

# ───────────────────────────────────────────────────────────
# 2. ОТКЛЮЧИТЬ GATHERING FACTS (если не нужны)
# ───────────────────────────────────────────────────────────
- name: Fast playbook
  hosts: all
  gather_facts: no      # Экономия ~3 сек на хост
  
  tasks:
    - name: Quick task
      ping:

# ───────────────────────────────────────────────────────────
# 3. SMART GATHERING (кэширование facts)
# ───────────────────────────────────────────────────────────
# ansible.cfg:
[defaults]
gathering = smart       # smart, explicit, implicit
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 86400  # 24 часа

# ───────────────────────────────────────────────────────────
# 4. SSH PIPELINING (ускорение SSH)
# ───────────────────────────────────────────────────────────
# ansible.cfg:
[ssh_connection]
pipelining = True       # Уменьшает количество SSH соединений
# Требует: sudo без requiretty

# На managed nodes:
# sudo visudo
# Defaults !requiretty  # Или закомментировать Defaults requiretty

# ───────────────────────────────────────────────────────────
# 5. SSH CONTROLMASTER (переиспользование соединений)
# ───────────────────────────────────────────────────────────
# ansible.cfg:
[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
control_path = /tmp/ansible-ssh-%%h-%%p-%%r

# ───────────────────────────────────────────────────────────
# 6. ASYNC TASKS (асинхронное выполнение)
# ───────────────────────────────────────────────────────────
- name: Long running tasks
  hosts: all
  
  tasks:
    # Fire-and-forget (не ждать результата)
    - name: Long backup
      command: /usr/local/bin/backup.sh
      async: 3600            # Максимум 1 час
      poll: 0                # Не ждать (poll=0)
      register: backup_job
    
    # Продолжаем другие задачи...
    - name: Other quick tasks
      debug:
        msg: "Doing other things while backup runs"
    
    # Проверить результат позже
    - name: Wait for backup
      async_status:
        jid: "{{ backup_job.ansible_job_id }}"
      register: job_result
      until: job_result.finished
      retries: 360           # Проверять до 1 часа
      delay: 10              # Каждые 10 секунд

# ───────────────────────────────────────────────────────────
# 7. STRATEGY (стратегия выполнения)
# ───────────────────────────────────────────────────────────
- name: Fast strategy
  hosts: all
  strategy: free          # Не ждать всех хостов на каждой стадии
  # linear (default) — ждать всех хостов
  # free — каждый хост идёт своим темпом
  # debug — отладочный режим
  
  tasks:
    - name: Task 1
      command: sleep 5
    - name: Task 2
      command: sleep 10

# ───────────────────────────────────────────────────────────
# 8. MITOGEN (альтернатива SSH)
# ───────────────────────────────────────────────────────────
# Ускорение в 1.25-7x!
# Установка:
# pip install mitogen

# ansible.cfg:
[defaults]
strategy_plugins = /path/to/mitogen/ansible_mitogen/plugins/strategy
strategy = mitogen_linear  # Или mitogen_free

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s

# ───────────────────────────────────────────────────────────
# 9. МИНИМИЗАЦИЯ ЗАДАЧ
# ───────────────────────────────────────────────────────────
# ПЛОХО: много tasks
- name: Install packages (slow)
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - postgresql
    - redis
  # 3 подключения к apt

# ХОРОШО: одна задача
- name: Install packages (fast)
  apt:
    name:
      - nginx
      - postgresql
      - redis
    state: present
  # 1 подключение к apt

# ───────────────────────────────────────────────────────────
# 10. TAGS (запуск только нужных частей)
# ───────────────────────────────────────────────────────────
- name: Full deployment
  hosts: all
  
  tasks:
    - name: Install packages
      apt:
        name: nginx
      tags: install
    
    - name: Configure
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      tags: config
    
    - name: Deploy app
      copy:
        src: app.tar.gz
        dest: /var/www/
      tags: deploy

# Запустить только config:
# ansible-playbook playbook.yml --tags config

# Пропустить install:
# ansible-playbook playbook.yml --skip-tags install

# ───────────────────────────────────────────────────────────
# 11. CHECK MODE (dry-run) — сначала проверить
# ───────────────────────────────────────────────────────────
# ansible-playbook playbook.yml --check --diff
# Показывает что БУДЕТ изменено, не применяя

# ───────────────────────────────────────────────────────────
# 12. LIMIT (запуск на подмножестве хостов)
# ───────────────────────────────────────────────────────────
# ansible-playbook playbook.yml --limit web1,web2
# ansible-playbook playbook.yml --limit 'webservers:!web3'

# ───────────────────────────────────────────────────────────
# 13. ПРОФИЛИРОВАНИЕ (найти узкие места)
# ───────────────────────────────────────────────────────────
# ansible.cfg:
[defaults]
callback_whitelist = profile_tasks, timer
# Покажет время выполнения каждой задачи

# ───────────────────────────────────────────────────────────
# 14. FREE STRATEGY с SERIAL (контролируемая параллельность)
# ───────────────────────────────────────────────────────────
- name: Rolling update
  hosts: webservers
  serial: 5               # По 5 хостов одновременно
  max_fail_percentage: 20 # Остановить если > 20% failed
  
  tasks:
    - name: Update
      apt:
        name: nginx
        state: latest
    - name: Restart
      service:
        name: nginx
        state: restarted

# Или батчами:
  serial:
    - 1      # Сначала 1 хост (canary)
    - 10%    # Затем 10% хостов
    - 100%   # Затем все остальные
```

**Итоговая таблица ускорения:**

text

```
┌──────────────────────┬────────────┬──────────────────────┐
│ Метод                │ Ускорение  │ Применимость         │
├──────────────────────┼────────────┼──────────────────────┤
│ forks = 50           │ 5-10x      │ Всегда               │
│ gather_facts: no     │ 1.2-1.5x   │ Если не нужны facts  │
│ fact_caching         │ 1.3x       │ Частые запуски       │
│ pipelining           │ 1.3-2x     │ Много tasks          │
│ strategy: free       │ 1.5-3x     │ Независимые хосты    │
│ mitogen              │ 1.25-7x    │ Много SSH соединений │
│ async tasks          │ Зависит    │ Долгие задачи        │
│ Оптимизация tasks    │ 2-5x       │ Много apt/yum        │
│ tags                 │ 10-100x    │ Частичные обновления │
└──────────────────────┴────────────┴──────────────────────┘
```

**Пример конфигурации для максимальной скорости:**

ini

```
# ansible.cfg
[defaults]
forks = 100
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 86400
callback_whitelist = profile_tasks, timer
host_key_checking = False

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -o PreferredAuthentications=publickey
pipelining = True
control_path = /tmp/ansible-ssh-%%h-%%p-%%r
```