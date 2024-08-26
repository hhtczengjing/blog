---
layout: post
title: "在Docker上运行OHPM私仓服务"
date: 2024-08-26 22:20:00 +0800
comments: true
tags: Note
---

OHPM(OpenHarmony Package Manager)是鸿蒙开发中的包管理工具，类似于Maven、npm、CocoaPods等，常用的是[OpenHarmony三方库中心仓](https://ohpm.openharmony.cn)，可以到上面搜索项目需要的第三方库。

![openharmony-ohpm](/images/ohpm-repo-on-docker/openharmony-ohpm.png)

使用起来非常简单，只要下面的命令就可以快速安装

```
ohpm install <package_name> 
```

在企业内部为了共享代码一般会搭建私仓，比如：

- [使用Verdaccio搭建npm仓库](https://blog.devzeng.com/blog/npm-repo-with-verdaccio.html)

- [搭建Dart Pub镜像服务](https://blog.devzeng.com/blog/self-hosted-pub-server.html)

好在鸿蒙官方提供了一个项目(ohpm-repo，轻量级的ohpm私仓服务)，可以比较方便的来搭建一个ohpm的私仓，方便在团队之间共享代码。搭建过程很简单，[官方文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-ohpm-repo-0000001749596668)写得非常详细。

首先到[下载中心](https://developer.huawei.com/consumer/cn/download/)下载最新的`ohpm-repo`工具。

![ohpm-repo-download](/images/ohpm-repo-on-docker/ohpm-repo-download.png)

修改配置 `conf/config.yaml`，更多配置可以参考[官方文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-ohpm-repo-configuration-V5)

```
listen: 0.0.0.0:8088        # 建议修改为具体的ip/域名
```

运行 `ohpm-repo` 服务，将 `bin` 目录加入到环境变量，运行：

```
ohpm-repo install

ohpm-repo start
```

说明：

- 要求 node.js 18.x 及以上版本
- ohpm-repo 私仓不允许使用 root 用户启动，需要使用其他用户安装运行
- ohpm-repo 首次启动时，默认创建一个管理员账号，账号名称：admin，密码：12345Qq!

刚开始想在docker上面部署的但是各种报错，主要还是用户权限的问题，最近偶然发现有[hangox/ohpm-repo](https://hub.docker.com/r/hangox/ohpm-repo)这个镜像已经能够实现我的需求。

简单看了下整理了一下在docker上运行ohpm-repo的配置步骤，`Dockerfile` 配置内容如下：

```
FROM node:18

# 创建用户ohpm
RUN adduser --disabled-password --gecos '' ohpm

# 拷贝文件
COPY ohpm-repo-5.0.5.0 /home/ohpm/ohpm-repo
COPY start.sh /home/ohpm

# 给目录/文件授权
RUN chown -R ohpm:ohpm /home/ohpm/ohpm-repo \
    && chmod +x /home/ohpm/ohpm-repo/bin/ohpm-repo \
    && chmod +x /home/ohpm/start.sh

# 切换到ohpm用户
USER ohpm

# 设置工作空间
WORKDIR /home/ohpm

# 将 ohpm-repo/bin 添加到环境变量
ENV PATH=/home/ohpm/ohpm-repo/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# 默认端口号
EXPOSE 8088

# 运行启动脚本
CMD ["./start.sh"]
```

其中 `start.sh` 脚本内容如下：

```
ohpm-repo install
ohpm-repo start
```

编译运行：

```
docker build -t zengjing/ohpm-repo:5.0.5.0 .

docker run -d -p 8088:8088 --name ohpm-repo zengjing/ohpm-repo:5.0.5.0
```

找了几个第三方库上传试了一下，效果还行：

![ohpm-repo-demo](/images/ohpm-repo-on-docker/ohpm-repo-demo.png)

当然还得解决配置和目录共享的问题，代码比较简单，完整配置代码见：

https://github.com/hhtczengjing/ohpm-repo-docker

### 参考资料

- [1、ohpm-repo私仓搭建工具](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-ohpm-repo-V5)

- [2、hangox/ohpm-repo](https://hub.docker.com/r/hangox/ohpm-repo)

- [3、ohpm私库部署（一）](https://laval.csdn.net/66c465f4a0bc797cf7b61425.html)