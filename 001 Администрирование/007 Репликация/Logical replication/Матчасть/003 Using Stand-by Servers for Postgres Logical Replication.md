# [Using Stand-by Servers for Postgres Logical Replication](https://www.decodable.co/blog/postgres-logical-replication)
## Why Use Logical Replication On Stand-By Servers?
When you have many logical replication slots and corresponding clients, this potentially can create an unduly heavy load on the primary database server. 

On the flip-side, there are some **implications** of this to consider.   
One is the slightly **increased end-to-end latency**: as changes are traveling from the primary server to the stand-by and then to the client, things will take a bit longer than when connecting your replication consumers directly to the primary.

While CDC in general **doesn’t require write access** to the database, there are some **exceptions** to that.

* In case of Debezium, you **won’t be able to use its incremental snapshotting** implementation, as this requires inserting watermarking events into a signaling table.
* Also you cannot use the **heartbeat feature**, which lets the connector **update its restart offsets** also in cases where there are no change events for any table the connector is capturing are coming in.

## Testing Things Out
The [test_decoding plug-in](https://www.postgresql.org/docs/current/test-decoding.html) which is used here, comes with Postgres out of the box and emits changes in a simple text-based representation.   
It allows you to examine the change events.
```
standby> SELECT * FROM pg_create_logical_replication_slot('demo_slot_standby', 'test_decoding');
```
```
standby> SELECT * FROM pg_logical_slot_get_changes('demo_slot_standby', NULL, NULL);
+-------------+------+----------------------------------------------------------------------------------------------------------------------------------------+
| lsn         | xid  | data                                                                                                                                   |
|-------------+------+----------------------------------------------------------------------------------------------------------------------------------------|
| 5E/90005120 | 6983 | BEGIN 6983                                                                                                                             |
| 5E/90005188 | 6983 | table public.some_data: INSERT: id[integer]:1 short_text[character varying]:'c4ca4' long_text[text]:'3a3c3274941c83e253ebf8d2438ea5a2' |
```

# [Stand-By Logical Replication With Debezium](https://www.decodable.co/blog/logical-replication-from-postgres-16-stand-by-servers-part-2-of-2)
[Debezium Releases Overview](https://debezium.io/releases/)   























