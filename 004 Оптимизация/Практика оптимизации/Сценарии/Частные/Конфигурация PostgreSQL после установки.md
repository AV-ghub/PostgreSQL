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

<details><summary><h5>Основные параметры конфигурации</h5></summary>

[src](https://github.com/aeuge/postgres16book/blob/main/scripts/parameters.md)   
[Пример реальной конфигурации промышленного сервера](https://github.com/AV-ghub/PostgreSQL/blob/main/004%20%D0%9E%D0%BF%D1%82%D0%B8%D0%BC%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F/%D0%9F%D1%80%D0%B0%D0%BA%D1%82%D0%B8%D0%BA%D0%B0%20%D0%BE%D0%BF%D1%82%D0%B8%D0%BC%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D0%B8/%D0%A1%D1%86%D0%B5%D0%BD%D0%B0%D1%80%D0%B8%D0%B8/%D0%A7%D0%B0%D1%81%D1%82%D0%BD%D1%8B%D0%B5/Linux%20HugePages.md#postgresqlconf)   
[для соответствующего железа](https://github.com/AV-ghub/PostgreSQL/blob/main/004%20%D0%9E%D0%BF%D1%82%D0%B8%D0%BC%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F/%D0%9F%D1%80%D0%B0%D0%BA%D1%82%D0%B8%D0%BA%D0%B0%20%D0%BE%D0%BF%D1%82%D0%B8%D0%BC%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D0%B8/%D0%A1%D1%86%D0%B5%D0%BD%D0%B0%D1%80%D0%B8%D0%B8/%D0%A7%D0%B0%D1%81%D1%82%D0%BD%D1%8B%D0%B5/Linux%20HugePages.md#benchmark-machine)

***
## shared_buffers
***
Рекомендуемое значение для данного параметра - 25% от общей оперативной памяти на сервере.     
The value for shared_buffers should never be set to reserve all of the system RAM for PostgreSQL.     
A value over 25% of the system RAM can be useful if, for example, it is set such that the entire database working set of data can fit in cache, as this would greatly reduce the amount of time reading from disk.

Alternately, ***while a larger shared_buffers value can increase performance in 'read heavy'*** use cases, having a large shared_buffer value ***can be detrimental for 'write heavy'*** use cases, as the entire contents of shared_buffers must be processed during writes.
***
## max_connections
***
Максимальное количество соединений. Для изменения данного параметра придётся перезапускать сервер. Если планируется использование PostgreSQL как DWH, то большое количество соединений не нужно. Данный параметр тесно связан с **work_mem**.
***
## effective_cache_size
***
Служит подсказкой для планировщика, сколько ОП у него в запасе. Можно определить как **shared_buffers** + ОП системы - ОП используемое самой ОС и другими приложениями. За счёт данного параметра планировщик может чаще использовать индексы, строить hash таблицы. Наиболее часто используемое значение 75% ОП от общей на сервере.    
A conservative value for  effective_cache_size  would be 1/2 of the total memory available on the system. Most commonly, the value is set to 75% of the total system memory on a dedicated DB server.

> If the value for effective_cache_size  is too low, then the query planner may decide ***not to use some indexes***, even if they would help greatly increase query speed.


***
## work_mem
***
Используется для сортировок, построения hash таблиц. Это позволяет выполнять данные операции в памяти, что гораздо быстрее обращения к диску. В рамках одного запроса данный параметр может быть использован несколько раз. Если ваш запрос содержит 5 операций сортировки, то память, которая может использоваться для его выполнения уже как минимум **work_mem** * 5. Т.к. скорее всего на сервере сессий много, то каждая из них может использовать этот параметр по нескольку раз, поэтому не рекомендуется делать его слишком большим. Можно выставить небольшое значение для глобального параметра в конфиге и потом, в случае сложных запросов, менять этот параметр локально (для текущей сессии). Обратите внимание, что при превышении этого параметра будет использовано временное пространство, расположенное на диске - запросы будут выполняться медленнее и при большом запросе с декартовым произведением могут привести к опустошению места на диске и завершаться с ошибкой, также могут способствовать приходу ООМ киллера в зависимости от конфигурации ОС.    
it's ideal to set the global value for  work_mem at a relatively low value, and then alter any specific queries themselves to use a higher  work_mem  value:
```sql
SET LOCAL work_mem = '256MB';
SELECT * FROM db ORDER BY LOWER(name);
```
***
## maintenance_work_mem
***
Определяет максимальное количество ОП для операций типа VACUUM, CREATE INDEX, CREATE FOREIGN KEY. Увеличение этого параметра позволит быстрее выполнять эти операции. Не связано с **work_mem** поэтому можно ставить в разы больше, чем **work_mem**    
Рекомендуемые параметры: классический подход в диапазоне от 128 МБ до 1 ГБ и больше, если достаточно оперативной памяти и выполнение обслуживания занимает существенную долю времени.   
***
## wal_buffers
***
Объём разделяемой памяти, который будет использоваться для буферизации данных WAL, ещё не записанных на диск. Если у вас большое количество одновременных подключений, увеличение параметра улучшит производительность. По умолчанию -1, определяется автоматически, как 1/32 от **shared_buffers**, но не больше, чем 16 МБ (вручную можно задавать больше). Обычно ставят 16 Мб.   
If the system being tuned has a large number of concurrent connections, then a higher value for wal_buffers can provide better performance.
***
## random_page_cost 
***
Задаёт приблизительную стоимость чтения одной произвольной страницы с диска. Значение по умолчанию равно 4.0. У твердотельных накопителей лучше выбрать меньшее значение random_page_cost, оптимально 1.1.
***
## max_worker_processes / max_parallel_workers_per_gather / max_parallel_maintenance_workers/ max_parallel_workers 
***
Используются для распараллеливания исполнения запросов - устанавливаем в зависимости от количества ядер ВМ. 
https://www.postgresql.org/docs/current/when-can-parallel-query-be-used.html    
https://dataegret.com/2018/04/lets-speed-things-up/   

<details><summary><h6>Пример конфигурации</h6></summary>

For example, for system with 32 CPU cores and SSDs, a good starting point is: 
```bash
max_worker_processes = 12
max_parallel_workers_per_gather = 4
max_parallel_workers = 12
```
These settings allow to run at least 3 parallel queries concurrently with maximum of 4 workers per query, and to have 20 cores for other, non-parallel queries.    
> If using background workers max_worker_processes should be increased accordingly.

![Background workers](https://github.com/AV-ghub/PostgreSQL/blob/main/999%20Resources/Images/background_workers.png)

</details>

#### Рекомендуемые параметры: 
**max_parallel_workers_per_gather** Если сервер имеет достаточное количество памяти и выполняет запросы с большим объёмом данных, 4 или 8.   
**max_parallel_workers** Значение по умолчанию 8. Если сервер имеет много ядер и большой объём оперативной памяти, можно увеличить до 16 или 32.   
**max_parallel_maintenance_workers** Значение по умолчанию 2. Если выполняется много обслуживания, можно увеличить до 4.   

***
## synchronous_commit 
***
Отключаем синхронную запись журнала изменений данных на диск, что позволяет увеличить скорость ответа СУБД от 10% до 3000+ % за счет подтверждения записи в каждой транзакции. Конечно, при сбое ВМ, мы можем потерять небольшую часть последних изменений.
***
### min_wal_size и max_wal_size 
***
Тюнинг параметров min_wal_size и max_wal_size связан с управлением журналом транзакций (Write-Ahead Log - WAL) в системе управления базами данных (СУБД) PostgreSQL. Эти параметры позволяют настроить размеры журнальных сегментов, которые используются для записи изменений в базу данных перед их фиксацией.

min_wal_size: Этот параметр задает минимальный размер журнального сегмента, до которого должен "опуститься" WAL перед переиспользованием. Если установить его слишком низко, может возникнуть увеличение количества записей (высокий I/O), так как PostgreSQL не сможет эффективно переиспользовать журнальные файлы. Рекомендуется установить его на достаточно высокое значение, чтобы уменьшить I/O операции записи, но не слишком высокое, чтобы избежать излишнего потребления места на диске.

max_wal_size: Этот параметр устанавливает максимальный размер журнального сегмента. Если установить его слишком низко, это может привести к тому, что база данных перестанет работать, когда достигнет предела размера журнала, и потребуется архивация WAL для освобождения места. Но если значение установлено слишком высоко, это может привести к тому, что вам понадобится больше места на диске.

Рекомендуется выбирать значения для min_wal_size и max_wal_size таким образом, чтобы обеспечить баланс между эффективностью записи и использованием дискового пространства, а также учитывать конкретные характеристики вашей базы данных и потребности в производительности.
***
## wal_segment_size
***
Этот параметр определяет размер каждого WAL сегмента в мегабайтах.
Изменение размера WAL сегмента требует внимания, поскольку это может повлиять на производительность и использование дискового пространства. Выполняйте эту операцию с осторожностью.
Для изменения необходимо использовать утилиту pg_resetwal.
***
## checkpoint_timeout
***
Чем реже происходит сбрасывание грязных буферов на диск, тем дольше будет восстановление БД после сбоя. Значение по умолчанию 5 минут, рекомендуемое - от 10 минут до часа. 
***
Необходимо "синхронизировать" два этих параметра. Для этого можно поставить **checkpoint_timeout** в выбранный промежуток, включить параметр **log_checkpoints** и по нему отследить, сколько было записано буферов. После чего подогнать параметр **max_wal_size**
***
## effective_io_concurrency
***
Задаёт оценку, сколько параллельных асинхронных запросов может выдержать дисковая подсистема. Современные твердотельные накопители 
эффективно справляются с этой задачей. Можно ставить 100-300. Правда если и ОС поддерживает posix_fadvise.
https://www.opennet.ru/man.shtml?topic=posix_fadvise&category=2&russian=0
***
## old_snapshot_threshold = -1
***
Ни в коем случае НЕ включать! Падение производительности может достигать 10х+

[PostgreSQL Configurator](https://pgconfigurator.cybertec.at/)
<details><summary><h6>Примеры конфигурации</h6></summary>

<details><summary><h6>2Gb/1CPU/1Disk/1GbDB</h6></summary>

```bash
# Connectivity
max_connections = 20
superuser_reserved_connections = 3

# Memory Settings
shared_buffers = '512 MB'
work_mem = '32 MB'
maintenance_work_mem = '320 MB'
huge_pages = off
effective_cache_size = '1 GB'
effective_io_concurrency = 100 # concurrent IO only really activated if OS supports posix_fadvise function
random_page_cost = 1.25 # speed of random disk access relative to sequential access (1.0)

# Monitoring
shared_preload_libraries = 'pg_stat_statements' # per statement resource usage stats
track_io_timing=on # measure exact block IO times
track_functions=pl # track execution times of pl-language procedures if any

# Replication
wal_level = replica # consider using at least 'replica'
max_wal_senders = 0
synchronous_commit = on

# Checkpointing:
checkpoint_timeout = '15 min'
checkpoint_completion_target = 0.9
max_wal_size = '1024 MB'
min_wal_size = '512 MB'


# WAL writing
wal_compression = on
wal_buffers = -1 # auto-tuned by Postgres till maximum of segment size (16MB by default)
wal_writer_delay = 200ms
wal_writer_flush_after = 1MB


# Background writer
bgwriter_delay = 200ms
bgwriter_lru_maxpages = 100
bgwriter_lru_multiplier = 2.0
bgwriter_flush_after = 0

# Parallel queries:
max_worker_processes = 2
max_parallel_workers_per_gather = 1
max_parallel_maintenance_workers = 1
max_parallel_workers = 2
parallel_leader_participation = on

# Advanced features
enable_partitionwise_join = on
enable_partitionwise_aggregate = on
jit = on
max_slot_wal_keep_size = '1000 MB'
track_wal_io_timing = on
maintenance_io_concurrency = 100
wal_recycle = on
```
  
</details>  
<details><summary><h6>2Gb/2CPU/1Disk/1GbDB</h6></summary>

```bash
# Connectivity
max_connections = 20
superuser_reserved_connections = 3

# Memory Settings
shared_buffers = '512 MB'
work_mem = '32 MB'
maintenance_work_mem = '320 MB'
huge_pages = off
effective_cache_size = '1 GB'
effective_io_concurrency = 100 # concurrent IO only really activated if OS supports posix_fadvise function
random_page_cost = 1.25 # speed of random disk access relative to sequential access (1.0)

# Monitoring
shared_preload_libraries = 'pg_stat_statements' # per statement resource usage stats
track_io_timing=on # measure exact block IO times
track_functions=pl # track execution times of pl-language procedures if any

# Replication
wal_level = replica # consider using at least 'replica'
max_wal_senders = 0
synchronous_commit = on

# Checkpointing:
checkpoint_timeout = '15 min'
checkpoint_completion_target = 0.9
max_wal_size = '1024 MB'
min_wal_size = '512 MB'


# WAL writing
wal_compression = on
wal_buffers = -1 # auto-tuned by Postgres till maximum of segment size (16MB by default)
wal_writer_delay = 200ms
wal_writer_flush_after = 1MB


# Background writer
bgwriter_delay = 200ms
bgwriter_lru_maxpages = 100
bgwriter_lru_multiplier = 2.0
bgwriter_flush_after = 0

# Parallel queries:
max_worker_processes = 2
max_parallel_workers_per_gather = 1
max_parallel_maintenance_workers = 1
max_parallel_workers = 2
parallel_leader_participation = on

# Advanced features
enable_partitionwise_join = on
enable_partitionwise_aggregate = on
jit = on
max_slot_wal_keep_size = '1000 MB'
track_wal_io_timing = on
maintenance_io_concurrency = 100
wal_recycle = on
```
  
</details>  
<details><summary><h6>4Gb/2CPU/1Disk/1GbDB</h6></summary>

```bash
# Connectivity
max_connections = 20
superuser_reserved_connections = 3

# Memory Settings
shared_buffers = '1024 MB'
work_mem = '32 MB'
maintenance_work_mem = '320 MB'
huge_pages = off
effective_cache_size = '3 GB'
effective_io_concurrency = 100 # concurrent IO only really activated if OS supports posix_fadvise function
random_page_cost = 1.25 # speed of random disk access relative to sequential access (1.0)

# Monitoring
shared_preload_libraries = 'pg_stat_statements' # per statement resource usage stats
track_io_timing=on # measure exact block IO times
track_functions=pl # track execution times of pl-language procedures if any

# Replication
wal_level = replica # consider using at least 'replica'
max_wal_senders = 0
synchronous_commit = on

# Checkpointing:
checkpoint_timeout = '15 min'
checkpoint_completion_target = 0.9
max_wal_size = '1024 MB'
min_wal_size = '512 MB'


# WAL writing
wal_compression = on
wal_buffers = -1 # auto-tuned by Postgres till maximum of segment size (16MB by default)
wal_writer_delay = 200ms
wal_writer_flush_after = 1MB


# Background writer
bgwriter_delay = 200ms
bgwriter_lru_maxpages = 100
bgwriter_lru_multiplier = 2.0
bgwriter_flush_after = 0

# Parallel queries:
max_worker_processes = 2
max_parallel_workers_per_gather = 1
max_parallel_maintenance_workers = 1
max_parallel_workers = 2
parallel_leader_participation = on

# Advanced features
enable_partitionwise_join = on
enable_partitionwise_aggregate = on
jit = on
max_slot_wal_keep_size = '1000 MB'
track_wal_io_timing = on
maintenance_io_concurrency = 100
wal_recycle = on
```
  
</details>  
<details><summary><h6>16Gb/4CPU/2Disk/10GbDB</h6></summary>

```bash
# Connectivity
max_connections = 100
superuser_reserved_connections = 3

# Memory Settings
shared_buffers = '4096 MB'
work_mem = '32 MB'
maintenance_work_mem = '320 MB'
huge_pages = off
effective_cache_size = '11 GB'
effective_io_concurrency = 100 # concurrent IO only really activated if OS supports posix_fadvise function
random_page_cost = 1.25 # speed of random disk access relative to sequential access (1.0)

# Monitoring
shared_preload_libraries = 'pg_stat_statements' # per statement resource usage stats
track_io_timing=on # measure exact block IO times
track_functions=pl # track execution times of pl-language procedures if any

# Replication
wal_level = replica # consider using at least 'replica'
max_wal_senders = 0
synchronous_commit = on

# Checkpointing:
checkpoint_timeout = '15 min'
checkpoint_completion_target = 0.9
max_wal_size = '1024 MB'
min_wal_size = '512 MB'


# WAL writing
wal_compression = on
wal_buffers = -1 # auto-tuned by Postgres till maximum of segment size (16MB by default)
wal_writer_delay = 200ms
wal_writer_flush_after = 1MB


# Background writer
bgwriter_delay = 200ms
bgwriter_lru_maxpages = 100
bgwriter_lru_multiplier = 2.0
bgwriter_flush_after = 0

# Parallel queries:
max_worker_processes = 4
max_parallel_workers_per_gather = 2
max_parallel_maintenance_workers = 2
max_parallel_workers = 4
parallel_leader_participation = on

# Advanced features
enable_partitionwise_join = on
enable_partitionwise_aggregate = on
jit = on
max_slot_wal_keep_size = '1000 MB'
track_wal_io_timing = on
maintenance_io_concurrency = 100
wal_recycle = on
```
  
</details>  
<details><summary><h6>32Gb/4CPU/2Disk/100GbDB</h6></summary>

```bash
# Connectivity
max_connections = 100
superuser_reserved_connections = 3

# Memory Settings
shared_buffers = '8192 MB'
work_mem = '32 MB'
maintenance_work_mem = '420 MB'
huge_pages = try # NB! requires also activation of huge pages via kernel params, see here for more: https://www.postgresql.org/docs/current/static/kernel-resources.html#LINUX-HUGE-PAGES
effective_cache_size = '22 GB'
effective_io_concurrency = 100 # concurrent IO only really activated if OS supports posix_fadvise function
random_page_cost = 1.25 # speed of random disk access relative to sequential access (1.0)

# Monitoring
shared_preload_libraries = 'pg_stat_statements' # per statement resource usage stats
track_io_timing=on # measure exact block IO times
track_functions=pl # track execution times of pl-language procedures if any

# Replication
wal_level = replica # consider using at least 'replica'
max_wal_senders = 0
synchronous_commit = on

# Checkpointing:
checkpoint_timeout = '15 min'
checkpoint_completion_target = 0.9
max_wal_size = '1024 MB'
min_wal_size = '512 MB'


# WAL writing
wal_compression = on
wal_buffers = -1 # auto-tuned by Postgres till maximum of segment size (16MB by default)
wal_writer_delay = 200ms
wal_writer_flush_after = 1MB


# Background writer
bgwriter_delay = 200ms
bgwriter_lru_maxpages = 100
bgwriter_lru_multiplier = 2.0
bgwriter_flush_after = 0

# Parallel queries:
max_worker_processes = 4
max_parallel_workers_per_gather = 2
max_parallel_maintenance_workers = 2
max_parallel_workers = 4
parallel_leader_participation = on

# Advanced features
enable_partitionwise_join = on
enable_partitionwise_aggregate = on
jit = on
max_slot_wal_keep_size = '1000 MB'
track_wal_io_timing = on
maintenance_io_concurrency = 100
wal_recycle = on
```
  
</details>  
<details><summary><h6>64Gb/4CPU/2Disk/1000GbDB</h6></summary>

```bash
# Connectivity
max_connections = 100
superuser_reserved_connections = 3

# Memory Settings
shared_buffers = '16384 MB'
work_mem = '64 MB'
maintenance_work_mem = '620 MB'
huge_pages = try # NB! requires also activation of huge pages via kernel params, see here for more: https://www.postgresql.org/docs/current/static/kernel-resources.html#LINUX-HUGE-PAGES
effective_cache_size = '45 GB'
effective_io_concurrency = 100 # concurrent IO only really activated if OS supports posix_fadvise function
random_page_cost = 1.25 # speed of random disk access relative to sequential access (1.0)

# Monitoring
shared_preload_libraries = 'pg_stat_statements' # per statement resource usage stats
track_io_timing=on # measure exact block IO times
track_functions=pl # track execution times of pl-language procedures if any

# Replication
wal_level = replica # consider using at least 'replica'
max_wal_senders = 0
synchronous_commit = on

# Checkpointing:
checkpoint_timeout = '15 min'
checkpoint_completion_target = 0.9
max_wal_size = '10240 MB'
min_wal_size = '5120 MB'


# WAL writing
wal_compression = on
wal_buffers = -1 # auto-tuned by Postgres till maximum of segment size (16MB by default)
wal_writer_delay = 200ms
wal_writer_flush_after = 1MB


# Background writer
bgwriter_delay = 200ms
bgwriter_lru_maxpages = 100
bgwriter_lru_multiplier = 2.0
bgwriter_flush_after = 0

# Parallel queries:
max_worker_processes = 4
max_parallel_workers_per_gather = 2
max_parallel_maintenance_workers = 2
max_parallel_workers = 4
parallel_leader_participation = on

# Advanced features
enable_partitionwise_join = on
enable_partitionwise_aggregate = on
jit = on
max_slot_wal_keep_size = '1000 MB'
track_wal_io_timing = on
maintenance_io_concurrency = 100
wal_recycle = on
```
  
</details>  
<details><summary><h6>128Gb/8CPU/3Disk/1000GbDB</h6></summary>

```bash
# Connectivity
max_connections = 100
superuser_reserved_connections = 3

# Memory Settings
shared_buffers = '32768 MB'
work_mem = '64 MB'
maintenance_work_mem = '920 MB'
huge_pages = try # NB! requires also activation of huge pages via kernel params, see here for more: https://www.postgresql.org/docs/current/static/kernel-resources.html#LINUX-HUGE-PAGES
effective_cache_size = '90 GB'
effective_io_concurrency = 200 # concurrent IO only really activated if OS supports posix_fadvise function
random_page_cost = 1.25 # speed of random disk access relative to sequential access (1.0)

# Monitoring
shared_preload_libraries = 'pg_stat_statements' # per statement resource usage stats
track_io_timing=on # measure exact block IO times
track_functions=pl # track execution times of pl-language procedures if any

# Replication
wal_level = replica # consider using at least 'replica'
max_wal_senders = 0
synchronous_commit = on

# Checkpointing:
checkpoint_timeout = '15 min'
checkpoint_completion_target = 0.9
max_wal_size = '10240 MB'
min_wal_size = '5120 MB'


# WAL writing
wal_compression = on
wal_buffers = -1 # auto-tuned by Postgres till maximum of segment size (16MB by default)
wal_writer_delay = 200ms
wal_writer_flush_after = 1MB


# Background writer
bgwriter_delay = 200ms
bgwriter_lru_maxpages = 100
bgwriter_lru_multiplier = 2.0
bgwriter_flush_after = 0

# Parallel queries:
max_worker_processes = 8
max_parallel_workers_per_gather = 4
max_parallel_maintenance_workers = 4
max_parallel_workers = 8
parallel_leader_participation = on

# Advanced features
enable_partitionwise_join = on
enable_partitionwise_aggregate = on
jit = on
max_slot_wal_keep_size = '1000 MB'
track_wal_io_timing = on
maintenance_io_concurrency = 200
wal_recycle = on
```
  
</details>  
<details><summary><h6>256Gb/8CPU/4Disk/1000GbDB</h6></summary>

```bash
# Connectivity
max_connections = 100
superuser_reserved_connections = 3

# Memory Settings
shared_buffers = '65536 MB'
work_mem = '128 MB'
maintenance_work_mem = '1620 MB'
huge_pages = try # NB! requires also activation of huge pages via kernel params, see here for more: https://www.postgresql.org/docs/current/static/kernel-resources.html#LINUX-HUGE-PAGES
effective_cache_size = '179 GB'
effective_io_concurrency = 200 # concurrent IO only really activated if OS supports posix_fadvise function
random_page_cost = 1.25 # speed of random disk access relative to sequential access (1.0)

# Monitoring
shared_preload_libraries = 'pg_stat_statements' # per statement resource usage stats
track_io_timing=on # measure exact block IO times
track_functions=pl # track execution times of pl-language procedures if any

# Replication
wal_level = replica # consider using at least 'replica'
max_wal_senders = 0
synchronous_commit = on

# Checkpointing:
checkpoint_timeout = '15 min'
checkpoint_completion_target = 0.9
max_wal_size = '10240 MB'
min_wal_size = '5120 MB'


# WAL writing
wal_compression = on
wal_buffers = -1 # auto-tuned by Postgres till maximum of segment size (16MB by default)
wal_writer_delay = 200ms
wal_writer_flush_after = 1MB


# Background writer
bgwriter_delay = 200ms
bgwriter_lru_maxpages = 100
bgwriter_lru_multiplier = 2.0
bgwriter_flush_after = 0

# Parallel queries:
max_worker_processes = 8
max_parallel_workers_per_gather = 4
max_parallel_maintenance_workers = 4
max_parallel_workers = 8
parallel_leader_participation = on

# Advanced features
enable_partitionwise_join = on
enable_partitionwise_aggregate = on
jit = on
max_slot_wal_keep_size = '1000 MB'
track_wal_io_timing = on
maintenance_io_concurrency = 200
wal_recycle = on
```
  
</details>  
<details><summary><h6>512Gb/16CPU/4Disk/10000GbDB</h6></summary>

```bash
# Connectivity
max_connections = 300
superuser_reserved_connections = 3

# Memory Settings
shared_buffers = '131072 MB'
work_mem = '256 MB'
maintenance_work_mem = '2520 MB'
huge_pages = try # NB! requires also activation of huge pages via kernel params, see here for more: https://www.postgresql.org/docs/current/static/kernel-resources.html#LINUX-HUGE-PAGES
effective_cache_size = '358 GB'
effective_io_concurrency = 200 # concurrent IO only really activated if OS supports posix_fadvise function
random_page_cost = 1.25 # speed of random disk access relative to sequential access (1.0)

# Monitoring
shared_preload_libraries = 'pg_stat_statements' # per statement resource usage stats
track_io_timing=on # measure exact block IO times
track_functions=pl # track execution times of pl-language procedures if any

# Replication
wal_level = replica # consider using at least 'replica'
max_wal_senders = 0
synchronous_commit = on

# Checkpointing:
checkpoint_timeout = '15 min'
checkpoint_completion_target = 0.9
max_wal_size = '32768 MB'
min_wal_size = '16384 MB'


# WAL writing
wal_compression = on
wal_buffers = -1 # auto-tuned by Postgres till maximum of segment size (16MB by default)
wal_writer_delay = 200ms
wal_writer_flush_after = 1MB


# Background writer
bgwriter_delay = 200ms
bgwriter_lru_maxpages = 100
bgwriter_lru_multiplier = 2.0
bgwriter_flush_after = 0

# Parallel queries:
max_worker_processes = 16
max_parallel_workers_per_gather = 8
max_parallel_maintenance_workers = 8
max_parallel_workers = 16
parallel_leader_participation = on

# Advanced features
enable_partitionwise_join = on
enable_partitionwise_aggregate = on
jit = on
max_slot_wal_keep_size = '1000 MB'
track_wal_io_timing = on
maintenance_io_concurrency = 200
wal_recycle = on
```
  
</details>  

</details>

</details>














