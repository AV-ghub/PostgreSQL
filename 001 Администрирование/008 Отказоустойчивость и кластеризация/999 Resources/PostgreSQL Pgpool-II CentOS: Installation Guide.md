[src](https://www.heatware.net/postgresql/postgres-pgpool-centos-setup-install/)
### Установка
```bash
sudo yum update
```
```bash
sudo yum install -y epel-release
sudo yum install -y pgpool-II-extensions
```
### Настройка
```bash
sudo nano /etc/pgpool-II/pgpool.conf
```
***
* listen_addresses: Set the IP address or hostname on which Pgpool-II will listen for incoming connections.
* backend_hostname: Specify the hostnames or IP addresses of your PostgreSQL servers.
* backend_port: Define the port numbers on which the PostgreSQL servers are listening.
* backend_weight: Assign weights to the PostgreSQL servers to control load distribution. Make the necessary changes to these parameters and save the file.
***
* num_init_children: Sets the number of child processes that Pgpool-II will create to handle incoming connections.
* load_balance_mode: Determines the load balancing behavior, such as using a round-robin algorithm or weighted load balancing.
* replication_mode: Specifies the replication mode, including streaming replication or logical replication.
* health_check_period: Defines the interval for health checks to monitor the status of PostgreSQL servers.
***
### Запуск
```bash
sudo systemctl start pgpool-II
# автозапуск на старте
sudo systemctl enable pgpool-II
```
### Установка админки
Зависимости
```bash
sudo yum install -y httpd php php-pgsql
```
Апач
```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```
Pgpool Admin package
eg http://www.pgpool.net/mediawiki/index.php/Downloads
```bash
wget http://www.pgpool.net/download.php?f=pgpoolAdmin-4.2.2.tar.gz -O pgpoolAdmin.tar.gz
```
Разархивируем
```bash
tar xvfz pgpoolAdmin.tar.gz
```
Перемещаем в апач
```bash
sudo mv pgpoolAdmin-4.2.2 /var/www/html/pgpoolAdmin
```
Права
```bash
sudo chown -R apache:apache /var/www/html/pgpoolAdmin
```
Создаем новый файл конфигурации
```bash
sudo nano /etc/httpd/conf.d/pgpoolAdmin.conf
```
Добавляем туда примерно следующее
```html
   <VirtualHost *:80>
       ServerAdmin admin@example.com
       DocumentRoot /var/www/html/pgpoolAdmin
       ServerName your_domain_or_IP
       ErrorLog /var/log/httpd/pgpoolAdmin-error.log
       CustomLog /var/log/httpd/pgpoolAdmin-access.log combined
       <Directory /var/www/html/pgpoolAdmin>
           Options FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>
   </VirtualHost>
```
> Replace your_domain_or_IP with your actual domain name or IP address.      

Рестартим апач
```bash
sudo systemctl restart httpd
```
Стучимся в Pgpool Admin
```html
http://your_domain_or_IP/pgpoolAdmin
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```

