---
layout: post
title: "在Docker上运行Scrapy"
date: 2017-10-10 10:08
comments: true
tags: Note
---

之前使用Scrapy写过一个空气质量的采集程序，最近要切换到另外一台服务器上面去，折腾了几个小时的安装环境还是没跑起来。几次之后就放弃了，刚好那台服务器上面安装了Docker的环境，运行了一个Nexus的服务几个月来一直都很稳定，那为啥不可以把Scrapy也放在上面运行呢？

### 操作过程

下面记录下我的处理的过程：

#### （1）创建dockfile

在`scrapy.cfg`文件所在的目录下面创建`dockfile`，里面的内容如下：

```
FROM ubuntu
MAINTAINER hhtczengjing@gmail.com
RUN apt-get update \
	&& apt-get -y dist-upgrade \
	&& apt-get install -y openssh-server  \
	&& apt-get install -y python2.7-dev python-pip  \
	&& apt-get install -y zlib1g-dev libffi-dev libssl-dev  \
	&& apt-get install -y libxml2-dev libxslt1-dev  \
	&& apt-get install -y libmysqlclient-dev \ 
	&& pip install setuptools  \
	&& pip install Scrapy \
	&& pip install MySQL \ 
	&& apt-get clean      \
 	&& apt-get autoclean  \
 	&& rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
COPY . /data
WORKDIR /data
CMD ["scrapy", "crawl", "aqi"]
```

#### （2）编译镜像

在命令行进入dockerfile所在目录执行如下(改过程持续时间比较长)：

```
docker build -t aqi:1.0.0 .
```

编译完成之后可以通过如下的方式运行：

```
docker run --rm aqi:1.0.0
```

#### （3）配置Crontab


在服务器上面创建一个`run.sh`的可执行文件，里面的内容如下：

```
#!/bin/bash

. /etc/profile
. ~/.bash_profile
docker run --rm aqi:1.0.0
```

使用`crontab -e`添加一条记录如下：

```
0 * * * * sh /home/aqi/run.sh >> /home/aqi/log/log.`date +\%Y\%m\%d\%H\%M\%S` 2>&1
```

这样下来就实现了一个在Docker上面运行Scrapy采集任务的功能。

### 参考资料

1、[Dockerfile reference](https://docs.docker.com/engine/reference/builder/)