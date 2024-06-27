[src](https://scalegrid.io/blog/postgresql-connection-pooling-part-4-pgbouncer-vs-pgpool/)
## Prerequisites
Using pgbench tool.    
Ran the same tests without a connection pooler too.   
## Testing Conditions
* scale factor of 100
* Disabled auto-vacuuming
* No other workload
* default pgbench script
* default settings for both PgBouncer and Pgpool-II
* ran as a single thread, on a single-CPU, 2-core machine, for a duration of 5 minutes
* Forced pgbench to create a new connection for each transaction using the -C option.

## Throughput Benchmark


## Final Words
Well-suited connection pooler can ***drastically increase the transaction throughput***, even with a fairly small number of clients.     
When the number of clients exceeds the max-clients supported by PostgreSQL server, PgBouncer is still able to maintain the transaction rate, whereas direct connections to PostgreSQL are aborted.

We must run Pgpool-II on a separate server to get a good performance. But even then, PgBouncer manages to provide better performance for these relatively small numbers of clients.

PgBouncer was the faster alternative and is the far better choice for connection pooling.    
