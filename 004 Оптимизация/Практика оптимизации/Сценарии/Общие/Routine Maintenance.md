
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

</details>






























