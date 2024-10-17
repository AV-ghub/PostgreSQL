```sql
$ mkdir /disk/pgdata
$ psql
postgres=# CREATE TABLESPACE disk LOCATION '/disk/pgdata';
postgres=# CREATE TABLE t(i int) TABLESPACE disk;
```
Databases and tables are by default, created in a virtual tablespace named **pg_default**.   
You _**can change**_ that by setting the default_tablespace parameter in the _**server configuration**_.   
It's also possible to relocate an entire database by _**setting the TABLESPACE**_ parameter when running _**CREATE DATABASE**_.  
