## Master 1 setup  
- - - -  
1. Reset MHA  
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

