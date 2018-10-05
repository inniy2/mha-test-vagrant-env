## MySQL 5.7.18  
- - - -  
1. Set up repository
```bash
# Disable mariadb
root> yum-config-manager --disable  mariadb

# Disable mysql80-community
root> yum-config-manager --disable  mysql80-community

# Enable mysql57-community
root> yum-config-manager --enable  mysql57-community

# Check changed
root> yum repolist enabled | grep mysql

# Check all version of MySQL
root> yum --showduplicates list mysql-community | expand
```


2. Install MySQL 5.7.18 (On mysql nodes)  
```bash
root> yum install -y mysql-community-server-5.7.18
```


3. Setup data directories (On mysql nodes)  
```bash
root> mkdir /MYSQL && chown mysql:mysql /MYSQL

mysql> mkdir -p /MYSQL/cloud101/{data,innodb,binlog,innodb-log,log,work}
mysql> mkdir -p /MYSQL/cloud101/work/tmp
mysql> mkdir -p ~/{sock,script}
mysql> touch /MYSQL/cloud101/log/cloud101.err
```

4. Set my.cnf and mysqld.service (On mysql nodes)  
```bash
# Backup my.cnf
root> cp /etc/my.cnf /etc/my.back
```
Change following options in my.cnf: [See my.cnf](./my.cnf)  
> server_id                 = {setup DBQ ID}  
> innodb_buffer_pool_size   = {Total memory size / 2}  
> innodb_log_buffer_size    = {Depends on disk size}  
> read_only                 = {Depends on master or slave}  



```bash
# PID and Log file path setup
root> vim /usr/lib/systemd/system/mysqld.service
```
Add following options to mysqld:  
> --pid-file=/usr/local/mysql/sock/cloud101.pid --log-error=/MYSQL/cloud101/log/cloud101.err  


Change PIDFile:  
> PIDFile=/usr/local/mysql/sock/cloud101.pid  


```bash
root> systemctl daemon-reload
```

5. Start mysqld and get temporary password   (On mysql nodes)  
```bash
root> systemctl status mysqld
root> systemctl start  mysqld
root> systemctl status mysqld | grep 'temporary password'
```

6. Reset root password   (On mysql nodes)  
```bash
root> mysql -uroot -p 
```
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'XXXXXX';
```

7. Create .mylogin.cnf   (On mysql nodes)  
```bash
mysql> mysql_config_editor set \
--login-path=client \
--host=localhost \
--user=root \
--password
```

# THIS IS NOT FOR INSTALLATION  
8. Create default users (On mysql nodes)  
```sql
use mysql;

set sql_log_bin = 0;

create user 'XXX'@'%'identified by 'XXXX';
OR
create user 'XXX'@'%'identified by password 'XXXX';

grant all privileges on *.*  to 'XXX'@'%';

flush privileges;
```
