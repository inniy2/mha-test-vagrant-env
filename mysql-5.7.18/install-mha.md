## Install MHA
- - - -  
1. Install dependency library
```bash
root> yum install -y perl perl-DBD-MySQL perl-Config-Tiny perl-Log-Dispatch perl-Parallel-ForkManager net-tools  mysql-community-client-5.7.23-1.el7.x86_64 

# Addition in vagrant
root> yum install -y perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker cpan
```

2. Download MHA
```bash
root> cd /tmp

root> wget https://downloads.mariadb.com/MHA/mha4mysql-manager-0.56.tar.gz

root> wget https://downloads.mariadb.com/MHA/mha4mysql-node-0.56.tar.gz

root> tar xvfPz mha4mysql-node-0.56.tar.gz

root> tar xvfPz mha4mysql-manager-0.56.tar.gz
```


3. Install MHA
```bash
root> cd mha4mysql-node-0.56

root> perl Makefile.PL

root> make && make install

root> cd ../mha4mysql-manager-0.56

root>  perl Makefile.PL

root>  make && make install
```


4. Create directories  
```bash
root> usermod -aG mysql mha

mha> mkdir ~/script

root> mkdir /var/log/masterha && mkdir /var/log/masterha/mha  

root> chown -R mha: /var/log/masterha
```


5. Create mhag1.cnf and masterha_default.cnf
```bash
mha> cat << EOF | tee ~/mhag1.cnf
[server default]
manager_workdir=/var/log/masterha/mha
manager_log=/var/log/masterha/mha.log
 
[server1]
hostname=192.168.33.21
candidate_master=1
[server2]
hostname=192.168.33.22
candidate_master=1
[server3]
hostname=192.168.33.23
no_master=1
EOF

mha> cat << EOF | tee ~/masterha_default.cnf
[server default]
ssh_user=mha
user=root
password=bigS3cret!!
repl_user=root
repl_password=bigS3cret!!
port=3301
ping_interval=3
master_ip_online_change_script=""
master_binlog_dir=/MYSQL/XXXXX/binlog
remote_workdir=/var/log/masterha/mha
secondary_check_script=""
master_ip_failover_script=""
shutdown_script=""
report_script=""
EOF
```

6. mha-manager.service  
```bash
root> cat << EOF | tee /usr/lib/systemd/system/mha-manager.service
[Unit]
Description=MHA service
After=syslog.target network-online.target

[Service]
User=mha
Group=mysql
Type=simple
PIDFile=/usr/local/mha/mha-manager.pid
KillMode=process
ExecStart=/usr/local/bin/masterha_manager --remove_dead_master_conf --global_conf=/usr/local/mha/masterha_default.cnf  --conf=/usr/local/mha/mhag1.cnf
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

