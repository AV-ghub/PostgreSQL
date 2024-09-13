[Первоначальная настройка](https://github.com/AV-ghub/PostgreSQL/blob/main/004%20%D0%9E%D0%BF%D1%82%D0%B8%D0%BC%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F/%D0%9F%D1%80%D0%B0%D0%BA%D1%82%D0%B8%D0%BA%D0%B0%20%D0%BE%D0%BF%D1%82%D0%B8%D0%BC%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D0%B8/%D0%A1%D1%86%D0%B5%D0%BD%D0%B0%D1%80%D0%B8%D0%B8/%D0%A7%D0%B0%D1%81%D1%82%D0%BD%D1%8B%D0%B5/%D0%9A%D0%BE%D0%BD%D1%84%D0%B8%D0%B3%D1%83%D1%80%D0%B0%D1%86%D0%B8%D1%8F%20PostgreSQL%20%D0%BF%D0%BE%D1%81%D0%BB%D0%B5%20%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B8.md)

[Пример тюнинга с тестами](https://github.com/AV-ghub/PostgreSQL/blob/main/004%20%D0%9E%D0%BF%D1%82%D0%B8%D0%BC%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F/%D0%9F%D1%80%D0%B0%D0%BA%D1%82%D0%B8%D0%BA%D0%B0%20%D0%BE%D0%BF%D1%82%D0%B8%D0%BC%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D0%B8/%D0%A1%D1%86%D0%B5%D0%BD%D0%B0%D1%80%D0%B8%D0%B8/%D0%A7%D0%B0%D1%81%D1%82%D0%BD%D1%8B%D0%B5/PostgreSQL%20and%20OS%20tuning%20with%20perf%20tests.md)   


<details><summary><h2>Tuning Your PostgreSQL Server</h2></summary>

  [Tuning Your PostgreSQL Server](https://wiki.postgresql.org/wiki/Tuning_Your_PostgreSQL_Server)
  
</details>

<details><summary><h2>Server Configuration Tuning</h2></summary>
  
  [3](https://github.com/AV-ghub/PostgreSQL/blob/main/998%20Books/List.md).[135]

  The main tunable settings for PostgreSQL are in a plain text file named **postgresql.conf**  
  ```
  postgres@anisimov-ubuntu-pg-03:~$ pg_lsclusters 
  Ver Cluster Port Status Owner    Data directory              Log file
  16  main    5432 online postgres /var/lib/postgresql/16/main /var/log/postgresql/postgresql-16-main.log
  
  postgres@anisimov-ubuntu-pg-03:~$ pg_ctlcluster 16 main status
  pg_ctl: server is running (PID: 819)
  /usr/lib/postgresql/16/bin/postgres "-D" "/var/lib/postgresql/16/main" "-c" "config_file=/etc/postgresql/16/main/postgresql.conf"
  ```
  
  This chapter is more focused on the guidelines for _**setting the most important values**_.

  Default you can check this using the _**pg_settings**_ view by looking at the _**boot_val**_ column.  
  There's also a default value that parameters will return to if you use the _**RESET**_ command to return them to their starting value;   
  this is labeled as _**reset_val**_ in the pg_settings view.  
  
  ### Allowed change context
  Every configuration setting has an associated context in which it's allowed to be changed.  
  This example shows one entry with each context type
  ```
  pgbench=# select name, context from pg_settings limit 6;
              name            |  context  
  ----------------------------+-----------
   allow_in_place_tablespaces | superuser
   allow_system_table_mods    | superuser
   application_name           | user
   archive_cleanup_command    | sighup
   archive_command            | sighup
   archive_library            | sighup
  ```
* internal: database internals set at compile time.
* postmaster: full server restart. All shared memory settings fall into this category.
* sighup: Sending the server a HUP signal.
* backend: similar to the sighup ones, not impact any already running database backend sessions. Only new sessions started after this will respect the change. 
* superuser: can be modified by any database superuser, and made active without even requiring a full configuration reload. 
* user: Individual user sessions can adjust these parameters.
* superuser-backend: can be changed in the postgresql.conf file without restarting the PostgreSQL server. This value cannot be changed after starting the session, and only the superuser can change these setting.

### Reloading the configuration file
There are three ways you can get the database to reload its configuration in order _**to update values in the sighup category**_.   
If you're connected to the database _**as a superuser**_, _**pg_reload_conf**_ will do that:
```
postgres=# SELECT pg_reload_conf();
pg_reload_conf
----------------
t
```
You can send a HUP signal manually using the UNIX _**kill command**_:
```
$ ps -eaf | grep "postgres -D"
postgres 11185 1 0 22:21 pts/0 00:00:00
/home/postgres/inst/bin/postgres -D /home/postgres/data/
$ kill -HUP 11185
```
Finally, you can trigger a SIGHUP signal for the server by using _**pg_ctl**_:
```
$ -- pg_ctl reload -- deprecated
$ pg_ctlcluster 16 main reload
LOG: received SIGHUP, reloading configuration files
server signaled
```
No matter which approach you use, you'll see the following in the database log files afterwards to confirm that the server received the message:
```
LOG: received SIGHUP, reloading configuration files
```
You can then confirm that your changes have taken place as expected using commands such as **SHOW**, or by looking at **pg_settings**.

## Database connections
## Shared memory
## Logging
## Vacuuming and ststistics
## Checkpoins
## PITR and WAL replication
### effective_cache_size
PostgreSQL is expected to have both its own **dedicated memory (shared_buffers)** in addition to utilizing the **filesystem cache**.
When making decisions, the database compares **the sizes it computes** against the **effective sum of all these caches**;   
that's what it expects to find in **effective_cache_size**.

The same rough rule of thumb that would **put shared_buffers at 25%** of system memory would set **effective_cache_size to between 50% and 75% of RAM**.   
To get a more accurate estimate, first observe the size of the filesystem cache: **add the free and cached numbers** shown by the **free** or **top** commands to estimate the filesystem cache size.

~$ free   
||total|used|free|shared|buff/cache|available|
|:-|:-|:-|:-|:-|:-|:-|
|Mem:|$\color{red}{2015880}$|$\color{green}{499164}$|$\color{blue}{244116}$|38172|$\color{blue}{1272600}$|1290600|

~$ top   
...   
|MiB Mem :|$\color{red}{1968,6}$ total|$\color{blue}{238,1}$ free|$\color{green}{487,7}$ used|$\color{blue}{1242,8}$ buff/cache|  
|:-|:-|:-|:-|:-|


</details>

































