---
title: Beego Docker-compose部署报views找不到
date: 2019-08-10 08:42:43
tags: Go
---

### Beego Docker-compose部署报 Handler crashed with error can't find templatefile in the path:views/user/index.html

解决办法
docker-compose.yml 里面增加working_dir目录，类似cd到这个目录里面后在启动

```
working_dir: '/admin'
```

[参考官方文档](https://docs.docker.com/compose/compose-file/#domainname-hostname-ipc-mac_address-privileged-read_only-shm_size-stdin_open-tty-user-working_dir)
https://docs.docker.com/compose/compose-file/#domainname-hostname-ipc-mac_address-privileged-read_only-shm_size-stdin_open-tty-user-working_dir
