# [Overview](https://debezium.io/documentation/reference/stable/connectors/postgresql.html#postgresql-overview)
The connector is tolerant of failures. As the connector reads changes and produces events, **it records the WAL position** for each event.  
If the connector stops for any reason (including communication failures, network problems, or crashes), upon restart the connector **continues reading the WAL where it last left** off. This includes snapshots.  
If the connector stops during a snapshot, the connector begins a new snapshot when it restarts.

Because logical decoding replication slots publish changes during commit — and not post commit — undesirable side-effects can occur.  
For example, a DebeziumEngine consumer receives a notification of a row that was created but it cannot be read by a transaction.

> Debezium currently supports databases with **UTF-8 character encoding only**.

## [How the connector works](https://debezium.io/documentation/reference/stable/connectors/postgresql.html#how-the-postgresql-connector-works)

### Security
It is best to create a dedicated Debezium replication user to which you grant specific privileges.  

#### [Setting up permissions](https://debezium.io/documentation/reference/stable/connectors/postgresql.html#postgresql-permissions)
```
CREATE ROLE <name> REPLICATION LOGIN;
```

In general, it is best to manually create publications for the tables that you want to capture, before you set up the connector.  
However, you can configure your environment in a way that permits Debezium to create publications automatically, and to specify the data that is added to them. 

### [Snapshots](https://debezium.io/documentation/reference/stable/connectors/postgresql.html#postgresql-snapshots)
Ad hoc snapshots require the use of signaling tables. You initiate an ad hoc snapshot by sending a signal request to the Debezium signaling table.

#### [Creating a signaling data collection](https://debezium.io/documentation/reference/stable/configuration/signalling.html#debezium-signaling-creating-a-signal-data-collection)
You create a signaling table by submitting a standard SQL DDL **query to the source database**.

When you initiate an ad hoc snapshot of an existing table, the connector appends content to the topic that already exists for the table.  
If a previously existing topic was removed, Debezium can create a topic automatically if automatic topic creation is enabled.































