---
title: stackstorm
date: 2016-11-29 18:24:08
categories: tools
tags: stackstorm
---

stackstorm的使用.

<!-- more -->
# 安装

[st2的服务]
第一次安装的时候，执行：
重装的时候，删掉/etc/st2/keys/datastore_key.json
然后执行：
./st2bootstrap-el7.sh --user=st2admin --password=Ch@ngeMe

----添加一个extra的值，作为判断。

systemctl list-unit-files | grep st2
st2actionrunner.service                enabled 
st2actionrunner@.service               disabled
st2api.service                         enabled 
st2auth.service                        enabled 
st2garbagecollector.service            enabled 
st2notifier.service                    enabled 
st2resultstracker.service              enabled 
st2rulesengine.service                 enabled 
st2sensorcontainer.service             enabled 
st2stream.service                      enabled 

# 使用
查看packs的所有操作：  
st2 action list --pack packs

# 与ansible集成
[安装pack](https://docs.stackstorm.com/packs.html#getting-a-pack)  
安装ansible的pack  
st2 run packs.install packs=ansible repo_url=https://github.com/StackStorm/st2contrib.git   
如果报错：fatal: destination path '/root/st2contrib' already exists and is not an empty directory.  
先删掉路径'/root/st2contrib' 

```
st2 run packs.download packs=ansible
st2 run packs.setup_virtualenv packs=ansible
st2 run packs.load register=all
st2 run packs.restart_component servicename=st2sensorcontainer


st2 action list --pack=ansible
st2 sensor list --pack=ansible
st2 trigger list --pack=ansible


st2 run packs.deploy packs=ansible



git clone https://github.com/StackStorm/st2contrib.git
cd st2contrib
git remote -v
git remote set-url origin git@git.elenet.me:cloud/st2contrib.git
git remote -v


st2 run packs.install packs=ansible repo_url=https://git.elenet.me/cloud/st2contrib.git 


st2 run packs.install packs=ansible

```
## 通过st2调用ansible
首先安装ansible  
```
yum -y install ansible
```

```
st2 run ansible.command_local args='echo $TERM'

st2 run ansible.command connection=local inventory_file='127.0.0.1,' hosts=all args='echo $SHELL'

st2 run ansible.command inventory_file='10.12.10.57,' hosts=all args='ip a'



# 执行一个playbook
st2 --debug  run ansible.playbook playbook=/data/mesos_ops/playbooks/agent.yml inventory_file=/data/mesos_ops/inventories/hosts extra_vars='@/data/mesos_ops/extra_vars/all.yml'

st2 --debug  run ansible.playbook playbook=/data/mesos_ops/playbooks/agent.yml inventory_file='10.12.10.57  DOCKER_FIXED_CIDR="10.12.10.208/29",' extra_vars='@/data/mesos_ops/extra_vars/all.yml'


```










