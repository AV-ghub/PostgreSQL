[src](https://blog.programs74.ru/how-to-install-yandex-odyssey/)



В мире пулеров PostgreSQL как известно 2 лидера — это Pgpool-II и PgBouncer, но есть и современные альтернативы в лице Yandex Odyssey и pgagroal.

В данной статье мы поговорим, а точнее установим и настроим Yandex Odyssey.


Исходные данные: ОС Oracle Linux 7
Задача: Собрать, установить и настроить пулер соединений Yandex Odyssey

В данной статье я не буду рассказывать об архитектуре или других особенностях Yandex Odyssey, все это хорошо описано в статье тут и на видео-докладе от создателей тут старее, а тут новее.

Отправной точкой для всего является репозиторий проекта на Github.

В принципе там выложен собранный бинарник версии 1.1, но непонятно с какой версией библиотеки libpq он собран, поэтому я опишу как собрать Odyssey из исходников.

Для начала у нас на сервере должен быть подключен оффициальный репозиторий пакетов PostgreSQL из которого мы установим необходимые для сборки пакеты, далее мы установим cmake 3 и pam-dev, скачаем исходники и соберем их.

1. Если оффициальный репозиторий PostgreSQL у Вас не подключен, то подключаем и устанавливаем devel-пакеты нужной версии PgSQL, у меня это будет 10-я версия.

yum -y localinstall https://download.postgresql.org/pub/repos/yum/11/redhat/rhel-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
yum -y install postgresql10-devel
2. Установим cmake и другие пакеты:

yum -y install cmake3 pam-devel
3. Сборка Odyssey из исходников:

cd ~
wget https://github.com/yandex/odyssey/archive/1.1.tar.gz -O odyssey-1.1.tar.gz
tar -zxf odyssey-1.1.tar.gz && rm -f odyssey-1.1.tar.gz
cd odyssey-1.1
mkdir build && cd build
cmake3 -DPOSTGRESQL_LIBPGPORT=/usr/pgsql-10/lib/libpgport.a -DPOSTGRESQL_LIBRARY=/usr/pgsql-10/lib/libpq.so -DPOSTGRESQL_INCLUDE_DIR=/usr/pgsql-10/include/server -DPQ_LIBRARY=/usr/pgsql-10/lib/libpq.a -DCMAKE_BUILD_TYPE=Release ..
make
Ниже я приведу вывод начала сборки и ее окончание:

# cmake3 -DPOSTGRESQL_LIBPGPORT=/usr/pgsql-10/lib/libpgport.a -DPOSTGRESQL_LIBRARY=/usr/pgsql-10/lib/libpq.so -DPOSTGRESQL_INCLUDE_DIR=/usr/pgsql-10/include/server -DPQ_LIBRARY=/usr/pgsql-10/lib/libpq.a -DCMAKE_BUILD_TYPE=Release ..
CMake Warning (dev) at /usr/share/cmake3/Modules/FindPackageHandleStandardArgs.cmake:272 (message):
  The package name passed to `find_package_handle_standard_args` (POSTGRESQL)
  does not match the name of the calling package (PostgreSQL).  This can lead
  to problems in calling code that expects `find_package` result variables
  (e.g., `_FOUND`) to follow a certain pattern.
Call Stack (most recent call first):
  cmake/FindPostgreSQL.cmake:52 (find_package_handle_standard_args)
  CMakeLists.txt:42 (find_package)
This warning is for project developers.  Use -Wno-dev to suppress it.
 
-- Found POSTGRESQL: /usr/pgsql-10/lib/libpq.so
-- Use shipped libmachinarium: /root/odyssey-1.1/third_party/machinarium
-- Use shipped libkiwi: /root/odyssey-1.1/third_party/kiwi
--
-- Odyssey (version: unknown release)
--
-- CMAKE_BUILD_TYPE:       Release
-- BUILD_DEBIAN:           OFF
-- POSTGRESQL_INCLUDE_DIR: /usr/pgsql-10/include/server
-- POSTGRESQL_LIBRARY:     /usr/pgsql-10/lib/libpq.so
-- POSTGRESQL_LIBPGPORT:   /usr/pgsql-10/lib/libpgport.a
-- PG_VERSION_NUM:         OFF
-- PQ_LIBRARY:             /usr/pgsql-10/lib/libpq.a
-- USE_BORINGSSL:          OFF
-- BORINGSSL_ROOT_DIR:
-- BORINGSSL_INCLUDE_DIR:
-- OPENSSL_VERSION:        1.0.2k
-- OPENSSL_ROOT_DIR:
-- OPENSSL_INCLUDE_DIR:    /usr/include
-- PAM_LIBRARY:            /usr/lib64/libpam.so
-- PAM_INCLUDE_DIR:        /usr/include/security
--
-- Configuring done
-- Generating done
-- Build files have been written to: /root/odyssey-1.1/build
 
# make
[  1%] Built target libkiwi
[  2%] Built target libmachinarium
[  2%] Built target build_libs
Scanning dependencies of target odyssey
[  3%] Building C object sources/CMakeFiles/odyssey.dir/daemon.c.o
[  4%] Building C object sources/CMakeFiles/odyssey.dir/pid.c.o
[  5%] Building C object sources/CMakeFiles/odyssey.dir/logger.c.o
....
....
[ 30%] Building C object sources/CMakeFiles/odyssey.dir/pam.c.o
[ 31%] Linking C executable odyssey
[ 31%] Built target odyssey
Scanning dependencies of target odyssey_test
[ 32%] Building C object test/CMakeFiles/odyssey_test.dir/odyssey_test.c.o
[ 33%] Building C object test/CMakeFiles/odyssey_test.dir/machinarium/test_init.c.o
[ 34%] Building C object test/CMakeFiles/odyssey_test.dir/machinarium/test_create0.c.o
....
....
[ 96%] Building C object test/CMakeFiles/odyssey_test.dir/odyssey/test_hgram.c.o
[ 97%] Linking C executable odyssey_test
[ 97%] Built target odyssey_test
Scanning dependencies of target odyssey_stress
[ 98%] Building C object stress/CMakeFiles/odyssey_stress.dir/odyssey_stress.c.o
[100%] Linking C executable odyssey_stress
[100%] Built target odyssey_stress
Если все прошло успешно и без ошибок, то у нас должен появиться бинарник sources/odyssey, проверим:

# file sources/odyssey
sources/odyssey: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=745f1f453c44b19ecc4782898c497dcbb4c0579f, not stripped
Отлично, идем дальше.

4. Создание пользователя, базовая настройка, запуск Odyssey:

Создание пользователя и вспомогательных каталогов:

groupadd --system odyssey
useradd --system --shell /sbin/nologin --gid odyssey --home-dir /var/lib/odyssey --no-create-home odyssey
mkdir /run/odyssey
chown -R odyssey:odyssey /run/odyssey
echo "d /run/odyssey 0755 odyssey odyssey - -" > /usr/lib/tmpfiles.d/odyssey.conf
mkdir /var/lib/odyssey
chown -R odyssey:odyssey /var/lib/odyssey
touch /var/log/odyssey.log
chown odyssey:odyssey /var/log/odyssey.log
mkdir /etc/odyssey
Копирование бинарника, unit-файла для systemd и базовая настройка Odyssey:

cd ~
cd odyssey-1.1
cp build/sources/odyssey /usr/bin
chmod a+x /usr/bin/odyssey
chown root:root /usr/bin/odyssey
cat odyssey.conf | sed \
-e '/^# pid_file.*/a \\pid_file \"/var/run/odyssey/odyssey.pid\"' \
-e 's|^unix_socket_dir.*|unix_socket_dir \"/var/run/odyssey\"|g' \
-e 's|^locks_dir.*|locks_dir \"/var/run/odyssey\"|g' \
> /etc/odyssey/odyssey.conf
chmod 644 /etc/odyssey/odyssey.conf
cp scripts/systemd/odyssey.service /usr/lib/systemd/system
systemctl daemon-reload
systemctl enable odyssey
Запуск Odyssey:

systemctl start odyssey
Проверим:

# systemctl status odyssey
● odyssey.service - Advanced multi-threaded PostgreSQL connection pooler and request router
   Loaded: loaded (/usr/lib/systemd/system/odyssey.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2020-09-10 13:36:30 MSK; 1s ago
 Main PID: 24848 (odyssey)
   CGroup: /system.slice/odyssey.service
           └─24848 /usr/bin/odyssey /etc/odyssey/odyssey.conf
 
Sep 10 13:36:30 srv-pg-01 odyssey[24848]: 24848 10 Sep 13:36:30.144 info [none none] (rules)   pool_rollback    yes
Sep 10 13:36:30 srv-pg-01 odyssey[24848]: 24848 10 Sep 13:36:30.144 info [none none] (rules)   client_fwd_error yes
Sep 10 13:36:30 srv-pg-01 odyssey[24848]: 24848 10 Sep 13:36:30.144 info [none none] (rules)   storage          postgres_server
Sep 10 13:36:30 srv-pg-01 odyssey[24848]: 24848 10 Sep 13:36:30.144 info [none none] (rules)   type             remote
Sep 10 13:36:30 srv-pg-01 odyssey[24848]: 24848 10 Sep 13:36:30.144 info [none none] (rules)   host             localhost
Sep 10 13:36:30 srv-pg-01 odyssey[24848]: 24848 10 Sep 13:36:30.144 info [none none] (rules)   port             5432
Sep 10 13:36:30 srv-pg-01 odyssey[24848]: 24848 10 Sep 13:36:30.144 info [none none] (rules)   log_debug        no
Sep 10 13:36:30 srv-pg-01 odyssey[24848]: 24848 10 Sep 13:36:30.144 info [none none] (rules)
Sep 10 13:36:30 srv-pg-01 odyssey[24848]: 24848 10 Sep 13:36:30.149 info [none none] (server) listening on 0.0.0.0:6432
Sep 10 13:36:30 srv-pg-01 odyssey[24848]: 24848 10 Sep 13:36:30.150 error [none none] (server) bind to '[::]:6432' failed: Address family not support... protocol
Hint: Some lines were ellipsized, use -l to show in full.
Поверим процесс:

# ps -ef | grep [o]dyssey
odyssey  24848     1  0 13:36 ?        00:00:00 /usr/bin/odyssey /etc/odyssey/odyssey.conf
Проверим порт:

# netstat -ltupn | grep odyssey
tcp        0      0 0.0.0.0:6432            0.0.0.0:*               LISTEN      24848/odyssey
Итак, мы видим, что Odyssey запустился с базовым конфигом и принимает сетевые подключения на всех сетевых интерфейсах по порту 6432

Далее нам нужно настроить пул для нашей базы PostgreSQL, а возможно несколько пулов и баз, а так же необходимо настроить сервисную базу из которой можно получать статистику работы Odyssey.

Начнем настройку:

1. Для начала немного безопасности.

Как мы знаем Odyssey принимает соединения на всех интерфейсах по порту 6432, это может быть опасно если Ваш сервер смотрит в Интернет и Вы не закроете порт 6432 для всех желающих.
В моем случае я перенастрою Odyssey чтобы он принимал соединения только со 127.0.0.1, для этого откроем файл конфигурации /etc/odyssey/odyssey.conf и в секции listen отредактируем параметр host

Получилось так:

listen {
        host "127.0.0.1"
        port 6432
        backlog 128
}
2. Настроим административную консоль для просмотра статистики.

Для этого откроем файл конфигурации /etc/odyssey/odyssey.conf и добавим в конец строки:

storage "local" {
        type "local"
}
database "console" {
        user default {
                authentication "none"
                pool "session"
                storage "local"
        }
}
Сохраняем конфиг и перезапускаем Odyssey:

systemctl restart odyssey
Проверим порт:

# netstat -ltupn | grep odyssey
tcp        0      0 127.0.0.1:6432          0.0.0.0:*               LISTEN      13499/odyssey
Отлично, теперь Odyssey принимает подключения только на 127.0.0.1 порт 6432

Теперь можно подключиться к сервисной базе:

psql -h 127.0.0.1 -p 6432 -d console
Попробуем ввести некоторые команды для просмотра состояния. К слову сказать, разработчики Odyssey постарались сделать такие же команды как и у PgBouncer, хотя далеко не все.

psql (10.9, server 9.6.0)
Type "help" for help.
 
console=> show stats;
 database  | total_xact_count | total_query_count | total_received | total_sent | total_xact_time | total_query_time | total_wait_time | avg_xact_count | avg_query_count | avg_recv | avg_sent | avg_xact_time | avg_query_time | avg_wait_time
-----------+------------------+-------------------+----------------+------------+-----------------+------------------+-----------------+----------------+-----------------+----------+----------+---------------+----------------+---------------
 console   |                0 |                 0 |              0 |          0 |               0 |                0 |               0 |              0 |               0 |        0 |        0 |             0 |              0 |             0
(1 rows)
 
console=> show pools;
 database  |   user   | cl_active | cl_waiting | sv_active | sv_idle | sv_used | sv_tested | sv_login | maxwait | maxwait_us | pool_mode
-----------+----------+-----------+------------+-----------+---------+---------+-----------+----------+---------+------------+-----------
 console   | root     |         0 |          1 |         0 |       0 |       0 |         0 |        0 |       0 |          0 | session
(1 rows)
 
console=> show clients;
 type |   user   | database |  state  |   addr    | port  | local_addr | local_port | connect_time | request_time | wait | wait_us |      ptr      | link | remote_pid | tls
------+----------+----------+---------+-----------+-------+------------+------------+--------------+--------------+------+---------+---------------+------+------------+-----
 C    | root     | console  | pending | 127.0.0.1 | 27194 | 127.0.0.1  |       6432 |              |              |    0 |       0 | cbbfabe55cdbf |      |          0 |
(1 rows)
 
console=> show lists;
     list      | items
---------------+-------
 databases     |     0
 users         |     0
 pools         |     1
 free_clients  |     0
 used_clients  |     1
 login_clients |     0
 free_servers  |     0
 used_servers  |     0
 dns_names     |     0
 dns_zones     |     0
 dns_queries   |     0
 dns_pending   |     0
(12 rows)
 
console=> show databases;
   name    |   host    | port | database  | force_user | pool_size | reserve_pool | pool_mode | max_connections | current_connections | paused | disabled
-----------+-----------+------+-----------+------------+-----------+--------------+-----------+-----------------+---------------------+--------+----------
 console   |           |    0 | console   |            |         0 |            0 | session   |               0 |                   1 |      0 |        0
(1 rows)
 
console=> show servers;
 type |   user   | database | state  |   addr    | port | local_addr | local_port | connect_time | request_time | wait | wait_us |      ptr      | link | remote_pid | tls
------+----------+----------+--------+-----------+------+------------+------------+--------------+--------------+------+---------+---------------+------+------------+-----
(0 rows)
3. Включим ведение лога.

Для этого откроем файл конфигурации /etc/odyssey/odyssey.conf и раскоментируем настройку log_file, у меня получилось так:

log_file "/var/log/odyssey.log"
Файл /var/log/odyssey.log должен существовать и его владельцем должен быть пользователь odyssey. Мы создали файл в самом начале статьи.

Сохраняем конфиг и перезапускаем Odyssey:

systemctl restart odyssey
Теперь можно посмотреть что будет писаться в лог:

tail -f /var/log/odyssey.log
Пример записей при подключении к административной консоле:

13499 10 Sep 15:35:29.516 info [c01ac457670b6 none] (main) client disconnected
13499 10 Sep 15:35:32.336 info [ca45b52665e0c none] (startup) new client connection 127.0.0.1:27590
13499 10 Sep 15:35:32.336 info [ca45b52665e0c none] (startup) route 'console.root' to 'console.default'
Формат лога очень гибко настраивается параметром log_format
Так же можно настроить ведение отладочной информации в лог — log_debug
Можно настроить запись в лог конфигурации при старте — log_config
Можно настроить запись статистики сессий в лог — log_session
Можно настроить запись клиентских запросов в лог — log_query
Настроить периодическую запись в лог статистики — log_stats и ее интервал — stats_interval
А так же можно настроить запись логов в syslog — log_syslog, log_syslog_ident, log_syslog_facility
И даже вывод лога в stdout — log_to_stdout
В общем очень много всего по логгированию, что довольно удобно.

4. Теперь займемся настройкой сторджей, баз и пользователей для работы пулера.

Odyssey позволяет очень гибко настраивать пулы с привязкой к конкретной базе и даже к конкретному пользователю, а так же позволяет настраивать для них различные методы аутентификации: по паролю или сертификату (clear_text, md5, scram-sha-256, certificate), по паролю из какой-то внешней базы (конечно только из postgresql базы) и через PAM.

Секции storage (мы их уже видели когда настраивали административный доступ) описывают подключение к удаленному (type «remote») серверу PostgreSQL, по умолчанию настроена одна секция storage с именем «postgres_server» с параметром host «localhost», то есть пулер будет подключаться к PostgreSQL расположенному на этом же хосте что и он сам и на стандартном порту 5432, вот пример этой секции:

storage "postgres_server" {
    type "remote"
    host "localhost"
    port 5432
}
Учитывая, что у меня PostgreSQL работает там же где и пулер, то я эту секцию не меняю.

Следующая секция, которая есть по умолчанию — это database с именем default, ниже все ее настройки. Если вы откроете конфиг, то ко всем настройкам Вы увидите детальное описание, что довольно удобно и позволяет легко сделать новые секции с необходимыми настройками.

database default {
    user default {
        authentication "none"
        storage "postgres_server"
        pool "transaction"
        pool_size 0
        pool_timeout 0
        pool_ttl 60
        pool_discard no
        pool_cancel yes
        pool_rollback yes
        client_fwd_error yes
        application_name_add_host yes
        server_lifetime 3600
        log_debug no
        quantiles "0.99,0.95,0.5"
    }
}
В секцию default направляются все соединения если не были найдены иные маршруты для баз и пользователей. Как мы видим секция default связана с сервером (storage) «postgres_server».

Особого смысла менять секцию default нет, мы создадим отдельные маршруты для конкретных баз и пользователей чуть ниже. Все что не будет подпадать под наши правила будет отправляться в default, а учитывая что там указан authentication «none», то никто не сможет подключиться к PostgreSQL.

Давайте создадим 1 маршрут для базы pgbench и пользователя pgtest.

Для начала создадим пользователя и базу в Pg.

psql -qAtXc "CREATE ROLE pgtest WITH LOGIN PASSWORD 'verybigandstrongpasswd';"
psql -qAtXc "CREATE DATABASE pgbench OWNER pgtest;"
Проверим соединение:

# psql -h 127.0.0.1 -U pgtest -W pgbench
pgbench=> \conninfo
You are connected to database "pgbench" as user "pgtest" on host "127.0.0.1" at port "5432".
Теперь добавим в файл конфигурации /etc/odyssey/odyssey.conf настройки:

database "pgbench" {
    user "pgtest" {
        authentication "md5"
        password "verybigandstrongpasswd"
        storage "postgres_server"
        pool "transaction"
        pool_size 5
        pool_timeout 3
        pool_ttl 60
        pool_discard no
        pool_cancel yes
        pool_rollback yes
        client_fwd_error yes
        application_name_add_host yes
        server_lifetime 3600
        log_debug no
        quantiles "0.99,0.95,0.5"
    }
}
Это типовые настройки, я скорректировал только несколько из них:

authentication "md5"
password "verybigandstrongpasswd"
pool_size 5
pool_timeout 3
Теперь можно либо перезапустить Odyssey, либо послать сигнал HUP процессу для перезагрузки конфигурации.

systemctl restart odyssey
Теперь можно подключиться к базе pgbench через пулер:

psql -h 127.0.0.1 -p 6432 -U pgtest -W -d pgbench
Вводим пароль и мы должны успешно подключиться, в лог /var/log/odyssey.log будет что-то подобное:

21564 18 Sep 10:51:41.001 info [cc39f41130933 none] (startup) new client connection 127.0.0.1:39592
21564 18 Sep 10:51:41.001 info [cc39f41130933 none] (startup) route 'pgbench.pgtest' to 'pgbench.pgtest'
21564 18 Sep 10:51:41.001 info [cc39f41130933 sd46abe7338d4] (setup) new server connection localhost:5432 (connect time: 226 usec, resolve time: 140 usec)
21564 18 Sep 10:51:41.004 info [cc39f41130933 none] (setup) login time: 3241 microseconds
Посмотрим данные из административной консоли Odyssey:

psql -h 127.0.0.1 -p 6432 -d console
 
console=> show pools;
 database  |   user   | cl_active | cl_waiting | sv_active | sv_idle | sv_used | sv_tested | sv_login | maxwait | maxwait_us |  pool_mode
-----------+----------+-----------+------------+-----------+---------+---------+-----------+----------+---------+------------+-------------
 pgbench   | pgtest   |         0 |          1 |         0 |       0 |       0 |         0 |        0 |       0 |          0 | transaction
 console   | root     |         0 |          1 |         0 |       0 |       0 |         0 |        0 |       0 |          0 | session
(2 rows)
 
console=> show servers;
 type |   user   | database  | state  |   addr    | port | local_addr | local_port | connect_time | request_time | wait | wait_us |      ptr      | link | remote_pid | tls
------+----------+-----------+--------+-----------+------+------------+------------+--------------+--------------+------+---------+---------------+------+------------+-----
 S    | pgtest   | pgbench   | idle   | 127.0.0.1 | 5432 | 127.0.0.1  |      13834 |              |              |    0 |       0 | sc98d31063787 |      |          0 |
(1 rows)
 
console=> show databases;
   name    |   host    | port | database  | force_user | pool_size | reserve_pool |  pool_mode  | max_connections | current_connections | paused | disabled
-----------+-----------+------+-----------+------------+-----------+--------------+-------------+-----------------+---------------------+--------+----------
 pgbench   | localhost | 5432 | pgbench   |            |         5 |            0 | transaction |               0 |                   1 |      0 |        0
 console   |           |    0 | console   |            |         0 |            0 | session     |               0 |                   1 |      0 |        0
(2 rows)
 
console=> show clients;
 type |   user   | database |  state  |   addr    | port  | local_addr | local_port | connect_time | request_time | wait | wait_us |      ptr      | link | remote_pid | tls
------+----------+----------+---------+-----------+-------+------------+------------+--------------+--------------+------+---------+---------------+------+------------+-----
 C    | pgtest   | pgbench  | pending | 127.0.0.1 | 39592 | 127.0.0.1  |       6432 |              |              |    0 |       0 | cc39f41130933 |      |          0 |
 C    | root     | console  | pending | 127.0.0.1 | 43076 | 127.0.0.1  |       6432 |              |              |    0 |       0 | c4def02139fe8 |      |          0 |
(2 rows)
Мы видим данные о нашем пуле, сервере, базе и соединении.

На этом все, до скорых встреч. Если у Вас возникли вопросы или Вы хотите чтобы я помог Вам, то Вы всегда можете связаться со мной разными доступными способами.
















Блог о системном администрированииБлог о системном администрировании
Сисадмин - это не профессия, это призвание.
О блоге
Добро пожаловать в мой блог о системном администрировании и администрировании баз данных.

Меня зовут Михаил Григорьев, я профессионально занимаюсь системным администрированием Linux (Debian, Ubuntu, CentOS, Oracle Linux) и Windows (2003/2008/2012) серверов на протяжении последних 24 лет.

В данном блоге Вы найдете в основном мои статьи о системном администрировании, базах данных и другой ИТ-тематике. Так же в блоге могут быть статьи других авторов, но тематика и общий стиль будут сохранены.

Мое подробное резюме можно посмотреть на hh.ru

Если у Вас возникли вопросы или есть ко мне предложения, то Вы можете написать мне на
Email: sleuthhound AT programs74 DOT ru или sleuthhound AT gmail DOT com
Telegram: @CHERTS

Так же Вы можете оказать помощь через:
YMoney: https://yoomoney.ru/fundraise/NVjVpgIuJiE.230108
Paypal: https://paypal.me/mikhailgrigorev1981
