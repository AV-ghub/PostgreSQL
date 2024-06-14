Включить в template0 нужные extension, чтобы не создавать их в каждой БД.   
<details><summary><h5>Включить pg_stat_statements (просадка производительности до 15%)</h5></summary>

Чтобы добавить pg_stat_statements, установите сначала пакет ***postgresql-contrib***.   
Чтобы загрузить расширение pg_stat_statements, нужно изменить файл конфигурации ***postgresql.conf*** для сервера PostgreSQL.
Откройте файл postgresql.conf в текстовом редакторе и измените строку shared_preload_libraries:   
```bash
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.track_utility = false
```
Эти изменения необходимы для мониторинга операторов SQL, кроме команд утилиты.   
> Состояние pg_stat_statements.track_utility назначает или изменяет только суперпользователь.

После обновления и сохранения postgresql.conf ***перезапустите сервер PostgreSQL***.   
Введите следующую команду SQL, используя psql, который должен быть связан с той же базой данных, которая будет позже указана в конфигурации агента, чтобы обеспечить возможность соединения JDBC:
```sql
create extension pg_stat_statements; 
select pg_stat_statements_reset();
```
> Запустить команду create extension и функцию pg_stat_statements_reset() может только суперпользователь.

Представление ***pg_stat_statements нужно включить для определенной базы данных***    
[Более подробно](https://www.postgresql.org/docs/9.6/static/pgstatstatements.html)

</details>

