```
# Бэкап
pg_dump dbname > backup.sql           # SQL-дамп
pg_dump -Fc dbname > backup.dump      # Custom формат (сжатый)
pg_dump -t tablename dbname > table.sql  # Только одна таблица
pg_dumpall > all_databases.sql        # Все БД

# Восстановление
psql dbname < backup.sql              # Из SQL
pg_restore -d dbname backup.dump      # Из custom формата
pg_restore -Fc -d dbname backup.dump  # С очисткой БД перед импортом
```