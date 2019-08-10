---
title: Golang生产环境中time包的zonefile.zip问题
date: 2019-08-10 08:41:21
tags: Go
---


docker 环境运行编译好的go文件
```
open /usr/local/go/lib/time/zoneinfo.zip: no such file or directory
```
解决办法 Dockerfile 里面增加COPY
```
COPY ./zoneinfo.zip /usr/local/go/lib/time/zoneinfo.zip
```
当然```./zoneinfo.zip``` 来自本地系统  ```/usr/local/go/lib/time/zoneinfo.zip```