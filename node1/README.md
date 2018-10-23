## Sysbench  
- - - -  
1. Sysbench 0.4.12.14  
```bash
root> yum -y groupinstall "Development Tools"

root> yum -y install mysql-community-client.x86_64 mysql-community-devel.x86_64



root> export mysql_library_path=/usr/lib64/mysql

root> export mysql_include_directory_path=/usr/include/mysql

root> export LD_LIBRARY_PATH=$mysql_library_path:$PATH



root> cd /tmp

root> wget https://downloads.mysql.com/source/sysbench-0.4.12.14.tar.gz

root> tar xvfPz sysbench-0.4.12.14.tar.gz

root> cd sysbench-0.4.12.14

root> ./configure --prefix=/usr --with-mysql-includes=$mysql_include_directory_path --with-mysql-libs=$mysql_library_path


root> sysbench --test=oltp --num-threads=1 --max-requests=10000 --max-time=60 --db-driver=mysql  --mysql-host='192.168.33.15' --mysql-user=root_RW --mysql-password='bigS3cret!!' --mysql-port=6033 prepare


root> sysbench --test=oltp --num-threads=1 --max-requests=10000 --max-time=60 --db-driver=mysql  --mysql-host='192.168.33.15' --mysql-user=root_RW --mysql-password='bigS3cret!!' --mysql-port=6033 run
```


2. Use sysbench 1.0.15  
It supports lua scripts
```bash
root> curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.rpm.sh | sudo bash
root> sudo yum -y install sysbench
```


3. Choose one of the default bundle lua script  
```
ls -al /usr/share/sysbench/

bulk_insert.lua
oltp_common.lua
oltp_delete.lua
oltp_insert.lua
oltp_point_select.lua
oltp_read_only.lua
oltp_read_write.lua
oltp_update_index.lua
oltp_update_non_index.lua
oltp_write_only.lua
select_random_points.lua
select_random_ranges.lua
```


4. New prepare (read_only multiplexing)  
if multiplexing is being used, handshake error will occurs.  
sysbench can not ignore handshake error, therefore, it will stop.  
```bash
root> sysbench oltp_read_only \
--mysql-db=sbtest \
--num-threads=1 \
--events=10000 \
--time=6000 \
--report-interval=1 \
--db-driver=mysql  \
--mysql-host='192.168.33.15' \
--mysql-user=root_R \
--mysql-password='bigS3cret!!' \
--mysql-port=6033 \
prepare
```


5. New run (read_only multiplexing)  
if multiplexing is being used, handshake error will occurs.  
sysbench can not ignore handshake error, therefore, it will stop.  
```bash
root> sysbench oltp_read_only \
--mysql-db=sbtest \
--num-threads=1 \
--events=10000 \
--time=6000 \
--report-interval=1 \
--db-driver=mysql  \
--mysql-host='192.168.33.15' \
--mysql-user=root_R \
--mysql-password='bigS3cret!!' \
--mysql-port=6033 \
run
```


6. New run (read_write)  
Multiplexing is not used.  
Sysbench will ignore 2013 error.  
```bash
root> sysbench oltp_read_write \
--mysql-db=sbtest \
--num-threads=1 \
--events=10000 \
--time=6000 \
--report-interval=1 \
--db-driver=mysql  \
--mysql-host='192.168.33.15' \
--mysql-user=root_RW \
--mysql-password='bigS3cret!!' \
--mysql-port=6033 \
--mysql-ignore-errors=all \
run 
```

7. New run (read_only)  
Multiplexing is not used.  
Sysbench will ignore 2013 error.  
```bash
root> sysbench oltp_point_select \
--mysql-db=sbtest \
--num-threads=1 \
--events=1000000 \
--time=6000 \
--report-interval=1 \
--db-driver=mysql  \
--mysql-host='192.168.33.15' \
--mysql-user=root_R \
--mysql-password='bigS3cret!!' \
--mysql-port=6033 \
--mysql-ignore-errors=all \
run 
```


