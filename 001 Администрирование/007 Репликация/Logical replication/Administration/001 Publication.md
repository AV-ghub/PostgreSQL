# Publication
## 📋 Публикации базы
```sql
\c database
SELECT * FROM pg_publication;
```

## 📋 Объекты публикации

```sql
-- Все публикации и их таблицы
SELECT 
    pubname AS publication_name,
    schemaname AS schema_name,
    tablename AS table_name
FROM pg_publication_tables
ORDER BY pubname, schemaname, tablename;
```

## 🔍 Детальная информация о конкретной публикации

```sql
-- Для конкретной публикации
SELECT 
    schemaname AS schema_name,
    tablename AS table_name
FROM pg_publication_tables
WHERE pubname = 'flink_cdctest_pub'  -- ваше имя публикации
ORDER BY schemaname, tablename;
```

## 📊 Разница между `pg_publication` и `pg_publication_tables`

| Представление | Что показывает | Когда использовать |
|---------------|----------------|-------------------|
| **`pg_publication`** | Метаданные публикаций (имя, параметры) | Когда нужно узнать список всех публикаций и их глобальные настройки |
| **`pg_publication_tables`** | Конкретные таблицы в каждой публикации | Когда нужно узнать, какие таблицы входят в публикацию |

### Пример запроса к `pg_publication`:
```sql
-- Базовая информация о публикациях
SELECT 
    pubname,
    puballtables,
    pubinsert,
    pubupdate,
    pubdelete,
    pubtruncate
FROM pg_publication;
```
- **`puballtables`** = `true` если публикация включает ВСЕ таблицы
- **`pubinsert`**, **`pubupdate`**, **`pubdelete`**, **`pubtruncate`** — какие операции реплицируются

## 💡 Полезные запросы

### 1. Посчитать количество таблиц в каждой публикации:
```sql
SELECT 
    pubname,
    COUNT(*) as table_count
FROM pg_publication_tables
GROUP BY pubname
ORDER BY table_count DESC;
```

### 2. Найти таблицы, которые входят в несколько публикаций:
```sql
SELECT 
    schemaname,
    tablename,
    STRING_AGG(pubname, ', ') as publications,
    COUNT(*) as pub_count
FROM pg_publication_tables
GROUP BY schemaname, tablename
HAVING COUNT(*) > 1
ORDER BY pub_count DESC;
```

### 3. Проверить, какие операции разрешены для публикации:
```sql
SELECT 
    p.pubname,
    p.puballtables,
    p.pubinsert,
    p.pubupdate,
    p.pubdelete,
    p.pubtruncate,
    COUNT(pt.tablename) as tables_in_publication
FROM pg_publication p
LEFT JOIN pg_publication_tables pt ON p.pubname = pt.pubname
WHERE p.pubname = 'flink_cdctest_pub'
GROUP BY p.pubname, p.puballtables, p.pubinsert, p.pubupdate, 
         p.pubdelete, p.pubtruncate;
```

## ⚠️ Важные нюансы

1. **Если `puballtables = true`** в `pg_publication`, то в `pg_publication_tables` будут ВСЕ таблицы базы данных
2. **DDL изменения не отображаются автоматически** — если добавили таблицу через `ALTER PUBLICATION`, она сразу появится в `pg_publication_tables`
3. **Для публикаций с `puballtables = true`** можно узнать конкретный список таблиц только через `pg_publication_tables`

## 🎯 Практический пример

```sql
-- Полная диагностика публикации
SELECT 
    'Publication: ' || p.pubname AS info,
    'All tables: ' || CASE WHEN p.puballtables THEN 'YES' ELSE 'NO' END,
    'Operations: ' ||
        CASE WHEN p.pubinsert THEN 'INSERT ' ELSE '' END ||
        CASE WHEN p.pubupdate THEN 'UPDATE ' ELSE '' END ||
        CASE WHEN p.pubdelete THEN 'DELETE ' ELSE '' END ||
        CASE WHEN p.pubtruncate THEN 'TRUNCATE' ELSE '' END,
    'Tables count: ' || COUNT(pt.tablename)
FROM pg_publication p
LEFT JOIN pg_publication_tables pt ON p.pubname = pt.pubname
WHERE p.pubname = 'flink_cdctest_pub'
GROUP BY p.pubname, p.puballtables, p.pubinsert, p.pubupdate, 
         p.pubdelete, p.pubtruncate;
```

## [REPLICA IDENTITY](https://github.com/AV-ghub/PostgreSQL/blob/main/001%20%D0%90%D0%B4%D0%BC%D0%B8%D0%BD%D0%B8%D1%81%D1%82%D1%80%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5/007%20%D0%A0%D0%B5%D0%BF%D0%BB%D0%B8%D0%BA%D0%B0%D1%86%D0%B8%D1%8F/Logical%20replication/%D0%9C%D0%B0%D1%82%D1%87%D0%B0%D1%81%D1%82%D1%8C/004%20REPLICA%20IDENTITY.md#replica-identity)
