---
layout: post
title: "Eclipse无法正常启动Tomcat项目解决办法"
date: 2017-06-07 13:51:00 +8000
comments: true
tags: Java
---

![eclipse-logo.png](/images/eclipse-tomcat-lanuch-error/eclipse-logo.png)

以前一直使用 `MyEclipse` 开发 JavaEE 项目，实在是太卡了，近期将之前的项目全部迁移到了 Eclipse 上面。一段时间内都好好的，这两天突然发现启动 Tomcat 不正常了，具体表现如下：

（1）Tomcat 在 Eclipse 里面能正常启动，但在浏览器中访问`http://localhost:8080/`报404错误。也就是说 Tomcat 启动了但是里面部署的 web 项目没有启动。

（2）关闭 Eclipse 里面的 Tomcat，在 Tomcat 安装目录下双击`startup.bat`手动启动 Tomcat 服务器。访问`http://localhost:8080/`能正常打开 Tomcat 里面部署的项目。

通过网上一篇文章[《eclipse启动tomcat无法访问》](http://blog.csdn.net/wqjsir/article/details/7169838/)终于解决了问题，貌似之前在MyEclipse里面也遇到过这个问题，下面记录一下解决的办法：

（1）在 Eclipse 中的 Server 页面，双击 Tomcat 服务

![open-server.png](/images/eclipse-tomcat-lanuch-error/open-server.png)

会看到如下所示的配置页面：
   
![open-server.png](/images/eclipse-tomcat-lanuch-error/config-error.png)

可以看到当前`Server Locations`选择的是 `Use workspace metadata(does not modify Tomcat installion)`

（2）如果该tomcat中部署了项目的话，这红圈中的选项会灰掉不能修改，要修改必须得先把tomcat中的部署的服务都移除。如下图所示:

![open-server.png](/images/eclipse-tomcat-lanuch-error/add-remove.png)

选择 `Add and Remove`，在弹出的对话框中移除已部署的项目。移除完确定后，将看到上面的选项面板部分可编辑了

（3）选择tomcat的安装目录来作为项目的发布目录

选择`Use tomcat installation(Task control of Tomcat installation)` 即选择tomcat的安装目录来作为项目的发布目录。

 （4） 修改部署文件夹的名称

然后,下来四行,看到`"Deploy Path"`了没?它后面的值默认是"`wtpwebapps`",把它改成"webapps",也就是tomcat中发布项目所在的文件夹名字。

修改后关掉该页面，保存配置。这样就将项目部署到了 `Tomcat` 安装目录下的 `webapp`
重启 `Tomcat` 服务器，访问`http://localhost:8080`则能正常访问了，自己部署的项目也能正常访问了。

### 参考资料

1、[eclipse启动tomcat无法访问](http://blog.csdn.net/wqjsir/article/details/7169838/)

2、[Eclipse官网](https://www.eclipse.org/)
