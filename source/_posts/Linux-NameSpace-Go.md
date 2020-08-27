---
title: Linux NameSpace Go
date: 2020-08-05 10:19:23
tags: [Go,Docker]
---



## 概念
Namespace是Linux内核对系统资源进行隔离和虚拟化的特性，这些系统资源包括进程ID、主机名、用户ID、网络访问、进程间通讯和文件系统等


### 当前Linux一共实现六种不同类型的namespace。
Namespace类型 | 系统调用参数 | 内核版本 |
---|---|---
UTS namespaces	    | CLONE_NEWUTS  |   2.6.19
IPC namespaces	    | CLONE_NEWIPC  |	2.6.19
PID namespaces	    | CLONE_NEWPID  |	2.6.24
Network namespaces	| CLONE_NEWNET  |	2.6.29
User namespaces     | CLONE_NEWUSER	|   3.8
Mount namespaces    | CLONE_NEWNS	|   2.4.19





### UTS Namespace
UTS namespace 功能最简单，它只隔离了 hostname 和 NIS domain name 两个资源。同一个 namespace 里面的进程看到的 hostname 和 domain name 是相同的，这两个值可以通过 sethostname(2) 和 setdomainname(2) 来进行设置，也可以通过 uname(2)、gethostname(2) 和 getdomainname(2) 来读取。

```
package main

import (
    "log"
    "os"
    "os/exec"
    "syscall"
)

func main() {
    cmd := exec.Command("sh")
    cmd.SysProcAttr = &syscall.SysProcAttr{
        Cloneflags: syscall.CLONE_NEWUTS, 
    }
    cmd.Stdin = os.Stdin
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr

    if err := cmd.Run(); err != nil {
        log.Fatal(err)
    }
}

```



### User Namesapce
User namespace 隔离的是用户和组信息，在不同的 namespace 中用户可以有相同的 UID 和 GID，它们之间互相不影响。另外，还有父子 namespace 之间用户和组映射的功能。父 namespace 中非 root 用户也能成为子 namespace 中的 root，这样就能增加安全性（如果所有 namespace 的 root 用户都是一样的，会带来子 namespace 操作父 namespace 内容的危险）。

```
package main

import (
    "log"
    "os"
    "os/exec"
    "syscall"
)

func main() {
    cmd := exec.Command("sh")

    cmd.SysProcAttr = &syscall.SysProcAttr{
        Cloneflags: syscall.CLONE_NEWNS |
            syscall.CLONE_NEWUTS |
            syscall.CLONE_NEWUSER,
        UidMappings: []syscall.SysProcIDMap{
            {
                ContainerID: 0,
                HostID:      os.Getuid(),
                Size:        1,
            },
        },
        GidMappings: []syscall.SysProcIDMap{
            {
                ContainerID: 0,
                HostID:      os.Getgid(),
                Size:        1,
            },
        },
    }


    cmd.Stdin = os.Stdin
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr

    if err := cmd.Run(); err != nil {
        log.Fatal(err)
    }
}
```



### Mount Namespace
mount namespace 是用来隔离各个进程看到的挂载点视图。在不同namespace中的进程看到的文件系统层次是不一样的。在mount namespace 中调用mount()和umount()仅仅只会影响当前namespace内的文件系统，而对全局的文件系统是没有影响的。




### PID namespace
PID namespace 隔离的是进程的 pid 属性，也就是说不同的 namespace 中的进程可以有相同的 pid。PID namespace 和我们常见的系统规则一样，都是从 pid 1 开始，每次 fork、vfork、clone 调用都会分配新的 pid。

PID namespace 第一个运行的进程的 pid 编号为 1，也被成为 init 进程。所有的孤儿进程（父进程被杀死）都会被 reparent 到 init 进程，如果 init 进程挂掉了，系统会发送 SIGKILL 信号给该 namespace 中的所有进程来杀死它们。由此可见，init 进程对于 PID namespace 至关重要，因此在容器中你可能听说过关于哪个程序最适合做 init 进程的争论。

PID namespace 另外一个特殊的特性是，通过 unshare 和 setns 系统调用都不会都不会把当前进程加入到新的 namespace，而是把该进程的子进程进入到里面。之所以这样设计，是因为 pid 是进程非常重要的信息，很多应用程序都会假定这个值不会变化，如果 unshare 或者 setns 把当前进程加入到新的 namespace 中，那么进程的 PID 将会发生变化，原来的 PID 也会被其他进程使用，会导致很多程序出现问题。


![avatar](https://uploads.toptal.io/blog/image/674/toptal-blog-image-1416487554032.png)




### IPC namespaces
IPC 是进程间通信的意思，作用是每个 namespace 都有自己的 IPC，防止不同 namespace 进程能互相通信（这样存在安全隐患）。

IPC namespace 隔离的是 IPC（Inter-Process Communication） 资源，也就是进程间通信的方式，包括 System V IPC 和 POSIX message queues。每个 IPC namespace 都有自己的 System V IPC 和 POSIX message queues，并且对其他 namespace 不可见，这样的话，只有同一个 namespace 下的进程之间才能够通信。

下面这些 /proc 中内容对于每个 namespace 都是不同的：

/proc/sys/fs/mqueue 下的 POSIX message queues
/proc/sys/kernel 下的 System V IPC，包括 msgmax, msgmnb, msgmni, sem, shmall, shmmax, shmmni, and shm_rmid_forced
/proc/sysvipc/：保存了该 namespace 下的 system V ipc 信息
在 linux 下和 ipc 打交道，需要用到以下两个命令：

ipcs：查看IPC(共享内存、消息队列和信号量)的信息
ipcmk：创建IPC(共享内存、消息队列和信号量)的信息


```
package main

import (
    "fmt"
    "os"
    "path/filepath"
    "syscall"
    "flag"
    "github.com/docker/docker/pkg/reexec"
    "os/exec"
)



func init() {

    fmt.Printf("arg0=%s,\n",os.Args[0])

    reexec.Register("initFuncName", func() {
        fmt.Printf("\n>> namespace setup code goes here <<\n\n")

        newRoot := os.Args[1]

        if err := mountProc(newRoot); err != nil {
            fmt.Printf("Error mounting /proc - %s\n", err)
            os.Exit(1)
        }

        fmt.Printf("newRoot:%s \n",newRoot)
        if err := pivotRoot(newRoot); err != nil {
            fmt.Printf("Error running pivot_root - %s\n", err)
            os.Exit(1)
        }

        nsRun() //calling clone() to create new process goes here
    })

    if reexec.Init() {
        os.Exit(0)
    }
}




func checkRootfs(rootfsPath string) {
    if _, err := os.Stat(rootfsPath); os.IsNotExist(err) {
        fmt.Printf("rootfsPath %s is not found you may need to download it",rootfsPath)
        os.Exit(1)
    }
}

//implement pivot_root by syscall
func pivotRoot(newroot string) error {

    preRoot := "/.pivot_root"
    putold := filepath.Join(newroot,preRoot) //putold:/tmp/ns-proc/rootfs/.pivot_root


    // pivot_root requirement that newroot and putold must not be on the same filesystem as the current root
    //current root is / and new root is /tmp/ns-proc/rootfs and putold is /tmp/ns-proc/rootfs/.pivot_root
    //thus we bind mount newroot to itself to make it different
    //try to comment here you can see the error
    if err := syscall.Mount(newroot, newroot, "", syscall.MS_BIND|syscall.MS_REC, ""); err != nil {
        fmt.Printf("mount newroot:%s to itself error \n",newroot)
        return err
    }

    // create putold directory, equal to mkdir -p xxx
    if err := os.MkdirAll(putold, 0700); err != nil {
        fmt.Printf("create putold directory %s erro \n",putold)
        return err
    }

    // call pivot_root
    if err := syscall.PivotRoot(newroot, putold); err != nil {
        fmt.Printf("call PivotRoot error, newroot:%s,putold:%s \n",newroot,putold)
        return err
    }

    // ensure current working directory is set to new root
    if err := os.Chdir("/"); err != nil {
        return err
    }

    // umount putold, which now lives at /.pivot_root
    putold = preRoot
    if err := syscall.Unmount(putold, syscall.MNT_DETACH); err != nil {
        fmt.Printf("umount putold:%s error \n",putold)
        return err
    }

    // remove putold
    if err := os.RemoveAll(putold); err != nil {
        fmt.Printf("remove putold:%s error \n",putold)
        return err
    }

    return nil
}


func mountProc(newroot string) error {
    source := "proc"
    target := filepath.Join(newroot, "/proc")
    fstype := "proc"
    flags := 0
    data := ""

    os.MkdirAll(target, 0755)
    if err := syscall.Mount(
        source,
        target,
        fstype,
        uintptr(flags),
        data,
    ); err != nil {
        return err
    }

    return nil
}




func nsRun() {
    cmd := exec.Command("/bin/sh")

    cmd.Env = []string{"PATH=/sbin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/bin"}

    //set identify for this demo
    cmd.Stdin = os.Stdin
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr


    if err := cmd.Run(); err != nil {
        fmt.Printf("Error running the /bin/sh command - %s\n", err)
        os.Exit(1)
    }
}




func main() {

    var rootfsPath string
    flag.StringVar(&rootfsPath, "rootfs", "/tmp/ns-proc/rootfs", "Path to the root filesystem to use")
    flag.Parse()

    checkRootfs(rootfsPath)

    cmd := reexec.Command("initFuncName",rootfsPath)

    cmd.Stdin = os.Stdin
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr

    cmd.SysProcAttr = &syscall.SysProcAttr{
        Cloneflags: syscall.CLONE_NEWNS |
            syscall.CLONE_NEWUTS |
            syscall.CLONE_NEWIPC |
            syscall.CLONE_NEWPID |
            syscall.CLONE_NEWNET |
            syscall.CLONE_NEWUSER,
        UidMappings: []syscall.SysProcIDMap{
            {
                ContainerID: 0,
                HostID:      os.Getuid(),
                Size:        1,
            },
        },
        GidMappings: []syscall.SysProcIDMap{
            {
                ContainerID: 0,
                HostID:      os.Getgid(),
                Size:        1,
            },
        },
    }


    if err := cmd.Run(); err != nil {
        fmt.Printf("Error running the reexec.Command - %s\n", err)
        os.Exit(1)
    }

}

```




### Network Namespace
Net namespace 隔离的是和网络相关的资源，包括网络设备、路由表、防火墙(iptables)、socket（ss、netstat）、 /proc/net 目录、/sys/class/net 目录、网络端口(network interfaces)等等。

一个物理网络设备只能出现在最多一个网络 namespace 中，不同网络 namespace 之间可以通过创建 veth pair 提供类似管道的通信。



```
// +build linux

package main

import (
	"bytes"
	"flag"
	"fmt"
	"github.com/docker/docker/pkg/reexec"
	"os"
	"os/exec"
	"syscall"

	"path/filepath"
	"net"
	"time"

)

func init() {

	fmt.Printf("arg0=%s,\n",os.Args[0])

	reexec.Register("initFuncName", func() {
		fmt.Printf("\n>> namespace setup code goes here <<\n\n")

		newRoot := os.Args[1]

		if err := mountProc(newRoot); err != nil {
			fmt.Printf("Error mounting /proc - %s\n", err)
			os.Exit(1)
		}

		fmt.Printf("newRoot:%s \n",newRoot)
		if err := pivotRoot(newRoot); err != nil {
			fmt.Printf("Error running pivot_root - %s\n", err)
			os.Exit(1)
		}


		if err := waitNetwork(); err != nil {
			fmt.Printf("Error waiting for network - %s\n", err)
			os.Exit(1)
		}


		nsRun() //calling clone() to create new process goes here
	})

	if reexec.Init() {
		os.Exit(0)
	}
}


func nsRun() {
	cmd := exec.Command("sh")

    cmd.Env = []string{"PATH=/sbin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/bin"}
	//set identify for this demo
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr


	if err := cmd.Run(); err != nil {
		fmt.Printf("Error running the /bin/sh command - %s\n", err)
		os.Exit(1)
	}
}

func main() {

	var rootfsPath,netsetPath string

	flag.StringVar(&netsetPath, "netsetPath", "./netsetter.sh", "Path to the netset shell")
	flag.StringVar(&rootfsPath, "rootfs", "/tmp/ns-proc/rootfs", "Path to the root filesystem to use")
	flag.Parse()

	checkRootfs(rootfsPath)
	checkNetsetter(netsetPath)

	cmd := reexec.Command("initFuncName",rootfsPath)

    cmd.Env = []string{"PATH=/sbin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/bin"}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWNS |
			syscall.CLONE_NEWUTS |
			syscall.CLONE_NEWIPC |
			syscall.CLONE_NEWPID |
			syscall.CLONE_NEWNET |
			syscall.CLONE_NEWUSER,
		UidMappings: []syscall.SysProcIDMap{
			{
				ContainerID: 0,
				HostID:      os.Getuid(),
				Size:        1,
			},
		},
		GidMappings: []syscall.SysProcIDMap{
			{
				ContainerID: 0,
				HostID:      os.Getgid(),
				Size:        1,
			},
		},
	}



	if err := cmd.Start(); err != nil {
		fmt.Printf("Error starting the reexec.Command - %s\n", err)
		os.Exit(1)
	}

	// run netsetgo using default args

	pid := fmt.Sprintf("%d", cmd.Process.Pid)

	//netsetCmd := exec.Command("whoami" ) //see current user , my result is ubuntu not root
	netsetCmd := exec.Command("sudo",netsetPath, pid) //
	var out bytes.Buffer
	var stderr bytes.Buffer
	netsetCmd.Stdout = &out
	netsetCmd.Stderr = &stderr

	if err := netsetCmd.Start(); err != nil {
		fmt.Printf("Error running netsetg:%s, stderr:%s, stdout:%s",fmt.Sprint(err),stderr.String(),out.String())
		os.Exit(1)
	}
	fmt.Printf("run netsetter: stdout:%s \n",out.String())

	if err := cmd.Wait(); err != nil {
		fmt.Printf("Error waiting for the reexec.Command - %s\n", err)
		os.Exit(1)
	}

}



func checkNetsetter(netsetPath string) {
	if _, err := os.Stat(netsetPath); os.IsNotExist(err) {
		errMsg := fmt.Sprintf(`file %s not found! you must have a netsetter binary or shell and run with root privilege，
or run with argument -netsetPath path_to_your_netsetter
`, netsetPath)
		fmt.Println(errMsg)
		os.Exit(1)
	}
}


func waitNetwork() error {
	maxWait := time.Second * 60
	checkInterval := time.Second
	timeStarted := time.Now()

	for {
		fmt.Printf("status: waiting network ...\n")
		interfaces, err := net.Interfaces()
		if err != nil {
			return err
		}

		if len(interfaces) > 1 {
			return nil
		}

		if time.Since(timeStarted) > maxWait {
			return fmt.Errorf("Timeout after %s waiting for network", maxWait)
		}

		time.Sleep(checkInterval)
	}
}


func checkRootfs(rootfsPath string) {
	if _, err := os.Stat(rootfsPath); os.IsNotExist(err) {
		fmt.Printf("rootfsPath %s is not found you may need to download it",rootfsPath)
		os.Exit(1)
	}
}

//implement pivot_root by syscall
func pivotRoot(newroot string) error {

	preRoot := "/.pivot_root"
	putold := filepath.Join(newroot,preRoot) //putold:/tmp/ns-proc/rootfs/.pivot_root


	// pivot_root requirement that newroot and putold must not be on the same filesystem as the current root
	//current root is / and new root is /tmp/ns-proc/rootfs and putold is /tmp/ns-proc/rootfs/.pivot_root
	//thus we bind mount newroot to itself to make it different
	//try to comment here you can see the error
	if err := syscall.Mount(newroot, newroot, "", syscall.MS_BIND|syscall.MS_REC, ""); err != nil {
		fmt.Printf("mount newroot:%s to itself error \n",newroot)
		return err
	}

	// create putold directory, equal to mkdir -p xxx
	if err := os.MkdirAll(putold, 0700); err != nil {
		fmt.Printf("create putold directory %s erro \n",putold)
		return err
	}

	// call pivot_root
	if err := syscall.PivotRoot(newroot, putold); err != nil {
		fmt.Printf("call PivotRoot error, newroot:%s,putold:%s \n",newroot,putold)
		return err
	}

	// ensure current working directory is set to new root
	if err := os.Chdir("/"); err != nil {
		return err
	}

	// umount putold, which now lives at /.pivot_root
	putold = preRoot
	if err := syscall.Unmount(putold, syscall.MNT_DETACH); err != nil {
		fmt.Printf("umount putold:%s error \n",putold)
		return err
	}

	// remove putold
	if err := os.RemoveAll(putold); err != nil {
		fmt.Printf("remove putold:%s error \n",putold)
		return err
	}

	return nil
}


func mountProc(newroot string) error {
	source := "proc"
	target := filepath.Join(newroot, "/proc")
	fstype := "proc"
	flags := 0
	data := ""

	os.MkdirAll(target, 0755)
	if err := syscall.Mount(
		source,
		target,
		fstype,
		uintptr(flags),
		data,
	); err != nil {
		return err
	}

	return nil
}
```





Centos7 系统默认没有开启NameSpases,以下方式开启
```
echo 640 > /proc/sys/user/max_user_namespaces
```

参考代码
* https://github.com/xiaoJack/linux-namespace-go


参考文献
* https://developer.aliyun.com/article/64928
* https://here2say.com/41/
* https://images.contentstack.io/v3/assets/blt300387d93dabf50e/bltb6200bc085503718/5e1f209a63d1b6503160c6d5/containers-vs-virtual-machines.jpg
