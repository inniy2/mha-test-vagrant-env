## Install users  
- - - -  
1. MySQL user
```bash
root> groupadd mysql -g 55000
root> useradd mysql -u 55000 -g 55000 -G mysql -d /usr/local/mysql -m -s /bin/bash -c 'mysql user'
```


1. Ansible user
```bash
root> groupadd ansible -g 57000
root> useradd  ansible -u 57000 -g 57000 -G ansible -d /usr/local/ansible -m -s /bin/bash -c 'ansible user'

root> cat << EOF | tee /etc/sudoers.d/ansible
%ansible ALL=(ALL) NOPASSWD: ALL
EOF
```


2. MHA user
```bash
root> groupadd mha -g 56000
root> useradd  mha -u 56000 -g 56000 -G mha -d /usr/local/mha -m -s /bin/bash -c 'MHA user'

mha>  ssh-keygen -f ~/.ssh/id_rsa -C mha
mha>  cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys
mha>  chmod 755 ~/.ssh && chmod 600 ~/.ssh/*

