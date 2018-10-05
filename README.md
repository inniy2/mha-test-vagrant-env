# mha-test-vagrant-env
- - - -  
### 1. Example of Vagrant up  
```bash
# cd $git_home_vagrant/mha-test-vagrant-env/X

export target_vagrant=$PWD

cd $target_vagrant && vagrant up
```


### 2. Connection example  
```bash
sed -i '' '/192.168.33/d' ~/.ssh/known_hosts

ssh vagrant@192.168.33.X

```


### 3. Example of Vagrant destroy
```bash
cd $target_vagrant && vagrant destroy -f
```


### 4. Box operations
#### 1) Export Box
```bash
cd $target_vagrant && vagrant package --output my-centos7.box
```

#### 2) Add Box
```bash
# Check Box list
vagrant box list

# Add Box to the list
vagrant box add my-centos7  my-centos7.box
```

