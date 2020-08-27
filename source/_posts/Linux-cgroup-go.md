---
title: Linux cgroup go
date: 2020-08-27 09:38:19
tags: [Go,Docker]
---


## CGroup 介绍
Linux CGroup全称Linux Control Group， 是Linux内核的一个功能，用来限制，控制与分离一个进程组群的资源（如CPU、内存、磁盘输入输出等）。这个项目最早是由Google的工程师在2006年发起（主要是Paul Menage和Rohit Seth），最早的名称为进程容器（process containers）。在2007年时，因为在Linux内核中，容器（container）这个名词太过广泛，为避免混乱，被重命名为cgroup，并且被合并到2.6.24版的内核中去



## 概念及原理
cgroups子系统
cgroups为每种可以控制的资源定义了一个子系统。典型的子系统介绍如下：

1. cpu 子系统，主要限制进程的 cpu 使用率
2. cpuacct 子系统，可以统计 cgroups 中的进程的 cpu 使用报告
3. cpuset 子系统，可以为 cgroups 中的进程分配单独的 cpu 节点或者内存节点
4. memory 子系统，可以限制进程的 memory 使用量
5. blkio 子系统，可以限制进程的块设备 io
6. devices 子系统，可以控制进程能够访问某些设备
7. net_cls 子系统，可以标记 cgroups 中进程的网络数据包，然后可以使用 tc 模块（traffic control）对数据包进行控制
8. freezer 子系统，可以挂起或者恢复 cgroups 中的进程
9. ns 子系统，可以使不同 cgroups 下面的进程使用不同的 namespace



## 使用方法
### 查看系统已有cgroup列表两种方法

```
[root@localhost cgroup]# mount -t cgroup
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,freezer)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,devices)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,net_cls,net_prio)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,pids)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,blkio)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,memory)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpu,cpuacct)
cgroup on /sys/fs/cgroup/rdma type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,rdma)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpuset)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,perf_event)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,hugetlb)

[root@localhost cgroup]# lssubsys  -m
cpuset /sys/fs/cgroup/cpuset
cpu,cpuacct /sys/fs/cgroup/cpu,cpuacct
blkio /sys/fs/cgroup/blkio
memory /sys/fs/cgroup/memory
devices /sys/fs/cgroup/devices
freezer /sys/fs/cgroup/freezer
net_cls,net_prio /sys/fs/cgroup/net_cls,net_prio
perf_event /sys/fs/cgroup/perf_event
hugetlb /sys/fs/cgroup/hugetlb
pids /sys/fs/cgroup/pids
rdma /sys/fs/cgroup/rdma
```

### 查看系统开启了哪些cgroup

```
[root@localhost cgroup]# cat /proc/cgroups
#subsys_name	hierarchy	num_cgroups	enabled
cpuset	            8	        1	    1
cpu	                7	        48	    1
cpuacct	            7	        48	    1
blkio	            10	        48	    1
memory	            2	        90	    1
devices	            12	        48	    1
freezer	            9	        1	    1
net_cls	            4	        1	    1
perf_event	        6	        1	    1
net_prio	        4	        1	    1
hugetlb	            5	        1	    1
pids	            3	        58	    1
rdma            	11	        1	    1
```



### 手动mount memory cgroup

```
[root@localhost cgroup]# mkdir cgroup-demo
[root@localhost cgroup]# mkdir cgroup-demo/memory
[root@localhost cgroup]# mount -t cgroup -o memory memory ./cgroup-demo/memory
[root@localhost cgroup]# ls cgroup-demo/memory/
cgroup.clone_children       memory.kmem.max_usage_in_bytes      memory.memsw.limit_in_bytes      memory.usage_in_bytes
cgroup.event_control        memory.kmem.slabinfo                memory.memsw.max_usage_in_bytes  memory.use_hierarchy
cgroup.procs                memory.kmem.tcp.failcnt             memory.memsw.usage_in_bytes      notify_on_release
cgroup.sane_behavior        memory.kmem.tcp.limit_in_bytes      memory.move_charge_at_immigrate  release_agent
demo                        memory.kmem.tcp.max_usage_in_bytes  memory.numa_stat                 system.slice
machine.slice               memory.kmem.tcp.usage_in_bytes      memory.oom_control               tasks
memory.failcnt              memory.kmem.usage_in_bytes          memory.pressure_level            user.slice
memory.force_empty          memory.limit_in_bytes               memory.soft_limit_in_bytes
memory.kmem.failcnt         memory.max_usage_in_bytes           memory.stat
memory.kmem.limit_in_bytes  memory.memsw.failcnt                memory.swappiness
[root@localhost ~]# mount -t cgroup
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,memory)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,pids)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,net_cls,net_prio)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,hugetlb)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,perf_event)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpu,cpuacct)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpuset)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,freezer)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,blkio)
cgroup on /sys/fs/cgroup/rdma type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,rdma)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,devices)
demo-memory on /go/cgroup/cgroup-demo/memory type cgroup (rw,relatime,seclabel,memory)

```


### 手动 mount cpu cgroup 报错，cpu already mounted or mount point busy. 解决办法

```
[root@localhost cgroup]# mkdir ./cgroup-demo/cpu
[root@localhost cgroup]# mount -t cgroup -o cpu cpu ./cgroup-demo/cpu
mount: /go/cgroup/cgroup-demo/cpu: cpu already mounted or mount point busy.
[root@localhost cgroup]# uname -r
4.18.0-193.el8.x86_64
[root@localhost cgroup]# cat /proc/cgroups
#subsys_name	hierarchy	num_cgroups	enabled
cpuset	8	1	1
cpu	7	48	1
cpuacct	7	48	1
blkio	10	48	1
memory	2	91	1
devices	12	48	1
freezer	9	1	1
net_cls	4	1	1
perf_event	6	1	1
net_prio	4	1	1
hugetlb	5	1	1
pids	3	58	1
rdma	11	1	1
[root@localhost cgroup]# cat /etc/redhat-release
CentOS Linux release 8.2.2004 (Core)
```

关于以上这个报错，google半天没找到眉头思路，最终在 [米开朗基杨](https://fuckcloudnative.io/) 大佬的指导下得知Centos8系统cpu,cpuacct要合并一起使用，应该是新的系统systemd把两个合并了

```
[root@localhost cgroup]# cat /etc/systemd/system.conf  |grep cpu
#JoinControllers=cpu,cpuacct net_cls,net_prio

[root@localhost cgroup]# mount -t cgroup -o cpu,cpuacct demo-cpu ./cgroup-demo/cpu
[root@localhost cgroup]# mount -t cgroup
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,memory)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,pids)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,net_cls,net_prio)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,hugetlb)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,perf_event)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpu,cpuacct)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpuset)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,freezer)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,blkio)
cgroup on /sys/fs/cgroup/rdma type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,rdma)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,devices)
demo-memory on /go/cgroup/cgroup-demo/memory type cgroup (rw,relatime,seclabel,memory)
demo-cpu on /go/cgroup/cgroup-demo/cpu type cgroup (rw,relatime,seclabel,cpu,cpuacct)

```


### 把当前bash进程ID加入到cgroup memory限制当中,限制100m内存

```
[root@localhost memory]# mkdir ./cgroup-demo/memory/demo-test/
[root@localhost memory]# cd ./cgroup-demo/memory/demo-test/
[root@localhost demo-test]# ls
cgroup.clone_children           memory.kmem.tcp.max_usage_in_bytes  memory.oom_control
cgroup.event_control            memory.kmem.tcp.usage_in_bytes      memory.pressure_level
cgroup.procs                    memory.kmem.usage_in_bytes          memory.soft_limit_in_bytes
memory.failcnt                  memory.limit_in_bytes               memory.stat
memory.force_empty              memory.max_usage_in_bytes           memory.swappiness
memory.kmem.failcnt             memory.memsw.failcnt                memory.usage_in_bytes
memory.kmem.limit_in_bytes      memory.memsw.limit_in_bytes         memory.use_hierarchy
memory.kmem.max_usage_in_bytes  memory.memsw.max_usage_in_bytes     notify_on_release
memory.kmem.slabinfo            memory.memsw.usage_in_bytes         tasks
memory.kmem.tcp.failcnt         memory.move_charge_at_immigrate
memory.kmem.tcp.limit_in_bytes  memory.numa_stat
[root@localhost demo-test]#  echo "100m" > memory.limit_in_bytes
[root@localhost demo-test]# cat memory.limit_in_bytes
104857600
[root@localhost demo-test]# echo $$ > tasks
[root@localhost demo-test]# cat tasks
1689
2239
[root@localhost demo-test]# ps aux |grep 1689
root         931  0.0  0.2  16892  2132 ?        Ss   22:30   0:00 /usr/sbin/mcelog --ignorenodev --daemon --foreground
root        1689  0.0  0.7  27448  6088 pts/0    Ss   22:33   0:00 -bash
root        2254  0.0  0.1  12320  1084 pts/0    S+   23:02   0:00 grep --color=auto 1689

#启动一个500m内存的测试程序
[root@localhost cgroup]# stress --vm-bytes 500m --vm-keep -m 1
stress: info: [2314] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd

#可以使用htop查看stress程序的内存情况，永远不会超过100m内存的，因为我们限制的是当前的bash,产生的子进程一并限制，想想docker 内存限制是不是也是类型这样的实现

```


### 把进程ID加入到cgroup cpu限制 20% 的使用率


```
[root@localhost cgroup]# mkdir ./cgroup-demo/cpu/demo-cpu
[root@localhost cgroup]# echo 20000 > ./cgroup-demo/cpu/demo-cpu/cpu.cfs_quota_us
[root@localhost cgroup]# echo  $$ > ./cgroup-demo/cpu/demo-cpu/tasks
[root@localhost cgroup]# cat  ./cgroup-demo/cpu/demo-cpu/tasks
1689
2495
[root@localhost cgroup]# stress --vm-bytes 500m --vm-keep -m 1
stress: info: [2504] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd

#可以使用htop查看stress程序的cpu的使用情况，永远不会超过20%CPU的，因为我们限制的是当前的bash,产生的子进程一并限制，想想docker CPU限制是不是也是类型这样的实现
```

### 查看进程ID都有哪些cgroup

```
[root@localhost ~]# cat /proc/2504/cgroup
12:devices:/system.slice/sshd.service
11:rdma:/
10:blkio:/system.slice/sshd.service
9:freezer:/
8:cpuset:/
7:cpu,cpuacct:/demo-cpu
6:perf_event:/
5:hugetlb:/
4:net_cls,net_prio:/
3:pids:/user.slice/user-0.slice/session-1.scope
2:memory:/demo-test
1:name=systemd:/user.slice/user-0.slice/session-1.scope
[root@localhost ~]#
```



### go语言简单实现，[github代码下载](https://github.com/xiaoJack/linux-cgroup-go)

```
package main

import (
	"fmt"
	"io/ioutil"
	"os"
	"os/exec"
	"path"
	"strconv"
	"syscall"
)

const (
	// 挂载了 memory subsystem的hierarchy的根目录位置
	cgroupMemoryHierarchyMount = "/sys/fs/cgroup/memory"
	cgroupCPUHierarchyMount    = "/sys/fs/cgroup/cpu"
)

func main() {

	if os.Args[0] == "/proc/self/exe" {
		//容器进程
		fmt.Printf("current pid %d \n", syscall.Getpid())

		cmd := exec.Command("sh", "-c", "stress --vm-bytes 500m --vm-keep -m 1")
		cmd.SysProcAttr = &syscall.SysProcAttr{}
		cmd.Stdin = os.Stdin
		cmd.Stdout = os.Stdout
		cmd.Stderr = os.Stderr
		if err := cmd.Run(); err != nil {
			panic(err)
		}
	}

	cmd := exec.Command("/proc/self/exe")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS,
	}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	err := cmd.Start()
	if err != nil {
		panic(err)
	}
	// 得到 fork出来进程映射在外部命名空间的pid
	fmt.Printf("%+v", cmd.Process.Pid)

	cgroupMenory(cmd.Process.Pid)
	cgroupCPU(cmd.Process.Pid)

	cmd.Process.Wait()
}

func cgroupCPU(pid int) {
	// 创建子cgroup
	newCgroupCPU := path.Join(cgroupCPUHierarchyMount, "cgroup-demo-cpu")
	os.Mkdir(newCgroupCPU, 0755)

	// 将容器进程放到子cgroup中
	if err := ioutil.WriteFile(path.Join(newCgroupCPU, "tasks"), []byte(strconv.Itoa(pid)), 0644); err != nil {
		panic(err)
	}
	// 限制cgroup的CPU使用
	if err := ioutil.WriteFile(path.Join(newCgroupCPU, "cpu.cfs_quota_us"), []byte("20000"), 0644); err != nil {
		panic(err)
	}
}

func cgroupMenory(pid int) {
	// 创建子cgroup
	newCgroupMemory := path.Join(cgroupMemoryHierarchyMount, "cgroup-demo-memory")
	os.Mkdir(newCgroupMemory, 0755)

	// 将容器进程放到子cgroup中
	if err := ioutil.WriteFile(path.Join(newCgroupMemory, "tasks"), []byte(strconv.Itoa(pid)), 0644); err != nil {
		panic(err)
	}
	// 限制cgroup的内存使用
	if err := ioutil.WriteFile(path.Join(newCgroupMemory, "memory.limit_in_bytes"), []byte("100m"), 0644); err != nil {
		panic(err)
	}
}

```



#### 参考文献
- https://learnku.com/articles/42117
- https://coolshell.cn/articles/17049.html
- https://tech.meituan.com/2015/03/31/cgroups.html
- https://segmentfault.com/a/1190000006917884