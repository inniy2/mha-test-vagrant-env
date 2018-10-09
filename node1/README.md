## Sysbench  
- - - -  
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
