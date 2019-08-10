---
title: docker-compose启动mysql报错
date: 2019-08-10 08:23:20
tags: Docker
---


今天使用在本地mac环境docker-compose配置一个项目的时候，mysql5.7启不来了

查看日志的关键信息如下，好像数据目录有问题，想起前几天跑过mysql8.0的镜像，可能是数据文件冲突导致
```
mysql_1                | 2017-10-04T04:21:57.641673Z 1 [ERROR] [FATAL] InnoDB: Table flags are 0x4800 in the data dictionary but the flags in file mysql.ibd are 0x800!

```

删除所有的本地docker volume
```
sudo docker volume rm $(sudo docker volume ls -qf dangling=true)
```


