[3](https://github.com/AV-ghub/PostgreSQL/blob/main/998%20Books/List.md).[100]
* pg_clog 
The transaction commit log data is stored here.   
This data is actively read from, by VACUUM in particular, and files are removed once there's nothing interesting in them.    
This directory is one of the reasons PostgreSQL **doesn't work very well** if you mount the database in a way that **bypasses the filesystem cache**.   
**If** reads and writes to the commit logs are **not cached by the operating system**, it will **heavily reduce performance** in several common situations.

* relocate onto its own disk is **pg_xlog (pg_wal)**.   
* add more tablespaces to split out **heavily accessed tables**.   
* move **[temporary files](https://github.com/AV-ghub/PostgreSQL/blob/main/004%20%D0%9E%D0%BF%D1%82%D0%B8%D0%BC%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F/%D0%9F%D1%80%D0%B0%D0%BA%D1%82%D0%B8%D0%BA%D0%B0%20%D0%BE%D0%BF%D1%82%D0%B8%D0%BC%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D0%B8/%D0%90%D0%BF%D0%BF%D0%B0%D1%80%D0%B0%D1%82%D0%BD%D0%B0%D1%8F%20%D1%87%D0%B0%D1%81%D1%82%D1%8C/Disk%20Setup/Disk%20layout%20for%20PostgreSQL/Temporary%20files.md)**.
