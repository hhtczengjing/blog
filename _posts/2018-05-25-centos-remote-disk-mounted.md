---
layout: post
title: "CentOS如何挂载远程盘"
date: 2018-05-25 11:46:29 +0800
comments: true
categories: Note
---

前段时间公司要迁移gitlab服务器，由于服务器剩余的空间不太多了，无法直接执行备份，考虑到Linux下面可以挂载其他机器的目录来直接使用，记录下整个操作的过程：

### 安装环境

(1) 检查nfs是否安装

```
rpm -qa | grep nfs
```

如果没有安装：

```
yum install nfs-utils -y
```

(2) 检查rpcbind是否安装

```
rpm -qa | grep rpcbind
```

如果没有安装：

```
yum install rpcbind  -y
```

### 服务端配置

```
vi /etc/exports

/home/data 192.168.3.93(rw,no_root_squash,no_all_squash,async)
```

这个配置表示开放本地存储目录`/home/data`只允许`192.168.3.93`这个主机有访问权限

- `rw`表示允许读写

- `no_root_squash`表示root用户具有完全的管理权限

- `no_all_squash`表示保留共享文件的UID和GID，此项是默认不写也可以；

- `async`表示数据可以先暂时在内存中，不是直接写入磁盘，可以提高性能，另外也可以配置`sync`表示数据直接同步到磁盘

配置生效

```
exportfs -r
```

启动:

```
service rpcbind start

service nfs start
```

### 客户端配置

（1）安装`nfs-utils`

```
yum -y install nfs-utils
```

（2）关闭防火墙

```
service iptables stop
```

（3）创建挂载点

```
mkdir /mnt/data
```

（4）挂载目录

```
mount -t nfs 192.168.3.102:/home/data /mnt/data -o proto=tcp -o nolock
```

1) 查看服务器共享目录信息：

```
showmount -e 192.168.3.102
```

2) 查看挂载情况：

```
df -h
```

（5）卸载目录

```
umount /mnt/data
```

### 参考资料

1、[Linux下NFS服务器的搭建与配置](https://www.cnblogs.com/liuyisai/p/5992511.html)

2、[用mount挂载远程服务器网络硬盘](https://blog.csdn.net/coolwubo/article/details/60779933)