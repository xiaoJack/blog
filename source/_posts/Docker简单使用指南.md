---
title: Docker简单使用指南
date: 2019-08-09 12:43:45
tags: Docker
---

## 镜像相关
1. 拉取官方镜像
    ```
    docker pull redis 
    ```
2. 当前目录构建自己的镜像
    ```
    docker build -t nginx:v3 . 
    ```
3. 查看本地已有的镜像
    ```
    docker image ls
    ```
4. 删除本地镜像
    ```
    docker image rm alpine
    ```
5. 本地镜像打Tag
    ```
    docker tag  alpine  xiaogt/alpine
    ```
6. 推送镜像到远程仓库
    - https://hub.docker.com 以hub上为例子，要注册网站上面的账号
    - 运行docker login ,输入注册的用户名，密码
    - 推送镜像到远程仓库
        ```
        docker push xiaogt/alpine
        ```

## 容器相关
1. 运行本地容器
    ```
    docker run --name some-redis -d -p 6379:6379 redis
    ```
2. 停止一个容器运行（停止完不会删除）
    ```
    docker container stop some-redis
    ```
3. 停止后重新启动
    ```
    docker container start some-redis
    ```
4. 直接重启运行中的容器
    ```
    docker container restart some-redis
    ```
5. 查看当前运行的容器
    ```
    docker ps -as
    ```
6. 进入正在运行的容器
    ```
    docker exec -it f3232bc0215c bash
    ```
7. 查看所有已经创建的包括终止状态的容器
    ```
    docker container ls -a 
    ```
8. 删除单个终止运行的容器
    ```
    docker container rm gopub_gopub_1
    ```
9. 删除所有处于终止状态的容器
    ```
    docker container prune
    ```
    
## 相关问题总结
1. 删除本地镜像报错

    ```
     ➜ /Users/jack >docker image  ls
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    alpine              latest              4d90542f0623        2 weeks ago         5.58MB
    golang              latest              9fe4cdc1f173        3 weeks ago         774MB
    redis               latest              3c41ce05add9        3 weeks ago         95MB
    gopub_gopub         latest              91463c0bc27f        6 weeks ago         24.6MB
    mysql               latest              990386cbd5c0        7 weeks ago         443MB
    ➜ /Users/jack >docker image rm gopub_gopub
    Error response from daemon: conflict: unable to remove repository reference "gopub_gopub" (must force) - container d635b917b0b7 is using its referenced image 91463c0bc27f
    ➜ /Users/jack >
    ```

---
解决思路：有终止运行的容器没有删除，有关联关系还在，导致删除镜像的时候报错，先把停止状态容器删除了，就可以删除镜像
