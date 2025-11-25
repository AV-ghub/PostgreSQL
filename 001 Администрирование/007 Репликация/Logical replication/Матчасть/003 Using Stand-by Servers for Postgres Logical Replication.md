# [Using Stand-by Servers for Postgres Logical Replication](https://www.decodable.co/blog/postgres-logical-replication)
## Why Use Logical Replication On Stand-By Servers?
When you have many logical replication slots and corresponding clients, this potentially can create an unduly heavy load on the primary database server. 

On the flip-side, there are some **implications** of this to consider.   
One is the slightly **increased end-to-end latency**: as changes are traveling from the primary server to the stand-by and then to the client, things will take a bit longer than when connecting your replication consumers directly to the primary.

While CDC in general **doesn’t require write access** to the database, there are some **exceptions** to that.

* In case of Debezium, you **won’t be able to use its incremental snapshotting** implementation, as this requires inserting watermarking events into a signaling table.
* Also you cannot use the **heartbeat feature**, which lets the connector **update its restart offsets** also in cases where there are no change events for any table the connector is capturing are coming in.






















