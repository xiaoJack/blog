---
title: Kubernetes-cluster-介绍
date: 2021-01-19 14:17:50
tags: Kubernetes
---


#  Kubernetes 组件  


<img src="https://raw.githubusercontent.com/kubernetes/kubernetes/master/logo/logo_with_border.png " width = "400" height = "400" alt="kubernetes" align=center />


Kubernetes 是一个自动化的容器(container)编排平台，它负责应用的部署、应用的弹性以及应用的管理，这些都是基于容器的


![image](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)



## Kubernetes架构图
![image](https://raw.githubusercontent.com/xiaoJack/blog-images/master/kubernetes.png)


![image](https://raw.githubusercontent.com/xiaoJack/blog-images/master/kubernetes-master.png)

## Master (Control Plane Components)
### apiserver 
Kubernetes 中所有的组件都会和 API Server 进行连接，组件与组件之间一般不进行独立的连接，都依赖于 API Server 进行消息的传送；
API Server 设计上考虑了水平伸缩，也就是说，它可通过部署多个实例进行伸缩。 你可以运行 API Server 的多个实例，并在这些实例之间平衡流量。

### scheduler 
控制平面组件，负责监视新创建的、未指定运行节点（node）的 Pods，选择节点让 Pod 在上面运行。

调度决策考虑的因素包括单个 Pod 和 Pod 集合的资源需求、硬件/软件/策略约束、亲和性和反亲和性规范、数据位置、工作负载间的干扰和最后时限。


### controller-manager 
从逻辑上讲，每个控制器都是一个单独的进程， 但是为了降低复杂性，它们都被编译到同一个可执行文件，并在一个进程中运行。

这些控制器包括:

节点控制器（Node Controller）: 负责在节点出现故障时进行通知和响应。
副本控制器（Replication Controller）: 负责为系统中的每个副本控制器对象维护正确数量的 Pod。
端点控制器（Endpoints Controller）: 填充端点(Endpoints)对象(即加入 Service 与 Pod)。
服务帐户和令牌控制器（Service Account & Token Controllers）: 为新的命名空间创建默认帐户和 API 访问令牌.

### etcd
etcd 是兼具一致性和高可用性的键值数据库，可以作为保存 Kubernetes 所有集群数据的后台数据库。




![image](https://raw.githubusercontent.com/xiaoJack/blog-images/master/kubernetes-node.png)
## Node 
### kubelet
一个在集群中每个节点（node）上运行的代理。 它保证容器（containers）都 运行在 Pod 中。

kubelet 接收一组通过各类机制提供给它的 PodSpecs，确保这些 PodSpecs 中描述的容器处于运行状态且健康。 kubelet 不会管理不是由 Kubernetes 创建的容器。


### kube-proxy 
kube-proxy 是集群中每个节点上运行的网络代理， 实现 Kubernetes 服务（Service） 概念的一部分。

kube-proxy 维护节点上的网络规则。这些网络规则允许从集群内部或外部的网络会话与 Pod 进行网络通信。

如果操作系统提供了数据包过滤层并可用的话，kube-proxy 会通过它来实现网络规则。否则， kube-proxy 仅转发流量本身。


### Container Runtime

Kubernetes 支持多个容器运行环境: Docker、 containerd、CRI-O 以及任何实现 Kubernetes CRI (容器运行环境接口)。




## 参考文献
- https://kubernetes.io/zh/docs/concepts/overview/components
- https://www.infoq.cn/article/knmavdo3jxs3qpkqtzbw

