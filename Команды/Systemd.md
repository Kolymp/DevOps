```
# Управление сервисами
systemctl start nginx                 # Запустить
systemctl stop nginx                  # Остановить
systemctl restart nginx               # Перезапустить
systemctl reload nginx                # Перезагрузить конфиг
systemctl status nginx                # Статус
systemctl enable nginx                # Автозапуск при загрузке
systemctl disable nginx               # Отключить автозапуск

# Логи
journalctl -u nginx                   # Логи сервиса
journalctl -u nginx -f                # Следить за логами
journalctl --since "1 hour ago"       # Логи за последний час
journalctl -p err                     # Только ошибки
```