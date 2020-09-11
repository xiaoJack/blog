---
title: Linux Union File System 文件系统
date: 2020-09-11 16:33:19
tags: Docker
---

# Linux Union File System 文件系统


## UnionFS
Union File System，所谓UnionFS就是把不同物理位置的目录合并mount到同一个目录中，中文叫联合文件系统的文件系统

具备如下特性：

> 联合挂载：将多个目录按层次组合，一并挂载到一个联合挂载点<br>
> 写时复制：对联合挂载点的修改不会影响到底层的多个目录，而是使用其他目录记录修改的操作<br>

目前有多种文件系统可以被当作联合文件系统，实现如上的功能：overlay2，aufs，devicemapper，btrfs，zfs，vfs等等。


## AUFS
AUFS 是一种UnionFS, AUFS 英文全称就 Advanced Multi-Layered Unification Filesystem 曾经也叫 Acronym Multi-Layered Unification Filesystem、Another Multi-Layered Unification Filesystem
到现在AUFS都还进不了Linux内核主干，据说是Linus一直不同意


### AUFS 使用

```
root@jackzhu-ubuntu:/aufs# tree
.
├── a
│   └── 1.txt
├── b
│   └── 2.txt
├── c
│   └── 3.txt
└── mnt

4 directories, 3 files
root@jackzhu-ubuntu:/aufs# cat */*.txt
a dir file
b dir file
c dir file

root@jackzhu-ubuntu:/aufs# mount -t aufs -o dirs=./a:./b:./c none ./mnt/
root@jackzhu-ubuntu:/aufs# tree
.
├── a
│   └── 1.txt
├── b
│   └── 2.txt
├── c
│   └── 3.txt
└── mnt
    ├── 1.txt
    ├── 2.txt
    └── 3.txt

4 directories, 6 files
```

刚刚创建了a,b,c,mnt四个目录，分别在a,b,c目录写入一个txt文件，然后把a,b,c三个文件夹mount到mnt目录里面,这个时候就能在mnt目录里面看到a,b,c目录里面的所有文件了，注意上面mount命令没有加权限的参数，接下来尝试在mnt目录修改几个txt文件

```
root@jackzhu-ubuntu:/aufs/mnt# pwd
/aufs/mnt
root@jackzhu-ubuntu:/aufs/mnt# cat 1.txt
a dir file
a dir file
root@jackzhu-ubuntu:/aufs/mnt# cat 2.txt
b dir file
b dir file
root@jackzhu-ubuntu:/aufs/mnt# cat 3.txt
c dir file
c dir file

root@jackzhu-ubuntu:/aufs# tree
.
├── a
│   ├── 1.txt
│   ├── 2.txt
│   └── 3.txt
├── b
│   └── 2.txt
├── c
│   └── 3.txt
└── mnt
    ├── 1.txt
    ├── 2.txt
    └── 3.txt

4 directories, 8 files

root@jackzhu-ubuntu:/aufs# cat b/2.txt
b dir file
root@jackzhu-ubuntu:/aufs# cat c/3.txt
c dir file
root@jackzhu-ubuntu:/aufs# cat a/1.txt
a dir file
a dir file
root@jackzhu-ubuntu:/aufs# cat a/2.txt
b dir file
b dir file
root@jackzhu-ubuntu:/aufs# cat a/3.txt
c dir file
c dir file
root@jackzhu-ubuntu:/aufs#
```
看上面的结果，很奇怪a目录多了2个txt文件，b,c目录的文件并没有改变,原因是上面mount操作的时候没有指定权限，默认mount aufs 多个目录的时候，只有第一个目录是可写的目录，其它后面几个目录都是只读的，那为什么第一个目录 会多几个文件，这个技术叫CoW，全称copy-on-wirte 中文叫【写时复制】，目的就是为了节省磁盘空间提高资源利用率

### 如果a,b,c目录有相同的文件名怎么办？<br>

```
root@jackzhu-ubuntu:/aufs# tree
.
├── a
│   └── 1.txt
├── b
│   └── 2.txt
├── c
│   └── 3.txt
└── mnt

4 directories, 3 files
root@jackzhu-ubuntu:/aufs# cat a/1.txt
a dir file
a dir file
root@jackzhu-ubuntu:/aufs# cat b/2.txt
b dir file
root@jackzhu-ubuntu:/aufs# cat c/3.txt
c dir file
root@jackzhu-ubuntu:/aufs# echo "a dir file" > a/2.txt
root@jackzhu-ubuntu:/aufs# tree
.
├── a
│   ├── 1.txt
│   └── 2.txt
├── b
│   └── 2.txt
├── c
│   └── 3.txt
└── mnt

4 directories, 4 files
root@jackzhu-ubuntu:/aufs# mount -t aufs -o dirs=./a:./b:./c none ./mnt
root@jackzhu-ubuntu:/aufs# tree
.
├── a
│   ├── 1.txt
│   └── 2.txt
├── b
│   └── 2.txt
├── c
│   └── 3.txt
└── mnt
    ├── 1.txt
    ├── 2.txt
    └── 3.txt

4 directories, 7 files
root@jackzhu-ubuntu:/aufs# cat ./mnt/2.txt
a dir file

root@jackzhu-ubuntu:/aufs# umount none
root@jackzhu-ubuntu:/aufs# mount -t aufs -o dirs=./b:./a:./c none ./mnt
root@jackzhu-ubuntu:/aufs# cat ./mnt/2.txt
b dir file
root@jackzhu-ubuntu:/aufs#
```
Aufs会根据mount命令的顺序最左边目录优先级越高，后面目录的文件就不会出现



## overlay2

overlay2是一个类似于aufs的现代的联合文件系统，并且更快。overlay2已被收录进linux内核，它需要内核版本不低于4.0，如果是RHEL或Centos的话则不低于3.10.0-514。

overlay2结构：<br>
![image](https://docs.docker.com/storage/storagedriver/images/overlay_constructs.jpg)

如上，overlay2包括lowerdir，upperdir和merged三个层次，注意：overlay2 的文件系统下面的几个参数就是mount的固定参数， 其中：

- lowerdir：表示较为底层的目录，修改联合挂载点不会影响到lowerdir, 类似容器镜像底层，只有只读权限
- upperdir：表示较为上层的目录，修改联合挂载点会在upperdir同步修改，读写层，CoW到这个目录
- merged：是lowerdir和upperdir合并后的联合挂载点，挂载展示
- workdir：用来存放挂载后的临时文件与间接文件，runtime临时目标

## overlay2 使用

```
[root@k8s-master overlay2]# pwd
/root/go-test/overlay2
[root@k8s-master overlay2]# mkdir lower1 lower2 merged upper work
[root@k8s-master overlay2]# tree
.
├── lower1
├── lower2
├── merged
├── upper
└── work

5 directories, 0 files
[root@k8s-master overlay2]# echo "lower1 a file" > lower1/a
[root@k8s-master overlay2]# echo "lower2 b file" > lower2/b
[root@k8s-master overlay2]# echo "upper c file" > upper/c
[root@k8s-master overlay2]# tree
.
├── lower1
│   └── a
├── lower2
│   └── b
├── merged
├── upper
│   └── c
└── work

5 directories, 3 files

[root@k8s-master overlay2]# mount -t overlay overlay -o lowerdir=lower1:lower2,upperdir=upper,workdir=work merged
[root@k8s-master overlay2]# tree merged/
merged/
├── a
├── b
└── c

0 directories, 3 files
[root@k8s-master overlay2]# cat merged/a
lower1 a file
[root@k8s-master overlay2]# cat merged/b
lower2 b file
[root@k8s-master overlay2]# cat merged/c
upper c file
[root@k8s-master overlay2]#

[root@k8s-master overlay2]# echo "dddddd" >  merged/d
[root@k8s-master overlay2]# tree
.
├── lower1
│   └── a
├── lower2
│   └── b
├── merged
│   ├── a
│   ├── b
│   ├── c
│   └── d
├── upper
│   ├── c
│   └── d
└── work
    └── work

6 directories, 8 files
[root@k8s-master overlay2]#

[root@k8s-master overlay2]# rm -f merged/d
[root@k8s-master overlay2]# tree
.
├── lower1
│   └── a
├── lower2
│   └── b
├── merged
│   ├── a
│   ├── b
│   └── c
├── upper
│   └── c
└── work
    └── work

6 directories, 6 files

[root@k8s-master overlay2]# echo "dddddd" >  merged/d
[root@k8s-master overlay2]# tree
.
├── lower1
│   └── a
├── lower2
│   └── b
├── merged
│   ├── a
│   ├── b
│   ├── c
│   └── d
├── upper
│   ├── c
│   └── d
└── work
    └── work

6 directories, 8 files

[root@k8s-master overlay2]# rm -f upper/d
[root@k8s-master overlay2]# tree
.
├── lower1
│   └── a
├── lower2
│   └── b
├── merged
│   ├── a
│   ├── b
│   └── c
├── upper
│   └── c
└── work
    └── work

6 directories, 6 files


[root@k8s-master overlay2]# echo "lower1 aaaaaa" > lower1/aabb
[root@k8s-master overlay2]# tree
.
├── lower1
│   ├── a
│   └── aabb
├── lower2
│   └── b
├── merged
│   ├── a
│   ├── aabb
│   ├── b
│   └── c
├── upper
│   └── c
└── work
    └── work

6 directories, 8 files
[root@k8s-master overlay2]# echo "lower1 aaaaaabbbb" >> lower1/aabb
[root@k8s-master overlay2]# tree
.
├── lower1
│   ├── a
│   └── aabb
├── lower2
│   └── b
├── merged
│   ├── a
│   ├── aabb
│   ├── b
│   └── c
├── upper
│   └── c
└── work
    └── work

6 directories, 8 files
[root@k8s-master overlay2]# cat merged/aabb
lower1 aaaaaa
lower1 aaaaaabbbb
[root@k8s-master overlay2]# echo "lower1 aaaaaabbbbcccccc" >> merged/aabb
[root@k8s-master overlay2]# tree
.
├── lower1
│   ├── a
│   └── aabb
├── lower2
│   └── b
├── merged
│   ├── a
│   ├── aabb
│   ├── b
│   └── c
├── upper
│   ├── aabb
│   └── c
└── work
    └── work

6 directories, 9 files
[root@k8s-master overlay2]# umount  /root/go-test/overlay2/merged
[root@k8s-master overlay2]# tree
.
├── lower1
│   ├── a
│   └── aabb
├── lower2
│   └── b
├── merged
├── upper
│   ├── aabb
│   └── c
└── work
    └── work

6 directories, 5 files

```




#### 参考文献
- https://www.infoq.cn/article/analysis-of-docker-file-system-aufs-and-devicemapper/
- https://fuckcloudnative.io/posts/use-devicemapper/
- https://coolshell.cn/articles/17061.html
- https://coolshell.cn/articles/17200.html
- https://ieevee.com/tech/2017/05/12/docker-dm.html
- https://segmentfault.com/a/1190000008489207
- https://staight.github.io/2019/10/04/%E5%AE%B9%E5%99%A8%E5%AE%9E%E7%8E%B0-overlay2/
- https://docs.docker.com/storage/storagedriver/overlayfs-driver/
- https://arkingc.github.io/2017/05/05/2017-05-05-docker-filesystem-overlay/




