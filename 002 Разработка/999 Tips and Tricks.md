[Replacing characters in string during loop in plpgsql](https://stackoverflow.com/a/22676183)

```
SELECT overlay('fooxxxxxxxbar' PLACING string_agg(g::text, '') FROM 4 FOR 7)
FROM   generate_series (1,4) g;

Result:
foo1234bar
```
[PostgreSQL OVERLAY() function](https://www.w3resource.com/PostgreSQL/overlay-function.php)  
[]()
