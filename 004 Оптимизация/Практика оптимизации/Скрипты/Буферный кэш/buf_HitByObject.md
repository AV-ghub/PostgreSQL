```sql
SELECT heap_blks_read, heap_blks_hit
FROM pg_statio_all_tables
WHERE relname = 'cacheme';
```
```
heap_blks_read | heap_blks_hit
−−−−−−−−−−−−−−−−+−−−−−−−−−−−−−−−
5 | 12
```
