```
# Подключение
psql -U username -d database          # Подключиться к БД
psql -h localhost -p 5432 -U postgres # С указанием хоста и порта
psql "postgresql://user:pass@host:5432/dbname"  # Через URL

# Команды внутри psql
\l                                    # Список баз данных
\c dbname                             # Переключиться на БД
\dt                                   # Список таблиц
\dt+                                  # С размерами
\d tablename                          # Описание таблицы
\du                                   # Список пользователей
\dn                                   # Список схем
\df                                   # Список функций
\x                                    # Expanded display (вкл/выкл)
\timing                               # Включить отображение времени
\q                                    # Выйти

# Выполнение SQL
SELECT * FROM users;
\i /path/to/script.sql                # Выполнить SQL-файл
\o /path/to/output.txt                # Перенаправить вывод в файл

# Экспорт/Импорт
\copy (SELECT * FROM users) TO '/tmp/users.csv' CSV HEADER
\copy users FROM '/tmp/users.csv' CSV HEADER

# Полезные запросы
SELECT version();                     # Версия PostgreSQL
SELECT current_database();            # Текущая БД
SHOW all;                             # Все настройки
SELECT pg_size_pretty(pg_database_size('dbname'));  # Размер БД
```