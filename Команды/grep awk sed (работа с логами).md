```
# grep
grep "ERROR" /var/log/app.log         # Найти строки с ERROR
grep -i "error" file                  # Регистронезависимый поиск
grep -v "INFO" file                   # Исключить строки с INFO
grep -A 5 "ERROR" file                # 5 строк ПОСЛЕ совпадения
grep -B 5 "ERROR" file                # 5 строк ДО совпадения
grep -C 5 "ERROR" file                # 5 строк ДО и ПОСЛЕ
grep -r "pattern" /var/log/           # Рекурсивный поиск

# awk
awk '{print $1}' file                 # Вывести первую колонку
awk '{print $1, $3}' file             # 1-я и 3-я колонки
awk '/ERROR/ {print $0}' file         # Строки с ERROR
awk -F: '{print $1}' /etc/passwd      # Разделитель — двоеточие

# sed
sed 's/old/new/' file                 # Заменить первое вхождение в строке
sed 's/old/new/g' file                # Заменить все вхождения
sed -i 's/old/new/g' file             # Изменить файл на месте
sed -n '10,20p' file                  # Вывести строки 10-20
```