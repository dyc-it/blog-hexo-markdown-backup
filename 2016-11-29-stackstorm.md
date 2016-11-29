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

# ansible集成
https://docs.stackstorm.com/packs.html#getting-a-pack
安装ansible的pack  
st2 run packs.install packs=ansible repo_url=https://github.com/StackStorm/st2contrib.git   
如果报错：fatal: destination path '/root/st2contrib' already exists and is not an empty directory.  
先删掉路径'/root/st2contrib' 






