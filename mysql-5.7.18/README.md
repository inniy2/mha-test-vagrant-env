## Vagrant  
- - - -  
1. Vagrantfile
```
config.vm.box = "my-centos7"
config.vm.network "private_network", ip: "192.168.33.21"
```

2. Contains action  
1) [Install mysql](./install-mysql.md)  
2) [Install mha](./install-mha.md)
3) [Install_proxysql](./install-proxysql.md)
4) [Install_keepalived](./install-keepalived.md)



3. Output box
> my-mysql5.7.18
