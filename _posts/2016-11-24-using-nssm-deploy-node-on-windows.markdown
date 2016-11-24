---
layout: post
title: "使用NSSM在Windows服务器上部署Node.js应用"
date: 2016-11-24 10:15
comments: true
tags: Node
---

![node-logo.png](/images/nssm-node-deploy/node-logo.png)

最近使用Node的Express框架做了一个简单的应用，原本是打算部署到CentOS服务器上面的，后来由于种种原因只能部署到Window的服务器上面了。Node.js在Linux上面部署非常的方便，可以使用[forever](https://github.com/foreverjs/forever)或者[pm2](https://github.com/unitech/pm2)来做这个事情，而且使用起来非常的简单，后续有机会会单独介绍如何使用，在Windows下就是稍微有点麻烦了，这两个组件都不支持。

找了一些资料发现了nssm这个工具，部署超级简单，而且会监控你安装的node服务，如果node挂了，nssm会自动重启它。下面记录下部署的步骤：

### (1)下载安装nssm

当前最新的[NSSM](https://nssm.cc)版本是2.24，可以到官网上下载最新版本。下载地址是`https://nssm.cc/release/nssm-2.24.zip`。

### (2)安装服务

#### 1）打开终端根据操作系统的位数(32/64)进入到对应的文件夹下：

```
cd F:/nssm-2.24/win32
```

nssm的使用方式如下：

```
nssm install servername //创建servername服务
nssm start servername //启动服务
nssm stop servername //暂停服务
nssm restart servername //重新启动服务
nssm remove servername //删除创建的servername服务
```

#### 2）执行创建服务的命令

```
nssm install hbtoutiaoapi
```

其中`hbtoutiaoapi`这个是创建的Windows服务的名称，命令执行成功之后会弹出一个对话框，如下图所示：

![nssm-install.png](/images/nssm-node-deploy/nssm-install.png)

说明：

- ①Path：指的是node.exe的路径
- ②Startup directory: 指的是启动的文件的路径
- ③Arguments: 指的是启动的文件的名称

总的说来其实就是相当于执行了`node.exe E:\hbtoutiaoapi\bin\www`这个命令。填写完成之后点击`Install service`就行了，然后在系统的服务里面就可以看到了。在浏览器中访问`http://localhost:3000`，如下图所示：

![preview.png](/images/nssm-node-deploy/preview.png)

###参考资料

1.[《Node.js官网》](https://nodejs.org/en/)

2.[《NSSM - the Non-Sucking Service Manager》](http://nssm.cc)

3.[《使用nssm在windows服务器上部署nodejs》](https://my.oschina.net/u/1582119/blog/316069)
