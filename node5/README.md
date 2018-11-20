## Slave 1 setup  
- - - -  
1. Config
```bash
root> rm -rf  /MYSQL/XXXXX/data/auto.cnf
root> vim /etc/my.cnf

server_id = 2
read_only

root> systemctl restart mysqld
```

2. Setup replication  
```
sql> use mysql;
sql> CHANGE MASTER TO \
       MASTER_HOST='192.168.33.21' ,\
       MASTER_USER='root' ,\
       MASTER_PASSWORD='bigS3cret!!' ,\
       MASTER_PORT=3301  ,\
       MASTER_LOG_FILE='binarylog.000004' ,\
       MASTER_LOG_POS=154  ;

sql> start slave;
```

