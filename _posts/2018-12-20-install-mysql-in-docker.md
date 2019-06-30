---
layout: post
title: "Docker安装MySQL数据库"
date: 2018-12-20 18:46:37 +0800
comments: true
tags: Note
---

近期经常需要安装MySQL数据库，在此记录一下：

1、初始化创建文件夹

```
mkdir -p ~/docker/mysql/conf/
mkdir -p ~/docker/mysql/data
```

2、在`conf`目录下创建配置文件`my.cnf`

```
[mysqld]
character-set-server=utf8
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
```

3、安装

```
docker run -d -p 3306:3306 \
    --privileged=true \
    --restart=always \
    -v /Users/zengjing/docker/mysql/conf/my.cnf:/etc/mysql/my.cnf \
    -v /Users/zengjing/docker/mysql/data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD="123456" \
    --name mysql57 \
    mysql:5.7
```

参数说明：

- (1) -d (Detached)表示容器将以后台模式运行，所有I/O数据只能通过网络资源或者共享卷组来进行交互。
- (2) -p 127.0.0.1:3306:3306将主机（127.0.0.1）的端口 3306 映射到容器的端口 3306 中。这样访问主机中的 3306 端口就等于访问容器中的 3306 端口。
- (3) --name mysql57给容器取名为 mysql57，这样方便记忆。
- (4) -v /Users/zengjing/docker/mysql/data:/var/lib/mysql 将本机的文件目录挂载到容器对应的目录（/var/lib/mysql）中。这样可以通过数据卷实现容器中数据的持久化。
- (6) -e MYSQL_ROOT_PASSWORD="111111"-e 表示设置环境变量，此处设置了 mysql root 用户的初始密码为 123456。
- (7) mysql:5.7表示使用 mysql 5.7 启动一个容器。