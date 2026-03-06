```
whoami                                # Текущий пользователь
id                                    # UID, GID, группы
useradd username                      # Создать пользователя
userdel username                      # Удалить пользователя
passwd username                       # Изменить пароль
su - username                         # Переключиться на пользователя
sudo command                          # Выполнить от root

# Группы
groups                                # Группы текущего пользователя
groupadd groupname                    # Создать группу
usermod -aG docker username           # Добавить пользователя в группу
```