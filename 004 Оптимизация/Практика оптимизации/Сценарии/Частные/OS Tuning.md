[Researching PostgreSQL Performance](https://www.pgcon.org/2008/schedule/track/DBA/90.en.html)
## Пример тюнинга параметров PostgreSQL + Linux
***
### Hardware
***
* Server ***16GB*** RAM, ***4 dual­core*** processors
* Storage 1TB
* 3 RAID 5 (data, index and wal)
* Operation System: Debian Etch 4.0 AMD64.   
***
### Memory
***
```bash
echo "8589934592" > /proc/sys/kernel/shmmax
echo "deadline" > /sys/block/sdX/queue/scheduler
echo "250 128000 32 256" > /proc/sys/kernel/sem
echo "2" > /proc/sys/vm/overcommit_memory
echo "16777216" > /proc/sys/net/core/rmem_default
echo "16777216" > /proc/sys/net/core/wmem_default
echo "16777216" > /proc/sys/net/core/wmem_max
echo "16777216" > /proc/sys/net/core/rmem_max
```
### /etc/security/limits.conf
```bash
postgres soft nofile 63536
postgres soft nproc 2047
postgres hard nofile 63536
postgres hard nproc 16384
```
***
### basic postgresql.conf
***
```bash
listen_addresses = '*'
max_connections = 110
max_fsm_pages = 204800
effective_cache_size = 10GB
```
***
### shared_buffers 
***
4GB (25%)
***
### sync opensync vs fdatasync 
***
opensync is better with pgbench transaction
fdatasync is better with load tables of the pgbench
***
### wal_delay 
***
500
***
### autovacuum_cost_delay 
***
100
### checkpoints_segments 
256
***
### max_connections
***
* 700 or more?
* 1500 connections decrease 3%
* 3000 connections decrease 5%
***
### full_page_write 
***
off is more than faster
***
### deadline I/O scheduler
***
deadline I/O scheduler > CFQ
***
### filesystem
***
Ext3 (writeback) ­ better in few transactions
XFS ­ better in many transactions
***
### synchronous_commit 
***
disabled decrease performance
***
### autovacuum
***
to turn off autovacuum is a bad idea





























