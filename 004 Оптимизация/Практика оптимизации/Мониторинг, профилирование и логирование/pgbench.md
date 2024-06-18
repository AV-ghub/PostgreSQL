<details><summary><h5>Инициализация</h5></summary>

Для запускаемого по умолчанию теста типа TPC-B требуется предварительно подготовить определённые таблицы.     
Чтобы создать и наполнить эти таблицы, следует запустить 
```sql
pgbench -i dbname
```
> Чтобы указать, как подключиться к серверу баз данных, вы также можете добавить параметры -h, -p и/или -U    

> pgbench -i создаёт четыре таблицы pgbench_accounts, pgbench_branches, pgbench_history и pgbench_tellers, предварительно уничтожая существующие таблицы с этими именами.    

С «коэффициентом масштаба», по умолчанию равным 1, эти таблицы изначально содержат такое количество строк:
```bash
table                   # of rows
---------------------------------
pgbench_branches        1
pgbench_tellers         10
pgbench_accounts        100000
pgbench_history         0
```
<details><summary>Скрипт</summary>
  
```sql
SELECT 
  pg_class.relname AS table_name,
  pg_size_pretty(pg_total_relation_size(pg_class.oid)) AS size,
  pg_total_relation_size(pg_class.oid) / (current_setting('block_size')::integer / 1024) AS num_blocks,
  pg_stat_user_tables.n_live_tup AS num_rows
FROM 
  pg_stat_user_tables 
JOIN 
  pg_class ON pg_stat_user_tables.relid = pg_class.oid 
WHERE 
  pg_class.relnamespace = (SELECT oid FROM pg_namespace WHERE nspname='public')
ORDER BY 
  pg_total_relation_size(pg_class.oid) DESC;
```

</details>

Эти числа можно (и в большинстве случаев даже нужно) увеличить, воспользовавшись параметром -s (коэффициент масштаба).    
При этом также может быть полезен ключ -F (фактор заполнения).

</details>
<details><summary><h5>Использование</h5></summary>

Подготовив требуемую конфигурацию, можно запустить тест производительности командой без -i, то есть:
```sql
pgbench [ параметры ] имя_базы
```

</details>


