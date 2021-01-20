---
title: Kubernetes-核心概念
date: 2021-01-20 14:10:45
tags: Kubernetes
---


# Kubernetes-核心概念

## Pod
Pod 是 Kubernetes 的一个最小调度以及资源单元。用户可以通过 Kubernetes 的 Pod API 生产一个 Pod，让 Kubernetes 对这个 Pod 进行调度，也就是把它放在某一个 Kubernetes 管理的节点上运行起来。一个 Pod 简单来说是对一组容器的抽象，它里面会包含一个或多个容器。

在 Pod 里面，我们也可以去定义容器所需要运行的方式。比如说运行容器的 Command，以及运行容器的环境变量等等。Pod 这个抽象也给这些容器提供了一个共享的运行环境，它们会共享同一个网络环境，这些容器可以用 localhost 来进行直接的连接。而 Pod 与 Pod 之间，是互相有 隔离的。


## Deployment
Deployment 是在 Pod 这个抽象上更为上层的一个抽象，它可以定义一组 Pod 的副本数目、以及这个 Pod 的版本。一般大家用 Deployment 这个抽象来做应用的真正的管理，而 Pod 是组成 Deployment 最小的单元。


Kubernetes 是通过 Controller，也就是我们刚才提到的控制器去维护 Deployment 中 Pod 的数目，它也会去帮助 Deployment 自动恢复失败的 Pod。

## Service
Service 提供了一个或者多个 Pod 实例的稳定访问地址。

实现 Service 有多种方式，Kubernetes 支持 Cluster IP，上面我们讲过的 kuber-proxy 的组网，它也支持 nodePort、 LoadBalancer 等其他的一些访问的能力。

## Ingress
Ingress Controller是一个统称，并不是只有一个，有如下这些：
- Ingress NGINX: Kubernetes 官方维护的方案，也是本次安装使用的 Controller。
- F5 BIG-IP Controller: F5 所开发的 Controller，它能够让管理员通过 CLI 或 API 让 Kubernetes 与 OpenShift 管理 F5 BIG-IP 设备。
- Ingress Kong: 著名的开源 API Gateway 方案所维护的 Kubernetes Ingress Controller。
- Traefik: 是一套开源的 HTTP 反向代理与负载均衡器，而它也支援了 Ingress。
- Voyager: 一套以 HAProxy 为底的 Ingress Controller。

## Namespace
Namespace 是用来做一个集群内部的逻辑隔离的，它包括鉴权、资源管理等。Kubernetes 的每个资源，比如刚才讲的 Pod、Deployment、Service 都属于一个 Namespace，同一个 Namespace 中的资源需要命名的唯一性，不同的 Namespace 中的资源可以重名。








## 参考文献
- https://www.infoq.cn/article/knmavdo3jxs3qpkqtzbw
- https://www.servicemesher.com/blog/kubernetes-ingress-controller-deployment-and-ha/
