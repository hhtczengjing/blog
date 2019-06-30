---
layout: post
title: "在Docker上搭建在线表格服务"
date: 2019-01-11 18:46:37 +0800
comments: true
tags: Note
---

工作中总少不了需要填写表格的情况，特别是对于一些需要收集信息(比如住址和号码)的表格，最近发现了一个很好用的工具[ethercalc](http://cn.ethercalc.net/), 可以很方便的搭建出多人协作的在线表格服务，而且用法和Excel一致。

![hello-ethercalc.png](/images/ethercalc-docker/hello-ethercalc.png)

下面记录一下如何快速搭建的过程：

(1) 安装redis

```
docker run --name redis -d -v /Users/zengjing/docker/redis:/data redis:latest redis-server --appendonly yes
```

(2) 安装ethercalc

```
docker run -d -p 8000:8000 --link redis:redis --restart=always audreyt/ethercalc
```

安装完成之后，在浏览器中输入`http://localhost:8000`即可。

![hello-ethercalc.png](/images/ethercalc-docker/overview-ethercalc.png)

(3) 快速创建

支持直接将Excel拖拽上去快速创建，另外也支持通过浏览器输入一个地址进行创建，比如想创建一个名字叫做`demo`的文档，直接输入`http://localhost:8000/demo`立即创建完成，并且可以将该文档链接地址发送给其他人进行填写。

![hello-ethercalc.png](/images/ethercalc-docker/fast-create.png)
