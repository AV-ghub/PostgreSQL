#### pg_clog 
The transaction commit log data is stored here.   
This data is actively read from, by VACUUM in particular, and files are removed once there's nothing interesting in them.    
This directory is one of the reasons PostgreSQL **doesn't work very well** if you mount the database in a way that **bypasses the filesystem cache**.   
**If** reads and writes to the commit logs are **not cached by the operating system**, it will **heavily reduce performance** in several common situations.
