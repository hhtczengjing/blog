---
layout: post
title: "Mac上Docker的安装和使用初探"
date: 2016-11-25 00:05:53 +0800
comments: true
tags: Docker
---

![docker-logo.png](/images/mac-docker-install/docker-logo-compressed.png)

[Docker](http://www.docker.com/) 是个划时代的开源项目，它彻底释放了虚拟化的威力，极大提高了应用的运行效率，降低了云计算资源供应的成本，同时让应用的部署、测试和分发都变得前所未有的高效和轻松！

[Docker](http://www.docker.com/) 最初是 dotCloud 公司创始人 Solomon Hykes 在法国期间发起的一个公司内部项目，它是基于 dotCloud 公司多年云服务技术的一次革新，并于2013年3月以 Apache 2.0 授权协议开源，主要项目代码在 [GitHub](https://github.com/docker/docker) 上进行维护。Docker 项目后来还加入了 Linux 基金会，并成立推动[开放容器联盟](https://www.opencontainers.org/)。

在平时的开发中经常需要在开发机器上面部署一些后台的环境，比如MySQL，Node.js，Python等，本文就以在Mac上面如何安装和配置一个MySQL数据库为基础简单介绍下Docker的初步使用。

### 下载安装步骤

[Docker for Mac](https://docs.docker.com/docker-for-mac/) 要求系统最低为 macOS 10.10.3 Yosemite，或者 2010 年以后的 Mac 机型，准确说是带 [Intel MMU 虚拟化](https://en.wikipedia.org/wiki/X86_virtualization#Intel-VT-d)的，最低 4GB 内存。如果系统不满足需求，可以考虑安装 [Docker Toolbox](https://docs.docker.com/toolbox/overview/)。如果机器安装了 [VirtualBox](https://www.virtualbox.org/) 的话，VirtualBox 的版本不要低于 4.3.30。

#### 1.下载

通过这个链接下载：<https://download.docker.com/mac/stable/Docker.dmg>

#### 2.安装

如同 macOS 其它软件一样，安装非常简单，双击下载的文件，然后将那只叫 [Moby](https://blog.docker.com/2013/10/call-me-moby-dock/) 的鲸鱼图标拖拽到 `Application` 文件夹即可（其间可能会询问系统密码）。

![install-mac-dmg.png](/images/mac-docker-install/install-mac-dmg.png)

#### 3.运行

从应用中找到 Docker 图标并点击运行。

![install-mac-apps.png](/images/mac-docker-install/install-mac-apps.png)

运行之后，会在右上角菜单栏看到多了一个鲸鱼图标，这个图标表明了 Docker 的运行状态。

![install-mac-menubar.png](/images/mac-docker-install/install-mac-menubar.png)

第一次点击图标，可能会看到这个安装成功的界面，点击 "Got it!" 可以关闭这个窗口。

![install-mac-success.png](/images/mac-docker-install/install-mac-success.png)

以后每次点击鲸鱼图标会弹出操作菜单。

![install-mac-menu.png](/images/mac-docker-install/install-mac-menu.png)

在国内使用 Docker 的话，需要配置加速器，在菜单中点击 `Preferences...`，然后查看 `Advanced` 标签，在其中的 `Registry mirrors` 部分里可以点击加号来添加加速器地址。

![install-mac-preference-advanced.png](/images/mac-docker-install/install-mac-preference-advanced.png)

国内很多云服务商都提供了加速器服务，例如：

- (1)[`阿里云加速器`](https://cr.console.aliyun.com/#/accelerator)

- (2)[`DaoCloud 加速器`](https://www.daocloud.io/mirror#accelerator-doc)

- (3)[`灵雀云加速器`](http://docs.alauda.cn/feature/accelerator.html)

注册用户并且申请加速器，会获得如 `https://mnt8vkd9.mirror.aliyuncs.com` 这样的地址。我们需要将其配置给 Docker 引擎。

安装完成后，在终端执行下面几个命令可以查看Docker的版本信息：

- (1)`docker --version`

- (2)`docker-compose --version`

- (3)`docker-machine --version`

### 安装MySQL

#### 1.下载MySQL镜像

```
docker pull mysql
```

#### 2.使用镜像创建并启动容器

Docker以镜像为基础创建启动容器的方式为：

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

启动成功后会在终端输出容器的ID，启动MySQL容器的脚本为：

```
docker run -d -p 127.0.0.1:3306:3306 \
			–-name mysql \
			-v /Users/zengjing/docker/mysql/data:/var/lib/mysql \
			-e MYSQL_ROOT_PASSWORD=”123456” mysql:latest
```

可以通过`docker ps -a`查看容器运行的情况。

参数说明：

- (1)  `-d`表示容器将以后台模式运行，所有I/O数据只能通过网络资源或者共享卷组来进行交互。
- (2)  `-p 127.0.0.1:3306:3306`将主机（127.0.0.1）的端口3306映射到容器的端口3306中。这样访问主机中的3306端口就等于访问容器中的3306端口。
- (3) `--name mysql`给容器取名为 mysql，这样方便识别。
- (4) `-v /Users/zengjing/docker/mysql/data:/var/lib/mysql`将本机的文件目录挂载到容器对应的目录（`/var/lib/mysql`）中。这样可以通过数据卷实现容器中数据的持久化。
- (5) `-e MYSQL_ROOT_PASSWORD="123456"`, `-e`表示设置环境变量，此处设置了mysql的root 用户的访问密码为 `123456`。
- (6) `mysql:latest`表示使用 mysql 的最新镜像启动一个容器。

#### 3.使用MySQL

完成上面的步骤之后就可以使用MySQL的客户端工具使用了,连接信息如下：

```
Host: 127.0.0.1
Port: 3306
UserName: root
Password: 123456
```

### 参考资料

1.[《Docker官方文档》](https://docs.docker.com)

2.[《在 Docker 中使用 mysql》](http://beyondvincent.com/2016/09/10/2016-09-10-use-mysql-with-docker/)

3.[《Docker — 从入门到实践》](https://yeasy.gitbooks.io/docker_practice/content/)

4..[《在 Mac 上跳舞的容器 — Docker》](http://mp.weixin.qq.com/s?__biz=MjM5ODQ2MDIyMA==&mid=2650712620&idx=1&sn=39b33e0f1dfc335e165051b2983f9192&scene=1&srcid=0908wpvocqwawzRQwEu9N1N7#rd)

