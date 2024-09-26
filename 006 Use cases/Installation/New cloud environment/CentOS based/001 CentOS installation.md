<details><summary><h4><a href="https://github.com/AV-ghub/PostgreSQL-Cloud-Solutions/blob/main/Linux/CentOS/Intro/Installation/001%20Installation.md">CentOS 7 Installation</a></h4></summary>

  ### Шаг 2: Создание виртуальной машины для CentOS

  * Запустите VirtualBox Менеджер и нажмите на кнопку «Создать»
  * Впишите имя машины
  * Укажите объем оперативной памяти
  * Оставьте выбранным пункт «Создать новый виртуальный жесткий диск»
  * Тип тоже не меняйте и оставьте VDI
  * Предпочтительный формат хранения — «динамический»
  * Размер для виртуального HDD выберите, исходя из доступного свободного места

  ### Шаг 3: Настройка виртуальной машины
  * Для входа в настройки нужно нажать правой кнопкой мыши по виртуальной машине и выбрать пункт «Настроить»
  * Во вкладке «Система» — «Процессор» можно увеличить количество процессоров до 2
  * Перейдя в «Дисплей», можете добавить некоторое количество МБ к видеопамяти и включить 3D-ускорение

  ### Шаг 4: Установка CentOS
  * Выделите кликом мыши виртуальную машину и нажмите на кнопку «Запустить»
  * После запуска VM нажмите на папку и через стандартный системный проводник укажите место, куда вы скачали образ ОС
  * Запустится установщик системы. При помощи стрелки вверх на клавиатуре выберите пункт «Install CentOS Linux 7» и нажмите Enter
  * Запустится графический установщик CentOS. Выберите ваш язык и его разновидность.
  * В окне с параметрами настройте:
  * Часовой пояс
  * Расположение установки
  * зайдите в меню с настройками, выделите виртуальный накопитель, который был создан вместе с виртуальной машиной, и нажмите «Готово»
  * Выбор программ
  * По умолчанию стоит минимальная установка, но она не имеет графического интерфейса. Вы можете выбрать, с какой средой будет установлена ОС: GNOME или KDE. Выбор зависит от ваших предпочтений, а мы рассмотрим инсталляцию с окружением KDE. После выбора оболочки в правой части окна появятся дополнения. Галочками можете отметить то, что хотели бы видеть в CentOS. По завершении выбора нажмите «Готово». ![](https://github.com/AV-ghub/PostgreSQL/blob/main/006%20Use%20cases/Installation/New%20cloud%20environment/CentOS%20based/Res/%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0%20CentOS%203.jpg)
  * Нажмите на кнопку «Начать установку».
  * Во время установки (состояние отображается в нижней части окна как прогресс-бар) вам будет предложено придумать пароль root и создать пользователя.
  * Впишите пароль для прав root (суперпользователя) 2 раза и нажмите «Готово». Если пароль будет простым, кнопку «Готово» потребуется нажать дважды. Не забудьте сперва переключить раскладку клавиатуры на английский язык. Текущий язык можно увидеть в правом верхнем углу окна.
  * Впишите желаемые инициалы в поле «Полное имя». Строка «Имя пользователя» будет заполнена автоматически, но ее можно изменить вручную. При желании назначьте этого пользователя администратором, установив соответствующую галочку. Придумайте пароль для учетной записи и нажмите «Готово».
  * Дождитесь установки ОС и нажмите на кнопку «Завершить настройку».
  * Нажмите на кнопку «Перезагрузка».
  * Появится загрузчик GRUB, который по умолчанию через 5 секунд продолжит загрузку ОС. Можно сделать это вручную, не дожидаясь таймера, нажав на Enter.
  * Появится окно загрузки CentOS.
  * Снова отобразится окно с настройками. На этот раз нужно принять условия лицензионного соглашения и настроить сеть.
  * Чтобы включить интернет, нажмите на параметр «Сеть и имя узла». Кликните на регулятор, и он сдвинется вправо.  ![](https://github.com/AV-ghub/PostgreSQL/blob/main/006%20Use%20cases/Installation/New%20cloud%20environment/CentOS%20based/Res/%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0%20CentOS%204.jpg)
  * Нажмите на кнопку «Завершить».
  * Вы попадете на экран входа в учетную запись. Кликните на нее.
  * Переключите раскладку клавиатуры, введите пароль и нажмите «Войти»

</details>

<details><summary><h4><a href="https://confluence.speechpro.com/pages/viewpage.action?pageId=165486262">Интернет. Прозрачный прокси-сервер (transparent proxy)</a></h4></summary>

  ```
  wget --no-check-certificate https://...
  wget --no-check-certificate https://...
  
  ```

</details>

<details><summary><h4><a href="https://serverfault.com/questions/904304/could-not-resolve-host-mirrorlist-centos-org-centos-7">Could not resolve host: mirrorlist.centos.org Centos 7</a></h4></summary>

  
  From first of July 2024 on CentOS 7, please switch to Vault archive repositories:

  ```
  sudo nano /etc/yum.repos.d/CentOS-Base.repo
  ```
  
  copy/paste the following and mind your OS version. Change if needed. In this config is version 7.9.2009:
  
  ```
  [base]
  name=CentOS-$releasever - Base
  baseurl=http://vault.centos.org/7.9.2009/os/$basearch/
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
  
  [updates]
  name=CentOS-$releasever - Updates
  baseurl=http://vault.centos.org/7.9.2009/updates/$basearch/
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
  
  [extras]
  name=CentOS-$releasever - Extras
  baseurl=http://vault.centos.org/7.9.2009/extras/$basearch/
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
  
  [centosplus]
  name=CentOS-$releasever - Plus
  baseurl=http://vault.centos.org/7.9.2009/centosplus/$basearch/
  gpgcheck=1
  enabled=0
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
  ```

  If needed, do:
  
  ```
  yum clean all
  ```

</details>

<details><summary><h4><a href="https://github.com/AV-ghub/PostgreSQL-Cloud-Solutions/blob/main/Linux/Ubuntu/001%20Installation.md#change-hostname">Change hostname</a></h4></summary>

  ```
  :~$ hostname
  :~$ hostnamectl
  
  # with reboot
  sudo nano /etc/hostname
  sudo nano /etc/hosts
  sudo reboot
  
  # without reboot
  sudo hostname new-server-name-here
  sudo nano /etc/hostname
  sudo nano /etc/hosts
  
  # via hostnamectl
  hostnamectl set-hostname viveks-laptop
  sudo nano /etc/hosts
  ```

</details>

<details><summary><h4><a href="https://github.com/AV-ghub/PostgreSQL-Cloud-Solutions/blob/main/Linux/Ubuntu/001%20Installation.md#renew-dhcp-ip-address">Renew DHCP IP Address</a></h4></summary>
  
  ```
  :~$ ps fax | grep dhclient
     2574 pts/0    S+     0:00  |       \_ grep --color=auto dhclient
  
  :~$ ip addr
  
  :~$ sudo dhclient -r
  
  :~$ sudo dhclient -v
  Internet Systems Consortium DHCP Client 4.4.1
  Copyright 2004-2018 Internet Systems Consortium.
  All rights reserved.
  For info, please visit https://www.isc.org/software/dhcp/
  
  :~$ ip addr
  ```

</details>

<details><summary><h4>Сеть VMWare</h4></summary>

  ![](https://github.com/AV-ghub/PostgreSQL/blob/main/006%20Use%20cases/Installation/New%20cloud%20environment/CentOS%20based/Res/%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0%20CentOS%205.jpg)

</details>

<details><summary><h4>Repo install</h4></summary>

[Общая информация по репозиториям](https://serveradmin.ru/ustanovka-repozitoriya-epel-rpmforge-v-centos/)

Установка репозитория postgres
```
$ sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
$ sudo yum -y repolist
epo id    repo name  status
base/x86_64          CentOS-7 - Base                                                                                                     10,072
extras/x86_64        CentOS-7 - Extras                                                                                                      526
pgdg-common/7/x86_64 PostgreSQL common RPMs for RHEL / CentOS 7 - x86_64                                                                    545
pgdg12/7/x86_64      PostgreSQL 12 for RHEL / CentOS 7 - x86_64                                                                           1,377
pgdg13/7/x86_64      PostgreSQL 13 for RHEL / CentOS 7 - x86_64                                                                           1,140
pgdg14/7/x86_64      PostgreSQL 14 for RHEL / CentOS 7 - x86_64                                                                             900
pgdg15/7/x86_64      PostgreSQL 15 for RHEL / CentOS 7 - x86_64                                                                             613
updates/x86_64       CentOS-7 - Updates                                                                                                   6,173
```
Можно установить внешний репозиторий [EPEL](https://docs.fedoraproject.org/en-US/epel/)
```
# yum -y install epel-release

$ sudo yum repolist
...
epel/x86_64           Extra Packages for Enterprise Linux 7 - x86_64
...
```
```
$ sudo yum -y update
```

</details>

<details><summary><h4>PG installation</h4></summary>

#### Базовая установка
```
$ sudo yum -y install postgresql postgresql-server postgresql-contrib postgresql-libs

Installed:
  postgresql.x86_64 0:9.2.24-9.el7_9            postgresql-contrib.x86_64 0:9.2.24-9.el7_9            postgresql-libs.x86_64 0:9.2.24-9.el7_9            postgresql-server.x86_64 0:9.2.24-9.el7_9           

$ pg_
pg_archivecleanup  pg_config          pg_ctl             pg_dumpall         pg_resetxlog       pg_standby         pg_test_timing     
pg_basebackup      pg_controldata     pg_dump            pg_receivexlog     pg_restore         pg_test_fsync      
```
#### Репозиторий Postgres
```
$ sudo yum -y repolist
repo id           repo name           
base/x86_64       CentOS-7 - Base     
extras/x86_64     CentOS-7 - Extras   
updates/x86_64    CentOS-7 - Updates  

$ sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

=========================================================================================================================================================================================
 Package                                           Arch                                    Version                                      Repository                                       
=========================================================================================================================================================================================
Installing:
 pgdg-redhat-repo                                  noarch                                  42.0-38PGDG                                  /pgdg-redhat-repo-latest.noarch                  
=========================================================================================================================================================================================
Installed:
  pgdg-redhat-repo.noarch 0:42.0-38PGDG                                                                                                                                                  

$ sudo yum -y repolist

repo id               repo name                                             
base/x86_64           CentOS-7 - Base                                       
extras/x86_64         CentOS-7 - Extras                                     
pgdg-common/7/x86_64  PostgreSQL common RPMs for RHEL / CentOS 7 - x86_64   
pgdg12/7/x86_64       PostgreSQL 12 for RHEL / CentOS 7 - x86_64            
pgdg13/7/x86_64       PostgreSQL 13 for RHEL / CentOS 7 - x86_64            
pgdg14/7/x86_64       PostgreSQL 14 for RHEL / CentOS 7 - x86_64            
pgdg15/7/x86_64       PostgreSQL 15 for RHEL / CentOS 7 - x86_64            
updates/x86_64        CentOS-7 - Updates                                                                                                   6,173

$ sudo yum -y update
```
#### postgresql15-server в RHEL нет для CentOS 7
```
$ sudo yum -y install postgresql15-server
Error: Package: postgresql15-15.8-1PGDG.rhel7.x86_64 (pgdg15)
           Requires: libzstd >= 1.4.0
Error: Package: postgresql15-server-15.8-1PGDG.rhel7.x86_64 (pgdg15)
           Requires: libzstd.so.1()(64bit)
Error: Package: postgresql15-15.8-1PGDG.rhel7.x86_64 (pgdg15)
           Requires: libzstd.so.1()(64bit)
```
#### postgresql14 находится
```
$ sudo yum -y install postgresql14 postgresql14-server postgresql14-contrib postgresql14-libs
Installed:
  postgresql14.x86_64 0:14.13-2PGDG.rhel7       postgresql14-contrib.x86_64 0:14.13-2PGDG.rhel7       postgresql14-libs.x86_64 0:14.13-2PGDG.rhel7       postgresql14-server.x86_64 0:14.13-2PGDG.rhel7      

[ bin]$ sudo ./postgresql-14-setup initdb

[anisimov@cspg-dev001 bin]$ sudo systemctl enable --now postgresql-14
Created symlink from /etc/systemd/system/multi-user.target.wants/postgresql-14.service to /usr/lib/systemd/system/postgresql-14.service.

[ bin]$ sudo su postgres
bash-4.2$ cd
bash-4.2$ systemctl status postgresql*
● postgresql-14.service - PostgreSQL 14 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-14.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2024-09-25 09:04:22 MSK; 1min 32s ago
     Docs: https://www.postgresql.org/docs/14/static/
  Process: 28052 ExecStartPre=/usr/pgsql-14/bin/postgresql-14-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
 Main PID: 28059 (postmaster)
   CGroup: /system.slice/postgresql-14.service
           ├─28059 /usr/pgsql-14/bin/postmaster -D /var/lib/pgsql/14/data/
           ├─28060 postgres: logger 
           ├─28062 postgres: checkpointer 
           ├─28063 postgres: background writer 
           ├─28064 postgres: walwriter 
           ├─28065 postgres: autovacuum launcher 
           ├─28066 postgres: stats collector 
           └─28067 postgres: logical replication launcher 

		   
bash-4.2$ pg_ctl status -D /var/lib/pgsql/14/data/
pg_ctl: server is running (PID: 28059)
/usr/pgsql-14/bin/postgres "-D" "/var/lib/pgsql/14/data/"

bash-4.2$ psql
psql (14.13)
Type "help" for help.

postgres=# \l+
                                                                    List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   |  Size   | Tablespace |                Description                 
-----------+----------+----------+-------------+-------------+-----------------------+---------+------------+--------------------------------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 8777 kB | pg_default | default administrative connection database
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +| 8625 kB | pg_default | unmodifiable empty database
           |          |          |             |             | postgres=CTc/postgres |         |            | 
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +| 8625 kB | pg_default | default template for new databases
           |          |          |             |             | postgres=CTc/postgres |         |            | 
(3 rows)
```
#### Ставим внешний репозиторий
```
$ yum -y install epel-release
Installed:                                                                                                                                                                               
  epel-release.noarch 0:7-11                                                                                                                                                             
  
$ sudo yum -y update
Updated:
  epel-release.noarch 0:7-14
```
#### postgresql15 ставится
```
$ sudo yum -y install postgresql15 postgresql15-server postgresql15-contrib postgresql15-libs

Installed:
  postgresql15.x86_64 0:15.8-1PGDG.rhel7        postgresql15-contrib.x86_64 0:15.8-1PGDG.rhel7        postgresql15-libs.x86_64 0:15.8-1PGDG.rhel7        postgresql15-server.x86_64 0:15.8-1PGDG.rhel7       

Dependency Installed:
  libzstd.x86_64 0:1.5.5-1.el7
```
#### postgresql16 в CentOS 7 не поддерживается в принципе
```
[anisimov@cspg-dev001 ~]$ sudo yum -y install postgresql16 postgresql16-server postgresql16-contrib postgresql16-libs
No package postgresql16 available.
No package postgresql16-server available.
No package postgresql16-contrib available.
No package postgresql16-libs available.
```

</details>

<details><summary><h4><a href="https://stackoverflow.com/questions/37861262/create-multiple-postgres-instances-on-same-machine">PG start/stop</a></h4></summary>

### Теория
#### Create the clusters
```
$ initdb -D /path/to/datadb1
$ initdb -D /path/to/datadb2
```
#### Run the instances
```
$ pg_ctl -D /path/to/datadb1 -o "-p 5433" -l /path/to/logdb1 start
$ pg_ctl -D /path/to/datadb2 -o "-p 5434" -l /path/to/logdb2 start
```
#### Test streaming

Now you have two instances running on ports 5433 and 5434. Configuration files for them are in data dirs specified by initdb. Tweak them for streaming replication.
Your default installation remains untouched in port 5432.

#### Steps to create New Server Instance on PostgreSQL 9.5
On command prompt run:
```
initdb -D Instance_Directory_path -U username -W
(prompts for password)
```
Once the new Instance Directory is created. Run command prompt as Administrator
```
pg_ctl register -N service_name -D Instance_Directory_path -o "-p port_no"
```
After the service is registered, start server
```
pg_ctl start -D Instance_Directory_path -o "-p port_no"
```

### Практика
Все стопаем и правим у всех порты в postgresql.conf (/var/lib/pgsql/(14,15..n)/data)
Стартуем как описано выше со своего локального pg_ctl, напр:
```
$ /usr/pgsql-14/bin/pg_ctl -D /var/lib/pgsql/14/data start
```
Енейблим все сервисы
```
$ systemctl enable postgresql-14
```
Далее
```
$ systemctl status postgresql-14
```
может возвращать ошибки и неопределенные статусы. Делаем
```
$ sudo reboot
```
Проверяем (под sudo su postgres)
```
[root@cspg-dev001 bin]# sudo su postgres

bash-4.2$ cd /usr/pgsql-14/bin/
bash-4.2$ ./pg_ctl status -D /var/lib/pgsql/14/data/
pg_ctl: server is running (PID: 1315)
/usr/pgsql-14/bin/postgres "-D" "/var/lib/pgsql/14/data/"

bash-4.2$ ./psql -p 5434
psql (14.13)
postgres=# \q

bash-4.2$ systemctl status postgresql-14
● postgresql-14.service - PostgreSQL 14 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-14.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2024-09-25 15:38:29 MSK; 6min ago

bash-4.2$ /usr/pgsql-15/bin/pg_ctl status -D /var/lib/pgsql/15/data/
pg_ctl: server is running (PID: 1294)
/usr/pgsql-15/bin/postgres "-D" "/var/lib/pgsql/15/data/"

bash-4.2$ /usr/bin/pg_ctl status -D /var/lib/pgsql/data/
pg_ctl: server is running (PID: 1407)
/usr/bin/postgres "-D" "/var/lib/pgsql/data" "-p" "5432"

bash-4.2$ systemctl status postgresql-15
● postgresql-15.service - PostgreSQL 15 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-15.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2024-09-25 15:38:29 MSK; 8min ago

bash-4.2$ systemctl status postgresql
● postgresql.service - PostgreSQL database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2024-09-25 15:38:30 MSK; 8min ago

bash-4.2$ /usr/pgsql-15/bin/psql -p 5435
psql (15.8)
Type "help" for help.
postgres=# \q
```

</details>















