## Install repos  
- - - -  
1. wget, vim 
```bash
sudo yum install -y wget vim
```

2. epel
```bash
cd /tmp

sudo wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

sudo rpm -ivh /tmp/epel-release-latest-7.noarch.rpm 
```


3. percona
```bash
sudo yum install -y http://www.percona.com/downloads/percona-release/redhat/0.1-6/percona-release-0.1-6.noarch.rpm

sudo yum-config-manager --disable percona-release-noarch percona-release-x86_64
```


4. ProxySQL
```bash
root> cat <<EOF | tee /etc/yum.repos.d/proxysql.repo
[proxysql_repo]
name= ProxySQL YUM repository
baseurl=http://repo.proxysql.com/ProxySQL/proxysql-1.4.x/centos/\$releasever
gpgcheck=1
gpgkey=http://repo.proxysql.com/ProxySQL/repo_pub_key
EOF

root> yum-config-manager --disable proxysql_repo
```


5. MariaDB
```bash
root> cat << EOF | tee /etc/yum.repos.d/MariaDB.repo
# MariaDB 10.3 CentOS repository list - created 2018-10-03 05:55 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.3/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
EOF

root> yum-config-manager --disable mariadb
```

6. MySQL
```bash
cd /tmp 

sudo wget https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm

sudo yum -y localinstall mysql80-community-release-el7-1.noarch.rpm

sudo yum-config-manager --disable mysql80-community

sudo yum-config-manager --enable  mysql57-community
```

7. Check repolist
```bash
sudo yum repolist
```
