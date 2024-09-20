
<details><summary><h2>Routine Maintenance</h2></summary>

  [3](https://github.com/AV-ghub/PostgreSQL/blob/main/998%20Books/List.md).[160]

  ### Transaction visibility with multiversion concurrency control
  #### Visibility computation internals
  The **essence of MVCC**: each database client session is allowed to make changes to a table, but it doesn't become visible to other sessions until the transaction
commits. 
  #### [Transaction ID wraparound](https://postgrespro.ru/docs/postgresql/16/routine-vacuuming#VACUUM-FOR-WRAPAROUND)
  The implementation of MVCC in PostgreSQL uses a transaction ID that is 32 bits in size.   
  A signed 32-bit number can only handle a range of about **2 billion transactions** before rolling over to zero.  
  
  The way that the 32-bit XID is mapped to handle many billions of transactions is that **each table and database has a reference XID**, **and every other XID is relative to it**.   
  This gives an effective range of 2 billion transactions before and after that value.   
  You can see how old these reference XID numbers are relative to current activity, starting with the oldest active entries, like this:
  ```
  SELECT relname,age(relfrozenxid) FROM pg_class WHERE relkind='r'
  ORDER BY age(relfrozenxid) DESC;
  SELECT datname,age(datfrozenxid) FROM pg_database ORDER BY
  age(datfrozenxid) DESC;
  ```
  ```
  SELECT c.oid::regclass as table_name,
         greatest(age(c.relfrozenxid),age(t.relfrozenxid)) as age
  FROM pg_class c
  LEFT JOIN pg_class t ON c.reltoastrelid = t.oid
  WHERE c.relkind IN ('r', 'm');
  
  SELECT datname, age(datfrozenxid) FROM pg_database;
  ```
  
  #### [Регламентная очистка](https://postgrespro.ru/docs/postgresql/16/runtime-config-autovacuum#RUNTIME-CONFIG-AUTOVACUUM)

  One of the things VACUUM does is **push forward the frozen value** once a threshold of transactions have passed, set by the autovacuum setting as [autovacuum_freeze_max_age](https://postgrespro.ru/docs/postgresql/16/runtime-config-autovacuum#GUC-AUTOVACUUM-FREEZE-MAX-AGE). This maintenance is also critical to cleaning up the commit log information stored in the pg_xact directory.  
  Some transactions **will fall off** the back here, if they have a **transaction ID so old** that it can't be represented relative to the new reference values.   
  These will have their XID replaced by a special magic value called the FrozenXID. Once that happens, those transactions will appear in the past relative to all active transactions.
  
  The values for these parameters are set very conservatively by default--things start to be frozen after only **200 million transactions**, even though wraparound isn't a concern until **2 billion**.   
  One reason for this is to keep the commit log disk space from growing excessively.  
  At the default value, it should never take up more than **50 MB**, while increasing the free age to its maximum (2 billion) will instead use up **to 500 MB** of space.   
  If you have large tables where that disk usage is trivial and you don't need to run vacuum regularly in order to reclaim space, **increasing the maximum free age parameters can be helpful** to keep autovacuum from doing more work than it has to in freezing your tables.

  Если по какой-либо причине автоочистка не может вычистить старые значения XID из таблицы, система начинает выдавать предупреждающие сообщения, когда самое старое значение XID в базе данных оказывается в **сорока миллионах транзакций** от точки зацикливания.   
  Если эти предупреждения игнорировать, система отключится и не будет начинать никаких транзакций, как только до точки зацикливания останется **менее трёх миллионов транзакций**.   
  В этом состоянии любые уже выполняемые транзакции могут продолжаться, но могут быть запущены лишь транзакции только для чтения.   
  Команду VACUUM по-прежнему можно запускать в обычном режиме.   
  
  Выполните следующие действия:
  1. Разберите старые подготовленные транзакции. Их можно найти, проверив **pg_prepared_xacts** на наличие строк с большим значением age(transactionid). Такие транзакции следует фиксировать или отменять.
  2. Завершите длительные открытые транзакции. Их можно найти, проверив **pg_stat_activity** на наличие строк с большим значением age(backend_xid) или age(backend_xmin). Такие транзакции следует фиксировать или отменять, либо можно **завершить сеанс с помощью pg_terminate_backend**.
  3. Удалите все старые слоты репликации. Используйте **pg_stat_replication**, чтобы найти слоты с большим значением age(xmin) или age(catalog_xmin). Во многих случаях такие слоты создавались для репликации на серверы, которых уже нет или которые давно не работают. Если удалить слот для сервера, который всё ещё существует и может по-прежнему пытаться подключиться к этому слоту, возможно, эту реплику придётся пересоздать.
  4. Выполните **VACUUM** в целевой базе данных. Проще всего использовать VACUUM для всей базы данных. Чтобы сократить время выполнения, также можно вручную выполнить команды VACUUM для таблиц с самым старым relminxid. Не используйте VACUUM FULL в этом сценарии, поскольку для него требуется XID и, следовательно, произойдёт сбой, за исключением режима суперпользователя, где напротив будет обрабатываться XID и, таким образом, увеличится риск зацикливания счётчика идентификатора транзакции. Не используйте VACUUM FREEZE, поскольку при этом выполнится объём работы, который будет больше минимально необходимого для восстановления нормального функционирования.
  5. После восстановления нормальной работы проверьте, что **автоочистка** правильно настроена в целевой базе данных, чтобы избежать проблем в будущем.

  В PostgreSQL имеется не обязательная, но настоятельно рекомендуемая к использованию функция, называемая [автоочисткой](https://postgrespro.ru/docs/postgresql/16/routine-vacuuming#AUTOVACUUM), предназначение которой — автоматизировать выполнение команд VACUUM и ANALYZE.   
  Автоочистка будет работать, только если параметр [track_counts](https://postgrespro.ru/docs/postgresql/16/runtime-config-statistics#GUC-TRACK-COUNTS) имеет значение true.   
  Этот контролирующий процесс распределяет работу по времени, стараясь запускать рабочий процесс для каждой базы данных каждые [autovacuum_naptime](https://postgrespro.ru/docs/postgresql/16/runtime-config-autovacuum#GUC-AUTOVACUUM-NAPTIME) секунд.   
  > Если всего имеется N баз данных, новый рабочий процесс будет запускаться каждые **autovacuum_naptime/N** секунд.

  Одновременно могут выполняться до [autovacuum_max_workers](https://postgrespro.ru/docs/postgresql/16/runtime-config-autovacuum#GUC-AUTOVACUUM-MAX-WORKERS) рабочих процессов.   
  Для отслеживания действий рабочих процессов можно установить параметр [log_autovacuum_min_duration](https://postgrespro.ru/docs/postgresql/16/runtime-config-logging#GUC-LOG-AUTOVACUUM-MIN-DURATION).    
  Число рабочих процессов для одной базы не ограничивается, при этом каждый процесс старается не повторять работу, только что выполненную другими.   
  Заметьте, что в ограничениях [max_connections](https://postgrespro.ru/docs/postgresql/16/runtime-config-connection#GUC-MAX-CONNECTIONS) или [superuser_reserved_connections](https://postgrespro.ru/docs/postgresql/16/runtime-config-connection#GUC-SUPERUSER-RESERVED-CONNECTIONS) число выполняющихся рабочих процессов не учитывается.   
  Базовый порог очистки при добавлении и коэффициент доли для очистки при добавлении определяются параметрами [autovacuum_vacuum_insert_threshold](https://postgrespro.ru/docs/postgresql/16/runtime-config-autovacuum#GUC-AUTOVACUUM-VACUUM-INSERT-THRESHOLD) и [autovacuum_vacuum_insert_scale_factor](https://postgrespro.ru/docs/postgresql/16/runtime-config-autovacuum#GUC-AUTOVACUUM-VACUUM-INSERT-SCALE-FACTOR), соответственно.  
  > Для таблиц, в которых выполняются в основном операции INSERT и практически не выполняются UPDATE/DELETE, может иметь смысл уменьшить параметр таблицы [autovacuum_freeze_min_age](https://postgrespro.ru/docs/postgresql/16/sql-createtable#RELOPTION-AUTOVACUUM-FREEZE-MIN-AGE), так как это позволит замораживать кортежи раньше.

  > В **секционированных таблицах** кортежи не хранятся напрямую и, следовательно, **не обрабатываются автоочисткой**. (Автоочистка обрабатывает секции таблицы так же, как и другие таблицы.) К сожалению, это означает, что **автоочистка не запускает ANALYZE для секционированных таблиц**, в результате чего **создаются неоптимальные планы** для запросов, ссылающихся на статистику секционированных таблиц. Эту проблему можно обойти, **вручную запуская ANALYZE для секционированных таблиц при их первом заполнении, а также всякий раз, когда распределение данных в их секциях существенно меняется**.

   **Автоочистка не обрабатывает временные таблицы**. Поэтому очистку и сбор статистики в них нужно производить с помощью SQL-команд в обычном сеансе.

  Используемые по умолчанию пороговые значения и коэффициенты берутся из postgresql.conf, однако их (и многие другие параметры, управляющие автоочисткой) можно переопределить для каждой таблицы; за подробностями обратитесь к разделу [Параметры хранения](https://postgrespro.ru/docs/postgresql/16/sql-createtable#SQL-CREATETABLE-STORAGE-PARAMETERS).   
  Если какие-либо значения определены **через параметры хранения таблицы**, при обработке этой таблицы **действуют они**, а в противном случае — глобальные параметры.

  Когда выполняются несколько рабочих процессов, [параметры задержки автоочистки по стоимости](https://postgrespro.ru/docs/postgresql/16/runtime-config-resource#RUNTIME-CONFIG-RESOURCE-VACUUM-COST) «распределяются» между всеми этими процессами, так что общее воздействие на систему остаётся неизменным, независимо от их числа. Однако этот алгоритм распределения нагрузки **не учитывает процессы**, обрабатывающие таблицы **с индивидуальными значениями параметров хранения autovacuum_vacuum_cost_delay и autovacuum_vacuum_cost_limit**.

  Рабочие процессы автоочистки обычно не мешают выполнению других команд. Если какой-либо **процесс попытается получить блокировку**, конфликтующую с блокировкой SHARE UPDATE EXCLUSIVE, которая удерживается в ходе автоочистки, **автоочистка прервётся и процесс получит нужную ему блокировку**. Однако если автоочистка выполняется для предотвращения зацикливания идентификаторов транзакций (т. е. описание запроса автоочистки в представлении pg_stat_activity заканчивается на (to prevent wraparound)), автоочистка не прерывается без ручного вмешательства.

  > При частом выполнении таких команд, как **ANALYZE**, которые затребуют блокировки, конфликтующие с SHARE UPDATE EXCLUSIVE, может получиться так, что **автоочистка не будет успевать завершаться** в принципе.
 
  #### [System Information Functions and Operators](https://www.postgresql.org/docs/current/functions-info.html)

  ### Vacuum
  
  **Cleaning up** after all these situations that produce dead rows (UPDATE, DELETE, ROLLBACK) is the job for an operation named **vacuuming**.  
  Each database **row** includes **status flags** called hint bits that track whether the transaction that updated the xmin or xmax values is known to be committed or aborted.  
  > The actual commit logs (**pg_xact** and sometimes pg_subtrans) are **consulted** to confirm the hint bits' transaction states.

  Vacuum does a scan of each table and index, looking for rows that can **no longer be visible**.   
  Once the **free space map** for a table has entries on it, **new allocations** for this table will **reuse that existing space** when possible, **rather than allocating new** space from the operating system.   
  In most situations, vacuum will **never release disk space**.   
  A **large deletion** of historical data is one way to **end up with** a table with **lots of free space at its beginning**.   

  PostgreSQL 9.0 has introduced a rewritten VACUUM FULL command that is modeled on the [CLUSTER](https://postgrespro.ru/docs/postgresql/16/sql-cluster) implementation of earlier versions.   
  
  Sometimes the transaction takes too long and holds the tuples for a very long time.   
  Configuration parameter [old_snapshot_threshold](https://postgrespro.ru/docs/postgresql/16/runtime-config-resource#GUC-OLD-SNAPSHOT-THRESHOLD) can be configured to specify **how long this snapshot is valid** for.   
  After that time, the dead tuple is a candidate for deletion, and if the transaction uses that tuple, it **gets an error**.

  #### HOT
  One of the major performance features added to PostgreSQL 8.3 is HOT (**Heap Only Tuples**).   
  HOT allows the reuse of space left behind by dead rows resulting from the DELETE or UPDATE operations under some common conditions.   
  The specific case that HOT helps with is when you are making **changes to a row that does not update any of its indexed columns**.

  The normal way to check if you are getting the benefit of HOT updates is to monitor [pg_stat_user_tables](https://postgrespro.ru/docs/postgresql/16/monitoring-stats) ([pg_stat_all_tables](https://postgrespro.ru/docs/postgresql/16/monitoring-stats#MONITORING-PG-STAT-ALL-TABLES-VIEW)) and compare the counts for **n_tup_upd** (regular updates) versus **n_tup_hot_upd**.

  One of the ways to **make HOT more effective** on your tables is to **use a larger fill factor** setting when creating them.   

  #### Cost-based vacuuming
  A manual vacuum worker will execute until it has **exceeded vacuum_cost_limit** of the estimated I/O, **defaulting to 200 units** of work.   
  At that point, it will then **sleep for vacuum_cost_delay milliseconds**, defaulting to 0--**which disables the cost delay feature** altogether with manually executed VACUUM statements.   
  Autovacuum workers have their own parameters that work the same way.   
  **autovacuum_vacuum_cost_limit** defaults to -1, which is shorthand for saying that they use the same cost limit structure as manual vacuum.   
  The main way that autovacuum diverges from a manual one is it defaults to the following cost delay:
  ```
  autovacuum_vacuum_cost_delay = 20ms
  ```

  If you want to **adjust a manual VACUUM** to run with the cost logic, you can tweak it before issuing any manual vacuum and it will effectively limit its impact for **just that session**:
  ```
  postgres=# SET vacuum_cost_delay='20';
  postgres=# show vacuum_cost_delay;
  vacuum_cost_delay
  -------------------
  20ms
  ```

  ### Autovacuum
  #### Autovacuum logging
  It's possible to watch it more directly by setting [log_min_messages](https://postgrespro.ru/docs/postgresql/16/runtime-config-logging#GUC-LOG-MIN-MESSAGES):
  ```
  log_min_messages =debug2
  ```
  You can monitor the daemon's activity by setting [log_autovacuum_min_duration](https://postgrespro.ru/docs/postgresql/16/runtime-config-logging#GUC-LOG-AUTOVACUUM-MIN-DURATION) to some number of milliseconds.   
  It defaults to -1, turning logging off. 
  Setting this to a moderate number of milliseconds (for example 1000=1 second) is a good practice to follow.   

  The best way to approach making sure autovacuum is doing what it should is to monitor what tables it's worked on instead:
  ```
  SELECT schemaname,relname,last_autovacuum,last_autoanalyze
  FROM pg_stat_all_tables;
  ```
  #### [Tuning Autovacuum in PostgreSQL and Autovacuum Internals](https://www.percona.com/blog/tuning-autovacuum-in-postgresql-and-autovacuum-internals/)
  Autovacuum is one of the background utility processes that starts automatically when you start PostgreSQL
  ```
  $ps -eaf | egrep "/post|autovacuum"
  ```
  We also need ANALYZE on the table that updates the table statistics, so that the optimizer can choose optimal execution plans for an SQL statement.    
  It is the **autovacuum** in postgres that is responsible for performing **both vacuum and analyze** on tables.   
  There exists another background process in postgres called **Stats Collector** that tracks the usage and activity information.

  Parameters needed to enable autovacuum in PostgreSQL are :
  ``` shell
  autovacuum = on  # ( ON by default )
  track_counts = on # ( ON by default )
  ```
  track_counts  is used by the stats collector. 
  
  ##### Logging autovacuum
  Set the parameter log_autovacuum_min_duration
  ```
  # Setting this parameter to 0 logs every autovacuum to the log file.
  log_autovacuum_min_duration = '250ms' # Or 1s, 1min, 1h, 1d
  ```
  The formula for calculating the effective table level autovacuum threshold is :
  ```Shell
  Autovacuum VACUUM thresold for a table = autovacuum_vacuum_scale_factor * number of tuples + autovacuum_vacuum_threshold
  ```
  * autovacuum_vacuum_scale_factor Or autovacuum_analyze_scale_factor : Fraction of the table records that will be added to the formula. For example, a value of 0.2 equals to 20% of the table records.
  * autovacuum_vacuum_threshold Or autovacuum_analyze_threshold : Minimum number of obsolete records or dml’s needed to trigger an autovacuum.
  
  PostgreSQL allows you to configure individual table level autovacuum settings that bypass global settings.
  ```
  $psql -d percona
   
  percona=# ALTER TABLE scott.employee SET (autovacuum_vacuum_scale_factor = 0, autovacuum_vacuum_threshold = 100);
  ALTER TABLE
  ```
  ##### How do we identify the tables that need their autovacuum settings tuned?
  You must know the number of inserts/deletes/updates on a table for an interval.   
  You can also view the postgres catalog view : pg_stat_user_tables to get that information.
  ```  
  percona=# SELECT n_tup_ins as "inserts",n_tup_upd as "updates",n_tup_del as "deletes", n_live_tup as "live_tuples", n_dead_tup as "dead_tuples"
  FROM pg_stat_user_tables
  WHERE schemaname = 'scott' and relname = 'employee';
   inserts | updates | deletes | live_tuples | dead_tuples 
  ---------+---------+---------+-------------+-------------
        30 |      40 |       9 |          21 |          39
  ```
  > Does increasing autovacuum_max_workers alone increase the number of autovacuum processes that can run in parallel?    
  > **NO** 

  

</details>






























