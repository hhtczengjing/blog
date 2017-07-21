---
layout: post
title: "CentOS安装Docker"
date: 2017-07-21 11:08
comments: true
tags: Note
---

![docker-logo-compressed.png](/images/install-docker-in-centos/docker-logo-compressed.png)

之前写过一篇关于在Mac上面使用并安装Docker的文章[《Mac上Docker的安装和使用初探》](http://blog.devzeng.com/blog/using-docker-on-macos.html)，介绍了在Macos上面安装Docker的步骤。近期由于需要在一台`CentOS 6.5`的服务器上面部署一些服务，考虑到使用Docker来做这些事情，记录一下处理的步骤。

### 检查内核版本

```
uname -r
```

如果输出的信息为`2.6.32-431.el6.centos.plus.x86_64`，表示当前的内核版本是`2.6.32`。docker需要的内核版本是`3.10`，所以需要升级Linux的内核，升级的步骤如下：

##### (1) 导入public key

```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```

##### (2) 安装ELRepo

```
rpm -Uvh http://www.elrepo.org/elrepo-release-6-6.el6.elrepo.noarch.rpm
```

##### (3) 安装内核

```
yum --enablerepo=elrepo-kernel install kernel-lt -y
```

目前在ELRepo源中存在如下几个版本的内核，参考地址`http://elrepo.org/linux/kernel/el6/x86_64/RPMS/`，long-term表示长期稳定版本

- 1) ml(main-line): 4.6

- 2) lt(long-term): 3.10

##### (4) 修改Grub引导顺序

```
vim /etc/grub.conf
修改default=0
```

##### (5) 重启

```
shutdown -r now
```

### 2、安装Docker

#### (1) 更新yum包

```
sudo yum update
```

#### (2) 下载rmp包

```
curl -O -sSL https://get.docker.com/rpm/1.7.0/centos-6/RPMS/x86_64/docker-engine-1.7.0-1.el6.x86_64.rpm
```

#### (3) 安装rmp包

```
sudo yum localinstall --nogpgcheck docker-engine-1.7.0-1.el6.x86_64.rpm
```

#### (4) 启动docker服务

```
sudo service docker start
```

#### (5) 验证Docker

```
sudo docker run hello-world
```

如果安装启动成功，控制台输出的结果如下所示：

![docker-hello-world.png](/images/install-docker-in-centos/docker-hello-world.png)

#### (6) 设置开机启动

```
sudo chkconfig docker on
```

如何卸载

```
yum list installed | grep docker
sudo yum -y remove docker-engine.x86_64 
```

### 参考资料

1、[《CentOS通过YUM升级centOS内核》](http://www.centoscn.com/image-text/config/2016/0707/7591.html)

2、[《Docker 1.7 Centos安装文档》](https://docs.docker.com/v1.7/docker/installation/centos/)

3、[《ELRepo.org》](http://elrepo.org/tiki/tiki-index.php)