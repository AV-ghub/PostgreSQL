[3](https://github.com/AV-ghub/PostgreSQL/blob/main/998%20Books/List.md).[100]
* pg_clog 
The transaction commit log data is stored here.   
This data is actively read from, by VACUUM in particular, and files are removed once there's nothing interesting in them.    
This directory is one of the reasons PostgreSQL **doesn't work very well** if you mount the database in a way that **bypasses the filesystem cache**.   
**If** reads and writes to the commit logs are **not cached by the operating system**, it will **heavily reduce performance** in several common situations.

* relocate onto its own disk is **pg_xlog (pg_wal)**.   
* add more tablespaces to split out **heavily accessed tables**.   
* move **temporary files**.
