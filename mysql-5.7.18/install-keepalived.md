## Install keepalived  
- - - -  
1. Install dependent packages  
```bash
root> yum install -y psmisc curl gcc openssl-devel libnl3-devel net-snmp-devel wget libnfnetlink-devel iptables-devel ipset-devel file-devel glib2-devel json-c-devel
```


2. Download keepalived  
```bash
root> cd /tmp

root> wget --no-check-certificate http://www.keepalived.org/software/keepalived-2.0.5.tar.gz 

root> tar xvfPz keepalived-2.0.5.tar.gz
```


3. Install keepalived  
```bash
root> cd keepalived-2.0.5

root> ./configure

root> make

root> make install
```


3. Create keepalived-state-listener
```bash
root> cat << EOF | tee /usr/local/bin/keepalived-state-listener
#!/usr/bin/env python

import os
import sys
import time
import subprocess

pid = str(os.getpid())
pidfile = "/tmp/keepalived.state.listener.pid"

if os.path.isfile(pidfile):
  #print "%s already exists, exiting" % pidfile
  sys.exit()
file(pidfile, 'w').write(pid)

state = "NT"
try:
  for x in range(5):
    f = open('/tmp/keepalived.state.flag', "r")
    lines = f.readlines()
    for line in lines:
      if line.startswith("MASTER"):
        #print "%s " % line
        #print "%s progessing ... MASTER"
        state = "MASTER"
      elif line.startswith("BACKUP"):
        #print "%s " % line
        #print "%s progessing ... BACKUP"
        state = "BACKUP"
      else:
        #print "%s " % line
        #print "%s progessing ... ELSE"
        state = "ELSE"
    f.close()
    time.sleep(10)
finally:
   os.unlink(pidfile)
   #print "%s is final state " % state
   if state == "MASTER":
     print " Start mha-manager service"
     subprocess.call("systemctl start mha-manager.service", shell=True)
   elif state == "BACKUP":
     print " Stop mha-manager service"
     subprocess.call("systemctl stop mha-manager.service", shell=True)
   else:
     print ""
EOF

root> chmod 755 /usr/local/bin/keepalived-state-listener

```


4. Create notify_state.sh  
```bash
root> cat << EOF | tee /usr/local/bin/notify_state.sh
#!/bin/bash
TYPE=$1
NAME=$2
STATE=$3
case $STATE in
        "MASTER") 
                  #touch /tmp/master-is-notified.log.`date +%Y-%m-%d_%H_%M_%S`
                  rm -rf /tmp/keeaplived.state.flag && echo "MASTER" > /tmp/keepalived.state.flag
                  /usr/local/bin/keepalived-state-listener
                  exit 1
                  ;;
        "BACKUP") 
                  #touch /tmp/backup-is-notified.log.`date +%Y-%m-%d_%H_%M_%S`
                  rm -rf /tmp/keeaplived.state.flag && echo "BACKUP" > /tmp/keepalived.state.flag
                  /usr/local/bin/keepalived-state-listener
                  exit 1
                  ;;
        "FAULT")  
                  #touch /tmp/fault-is-notified.log.`date +%Y-%m-%d_%H_%M_%S`
                  rm -rf /tmp/keeaplived.state.flag && echo "FAULT" > /tmp/keepalived.state.flag
                  exit 1
                  ;;
        *)       
                  #touch /tmp/nothing-is-notified.log.`date +%Y-%m-%d_%H_%M_%S`
                  rm -rf /tmp/keeaplived.state.flag && echo "NT" > /tmp/keepalived.state.flag
                  exit 1
                  ;;
esac

EOF 

root> chmod 755 /usr/local/bin/notify_state.sh

```

5. Create keepalived.conf
```bash
root> mkdir /etc/keepalived

root> cat << EOF | tee /etc/keepalived/keepalived.conf
global_defs {
  # Keepalived process identifier
  #lvs_id proxy_HA
  #router_id LVS
  #enable_script_security
}
# Script used to check if Proxy is running
vrrp_script check_proxy {
  script "/bin/killall -0 proxysql"
  interval 2
  weight 2
}
# Virtual interface
# The priority specifies the order in which the assigned interface to take over in a failover
vrrp_instance VI_01 {
  state MASTER
  interface eth1
  virtual_router_id 51
  priority 101
 
  # The virtual ip address shared between the two loadbalancers
  virtual_ipaddress {
    192.168.33.15 dev eth1
  }
  track_script {
    check_proxy
  }

  notify /usr/local/bin/notify_state.sh
}
EOF
```

