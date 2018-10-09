## ProxySQL setup  
- - - -  
1. Create MySQL user  
```
sql> use mysql;

sql> create user 'root_RW'@'%' identified by 'bigS3cret!!';
sql> create user 'root_W'@'%'  identified by 'bigS3cret!!';
sql> create user 'root_R'@'%'  identified by 'bigS3cret!!';
sql> create user 'monitor'@'%' identified by 'bigS3cret!!';

sql> grant all privileges on *.* to 'root_RW'@'%' with grant option;
sql> grant all privileges on *.* to 'root_W'@'%'  with grant option;
sql> grant all privileges on *.* to 'root_R'@'%'  with grant option;
sql> grant all privileges on *.* to 'monitor'@'%' with grant option;
```


2. Configuration init  

```bash
root> cat << EOF | tee /etc/proxysql.cnf
#file proxysql.cfg

# This config file is parsed using libconfig , and its grammar is described in:
# http://www.hyperrealm.com/libconfig/libconfig_manual.html#Configuration-File-Grammar
# Grammar is also copied at the end of this file



datadir="/var/lib/proxysql"

admin_variables=
{
	admin_credentials="admin:admin"
#	mysql_ifaces="127.0.0.1:6032;/tmp/proxysql_admin.sock"
	mysql_ifaces="0.0.0.0:6032"
#	refresh_interval=2000
#	debug=true
}

mysql_variables=
{
	threads=4
	max_connections=2048
	default_query_delay=0
	default_query_timeout=36000000
	have_compress=true
	poll_timeout=2000
#	interfaces="0.0.0.0:6033;/tmp/proxysql.sock"
	interfaces="0.0.0.0:6033"
	default_schema="mysql"
	stacksize=1048576
	server_version="5.5.30"
	connect_timeout_server=3000
# make sure to configure monitor username and password
# https://github.com/sysown/proxysql/wiki/Global-variables#mysql-monitor_username-mysql-monitor_password
	monitor_username="monitor"
	monitor_password="bigS3cret!!"
	monitor_history=600000
	monitor_connect_interval=60000
	monitor_ping_interval=10000
	monitor_read_only_interval=1500
	monitor_read_only_timeout=500
	ping_interval_server_msec=120000
	ping_timeout_server=500
	commands_stats=true
	sessions_sort=true
	connect_retries_on_failure=10
}


# defines all the MySQL servers
mysql_servers =
(
	{
		address = "192.168.33.21" # no default, required . If port is 0 , address is interpred as a Unix Socket Domain
		port = 3301           # no default, required . If port is 0 , address is interpred as a Unix Socket Domain
		hostgroup = 600       # no default, required
		status = "ONLINE"     # default: ONLINE
		weight = 1000         # default: 1
		compression = 0       # default: 0
   max_replication_lag = 0  # default 0 . If greater than 0 and replication lag passes such threshold, the server is shunned
	},
 
	{
		address = "192.168.33.22" # no default, required . If port is 0 , address is interpred as a Unix Socket Domain
		port = 3301           # no default, required . If port is 0 , address is interpred as a Unix Socket Domain
		hostgroup = 601       # no default, required
		status = "ONLINE"     # default: ONLINE
		weight = 1000         # default: 1
		compression = 0       # default: 0
   max_replication_lag = 10  # default 0 . If greater than 0 and replication lag passes such threshold, the server is shunned
	},
        {
		address = "192.168.33.23" # no default, required . If port is 0 , address is interpred as a Unix Socket Domain
		port = 3301           # no default, required . If port is 0 , address is interpred as a Unix Socket Domain
		hostgroup = 601       # no default, required
		status = "ONLINE"     # default: ONLINE
		weight = 1000         # default: 1
		compression = 0       # default: 0
   max_replication_lag = 10  # default 0 . If greater than 0 and replication lag passes such threshold, the server is shunned
	}

#	{
#		address = "/var/lib/mysql/mysql.sock"
#		port = 0
#		hostgroup = 0
#	},
#	{
#		address="127.0.0.1"
#		port=21891
#		hostgroup=0
#		max_connections=200
#	},
#	{ address="127.0.0.2" , port=3306 , hostgroup=0, max_connections=5 },
#	{ address="127.0.0.1" , port=21892 , hostgroup=1 },
#	{ address="127.0.0.1" , port=21893 , hostgroup=1 }
#	{ address="127.0.0.2" , port=3306 , hostgroup=1 },
#	{ address="127.0.0.3" , port=3306 , hostgroup=1 },
#	{ address="127.0.0.4" , port=3306 , hostgroup=1 },
#	{ address="/var/lib/mysql/mysql.sock" , port=0 , hostgroup=1 }
)


# defines all the MySQL users
mysql_users:
(
	{
		username = "root_RW" # no default , required
		password = "bigS3cret!!" # default: ''
		default_hostgroup = 600 # default: 0
		active = 1            # default: 1
	},
  {
		username = "root_W" # no default , required
		password = "bigS3cret!!" # default: ''
		default_hostgroup = 600 # default: 0
		active = 1            # default: 1
	},
  {
		username = "root_R" # no default , required
		password = "bigS3cret!!" # default: ''
		default_hostgroup = 601 # default: 0
		active = 1            # default: 1
	}
#	{
#		username = "root"
#		password = ""
#		default_hostgroup = 0
#		max_connections=1000
#		default_schema="test"
#		active = 1
#	},
#	{ username = "user1" , password = "password" , default_hostgroup = 0 , active = 0 }
)



#defines MySQL Query Rules
mysql_query_rules:
(
	{
		rule_id=1
		active=1
		match_pattern="^SELECT .* FOR UPDATE$"
		destination_hostgroup=600
		apply=1
	},
	{
		rule_id=2
		active=1
		match_pattern="^SELECT"
		destination_hostgroup=601
		apply=1
	}
)

scheduler=
(
#  {
#    id=1
#    active=0
#    interval_ms=10000
#    filename="/var/lib/proxysql/proxysql_galera_checker.sh"
#    arg1="0"
#    arg2="0"
#    arg3="0"
#    arg4="1"
#    arg5="/var/lib/proxysql/proxysql_galera_checker.log"
#  }
)


mysql_replication_hostgroups=
(
        {
                writer_hostgroup=600
                reader_hostgroup=601
                comment="test repl 1"
        }
#       {
#                writer_hostgroup=50
#                reader_hostgroup=60
#                comment="test repl 2"
#        }
)




# http://www.hyperrealm.com/libconfig/libconfig_manual.html#Configuration-File-Grammar
#
# Below is the BNF grammar for configuration files. Comments and include directives are not part of the grammar, so they are not included here. 
#
# configuration = setting-list | empty
#
# setting-list = setting | setting-list setting
#     
# setting = name (":" | "=") value (";" | "," | empty)
#     
# value = scalar-value | array | list | group
#     
# value-list = value | value-list "," value
#     
# scalar-value = boolean | integer | integer64 | hex | hex64 | float
#                | string
#     
# scalar-value-list = scalar-value | scalar-value-list "," scalar-value
#     
# array = "[" (scalar-value-list | empty) "]"
#     
# list = "(" (value-list | empty) ")"
#     
# group = "{" (setting-list | empty) "}"
#     
# empty =
# ###########

EOF
```

```

admin> LOAD MYSQL SERVERS      FROM CONFIG;  --( Include mysql_replication_hostgroups )
admin> LOAD MYSQL USERS        FROM CONFIG;
admin> LOAD MYSQL QUERY RULES  FROM CONFIG;
admin> LOAD MYSQL VARIABLES    FROM CONFIG;


admin> LOAD MYSQL SERVERS      TO RUNTIME;
admin> LOAD MYSQL USERS        TO RUNTIME;
admin> LOAD MYSQL QUERY RULES  TO RUNTIME;
admin> LOAD MYSQL VARIABLES    TO RUNTIME;

admin> SAVE MYSQL SERVERS      TO DISK;
admin> SAVE MYSQL USERS        TO DISK;
admin> SAVE MYSQL QUERY RULES  TO DISK;
admin> SAVE MYSQL VARIABLES    TO DISK;

admin> SELECT * FROM monitor.mysql_server_read_only_log ORDER BY time_start_us DESC LIMIT 10;

mysql -h192.168.33.12 -uroot_RW -pbigS3cret\!\! -P6033 -e'show variables like "%server_id%"'
mysql -h192.168.33.12 -uroot_W  -pbigS3cret\!\! -P6033 -e'show variables like "%server_id%"'
mysql -h192.168.33.12 -uroot_R  -pbigS3cret\!\! -P6033 -e'show variables like "%server_id%"'

```

3. SQL init  
```
admin>
admin>
admin>
```


4. Keepalived (With MHA)  
```bash
root> systemctl start keepalived
```


5. Rest MHA  
```bash
root> rm -rf  /var/log/masterha/mha/*

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

root> systemctl start mha-manager
```










