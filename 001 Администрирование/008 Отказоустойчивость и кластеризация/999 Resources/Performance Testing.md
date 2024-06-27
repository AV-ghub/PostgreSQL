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
![Benchmark](https://github.com/AV-ghub/PostgreSQL/blob/main/001%20%D0%90%D0%B4%D0%BC%D0%B8%D0%BD%D0%B8%D1%81%D1%82%D1%80%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5/008%20%D0%9E%D1%82%D0%BA%D0%B0%D0%B7%D0%BE%D1%83%D1%81%D1%82%D0%BE%D0%B9%D1%87%D0%B8%D0%B2%D0%BE%D1%81%D1%82%D1%8C%20%D0%B8%20%D0%BA%D0%BB%D0%B0%D1%81%D1%82%D0%B5%D1%80%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F/999%20Resources/images/PgBouncer%20vs%20Pgpool-II%20Throughput%20Benchmark.png)
![Benchmark prc](https://github.com/AV-ghub/PostgreSQL/blob/main/001%20%D0%90%D0%B4%D0%BC%D0%B8%D0%BD%D0%B8%D1%81%D1%82%D1%80%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5/008%20%D0%9E%D1%82%D0%BA%D0%B0%D0%B7%D0%BE%D1%83%D1%81%D1%82%D0%BE%D0%B9%D1%87%D0%B8%D0%B2%D0%BE%D1%81%D1%82%D1%8C%20%D0%B8%20%D0%BA%D0%BB%D0%B0%D1%81%D1%82%D0%B5%D1%80%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F/999%20Resources/images/PgBouncer%20vs%20Pgpool-II%20Throughput%20Benchmark%20prc.png)

## Final Words
Well-suited connection pooler can ***drastically increase the transaction throughput***, even with a fairly small number of clients.     
When the number of clients exceeds the max-clients supported by PostgreSQL server, PgBouncer is still able to maintain the transaction rate, whereas direct connections to PostgreSQL are aborted.

We must run Pgpool-II on a separate server to get a good performance. But even then, PgBouncer manages to provide better performance for these relatively small numbers of clients.

PgBouncer was the faster alternative and is the far better choice for connection pooling.    
