```
# Ad-hoc команды
ansible all -m ping                   # Проверить доступность хостов
ansible all -m shell -a "uptime"      # Выполнить команду
ansible webservers -m apt -a "name=nginx state=present" --become

# Playbooks
ansible-playbook playbook.yml         # Запустить playbook
ansible-playbook playbook.yml --check # Dry-run (проверка)
ansible-playbook playbook.yml --diff  # Показать изменения
ansible-playbook playbook.yml --limit webservers  # Только конкретная группа
ansible-playbook playbook.yml --tags "deploy"     # Только таски с тегом
ansible-playbook playbook.yml --skip-tags "slow"  # Пропустить тег
ansible-playbook playbook.yml -e "version=1.2.3"  # Передать переменную
ansible-playbook playbook.yml -vvv    # Verbose (детальный вывод)

# Inventory
ansible-inventory --list              # Показать инвентарь
ansible-inventory --graph             # Древовидный вывод
ansible all --list-hosts              # Список хостов

# Vault (шифрование секретов)
ansible-vault create secrets.yml      # Создать зашифрованный файл
ansible-vault edit secrets.yml        # Редактировать
ansible-vault encrypt file.yml        # Зашифровать существующий файл
ansible-vault decrypt file.yml        # Расшифровать
ansible-playbook playbook.yml --ask-vault-pass  # Запросить пароль vault

# Galaxy (установка ролей)
ansible-galaxy install geerlingguy.nginx
ansible-galaxy install -r requirements.yml
```