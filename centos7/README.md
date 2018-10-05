## CentOS7
- - - -  
1. Vagrantfile
```
config.vm.box = "centos/7"
config.vm.network "private_network", ip: "192.168.33.10"

config.vm.provision "shell", inline: <<-SHELL
    sshd_vars="PasswordAuthentication PermitRootLogin PubkeyAuthentication"
    sed -i'' "s/#PasswordAuthentication/PasswordAuthentication/" /etc/ssh/sshd_config
    sed -i'' "s/#PermitRootLogin/PermitRootLogin/" /etc/ssh/sshd_config
    sed -i'' "s/#PubkeyAuthentication/PubkeyAuthentication/" /etc/ssh/sshd_config

    systemctl restart sshd
SHELL

config.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/id_rsa.pub"
config.vm.provision "shell", inline: "cat ~vagrant/.ssh/id_rsa.pub >> ~vagrant/.ssh/authorized_keys"
```


2. Contains action  
1) [Install repos](./install-repo.md)  
2) [Install users](./install-user.md)  
3) [Install selinux](./install-selinux.md)  


3. Output box
> my-centos7
