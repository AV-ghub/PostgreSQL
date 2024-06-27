[src](https://eax.me/pgbouncer/)

Типичные веб-проекты, разрабатываемые на чем-то вроде Python или PHP, характерны тем, что ***создают большое количество соединений*** к СУБД — ***по одному, а иногда и по несколько, на каждый HTTP-запрос***. Имея классическую архитектуру «один процесс на соединение», ***PostgreSQL не очень хорошо справляется с большим (условно, больше 100)*** количеством соединений. ***PgBouncer может поддерживать большое количество (тысячи)*** соединений, которые ***проксируются на несколько (пара десятков)*** соединений непосредственно к PostgreSQL.

Устанавливается PgBouncer очень просто:
```bash
sudo apt-get install pgbouncer
```
По умолчанию прокси слушает ***порт 6432***. Логи можно почитать так:
```bash
less /var/log/postgresql/pgbouncer.log.
```
Конфигурационный файл называется ***/etc/pgbouncer/pgbouncer.ini***
```bash
;; database name = connect string
;;
;; connect string params:
;;   dbname= host= port= user= password=
;;   client_encoding= datestyle= timezone=
;;   pool_size= connect_query=
;;   auth_user=
[databases]

* = host=localhost port=5432
```
Здесь мы можем указать, на каких серверах какие базы нужно искать. Звездочкой обозначается дэфолтный сервер

важная настройка:
```bash
; When server connection is released back to pool:
;   session      - after client disconnects
;   transaction  - after transaction finishes
;   statement    - after statement finishes
pool_mode = transaction
```
Настройки размера пула:
```bash
; total number of clients that can connect
max_client_conn = 1000

; default pool size.  20 is good number when transaction pooling
; is in use, in session pooling it needs to be the number of
; max clients you want to handle at any moment
default_pool_size = 20

;; Minimum number of server connections to keep in pool.
;min_pool_size = 0
```
Настройки аутентификации:
```bash
; any, trust, plain, crypt, md5, cert, hba, pam
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
```
и доступа к админке pgbouncer:
```bash
;;;
;;; Users allowed into database 'pgbouncer'
;;;

; comma-separated list of users, who are allowed to change settings
admin_users = eax

; comma-separated list of users who are just allowed to use
;    SHOW command
;stats_users = stats, root
```
> Параметр auth_type по умолчанию имеет значение trust. То есть, PgBouncer будет пускать всех в базу данных без запроса пароля.

Файл ***/etc/pgbouncer/userlist.txt*** содержит имена пользователей и пароли, с которыми PgBouncer подключается к базе. Например:
```bash
"eax" "qwerty"
```
Пароли не обязательно хранить открытым текстом:
```bash
"eax" "md5175c641eaed0b6c05ae8444b73d789f0"
```
Здесь хэш 175c641e... был посчитан как MD5 от пароля, следом за которым записано имя пользователя:
```bash
echo -n 'qwertyeax' | md5sum
```
Теперь, когда PgBouncer настроен, можно перезапустить его:
```bash
sudo service pgbouncer restart
```
и ломиться в базу:
```bash
psql -p 6432 -U eax
```
Если все было сделано правильно, PgBouncer запросит пароль. В приведенном примере пароль, с которым пользователь ходит в PostgreSQL напрямую, и пароль, запрашиваемый PgBouncer — это один и тот же пароль. Если вдруг что-то не работает, рекомендую начать с проверки настройки аутентификации самого PostgreSQL (файл pg_hba.conf).

Выше пользователь eax был добавлен в admin_users, что дает ему доступ в админку PgBouncer. Вход в админку осуществляется так:
```bash
psql -p 6432 -U eax pgbouncer
```
Наиболее интересная из доступных команд — это перечитывание конфигурации без перезапуска сервера:
```bash
RELOAD;
```
Информацию о других доступных командах можно посмотреть так:
```bash
SHOW HELP;
```
Пример вывода:
```bash
DETAIL:
    SHOW HELP|CONFIG|DATABASES|POOLS|CLIENTS|SERVERS|VERSION
    SHOW FDS|SOCKETS|ACTIVE_SOCKETS|LISTS|MEM
    SHOW DNS_HOSTS|DNS_ZONES
    SHOW STATS|STATS_TOTALS|STATS_AVERAGES
    SET key = arg
    RELOAD
    PAUSE [<db>]
    RESUME [<db>]
    DISABLE <db>
    ENABLE <db>
    KILL <db>
    SUSPEND
    SHUTDOWN
```
Проверяем, что все работает, запустив pgbench с 900 соединенинями:
```bash
pgbench -i
pgbench -p 6432 -c 900 -C -T 60 -P 1
```
Если все было сделано правильно, pgbench будет превосходно работать, а команда SHOW CLIENTS;, выполненная в админке PgBouncer, покажет 900 соединений.    
При этом вывод
```bash
ps wuax | grep postgres
```
покажет, что реально запущено лишь 20 бэкендов PostgreSQL.





