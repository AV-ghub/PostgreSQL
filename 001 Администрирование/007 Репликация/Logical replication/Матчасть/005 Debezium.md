# Overview
The connector is tolerant of failures. As the connector reads changes and produces events, **it records the WAL position** for each event.  
If the connector stops for any reason (including communication failures, network problems, or crashes), upon restart the connector **continues reading the WAL where it last left** off. This includes snapshots.  
If the connector stops during a snapshot, the connector begins a new snapshot when it restarts.

