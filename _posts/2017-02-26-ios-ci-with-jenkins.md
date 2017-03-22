---
layout: post
title: "iOS中使用Jenkins搭建持续集成环境"
date: 2017-02-25 11:00:00 +8000
comments: true
tags: iOS
---

在持续集成(Continuous integration，简称CI)这块，[Jenkins](https://jenkins.io)无疑是目前使用的比较多的一个开源框架。本文就如何快速搭建一个iOS的持续集成环境进行介绍。

![jenkins-logo.png](/images/ios-jenkins/jenkins-logo.png)

### Jenkins安装

系统要求：必须安装JDK 1.5以上版本，推荐安装最新版本的JDK。可以通过`java -version`查看是否安装JDK。

```
$ java -version 
java version "1.8.0_91"
Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)
```

#### 1、下载Jenkins

到官网`https://jenkins.io`选择下载最新版本的war包，推荐下载LTS(Long-Term Support，稳定版本)版本。

![jenkins-download.png](/images/ios-jenkins/jenkins-download.png)

#### 2、运行Jenkins

下载完成之后只有一个文件`jenkins.war`，运行Jenkins相当的方便，可以通过命令行直接运行，也可以将war包丢在Tomcat的webapp目录下面。

在测试阶段可以使用命令行方式进行启动，在终端执行

```
java -jar jenkins.war
```
默认的端口号是8080，如果需要指定其他端口号可以使用如下方式(示例指定了9999端口号)

```
java -jar jenkins.war --httpPort=9999  
```

### 配置Jenkins

第一次运行启动`Jenkins`，在浏览器打开`http://localhost:9999`，会出现如下界面，提示需要填写指定路径文件里面的内容（该内容也可以在终端上面看到）。

![unlock-jenkins.png](/images/ios-jenkins/unlock-jenkins.png)

输入完成之后点击`continue`进入到插件安装页面，为了避免后续出现一些问题建议选择安装推荐的插件(`install suggested plugins`).

![customize-jenkins.png](/images/ios-jenkins/customize-jenkins.png)

选择安装推荐的插件(`install suggested plugins`)后会出现安装进度界面，如下图所示：

![suggested-plugin-install.png](/images/ios-jenkins/suggested-plugin-install.png)

插件安装完成之后就可以创建管理员用户了

![create-admin-account.png](/images/ios-jenkins/create-admin-account.png)

全部做完之后就可以愉快的使用了

![install-success.png](/images/ios-jenkins/install-success.png)

### 配置slave

公司目前有几台Mac Mini，另外考虑到后续希望Android、Java Web开发的同事也能接入进来，目前是将Jenkins部署在CentOS上面，通过配置Slave的方式将Windows/MacOS/Linux进行统一管理，实现iOS、Android项目各自使用指定的节点。

点击`系统管理` -> `管理节点`进入到节点管理界面，可以查看和管理目前系统配置的所有节点。

![node-manage.png](/images/ios-jenkins/node-manage.png)

#### 创建节点

（1）选择“新建节点”的菜单按钮，进入到节点的创建界面。

![create-slave-node-1.png](/images/ios-jenkins/create-slave-node-1.png)

（2）填写节点的一些基本信息

![create-slave-node-2.png](/images/ios-jenkins/create-slave-node-2.png)

说明：标签这个字段比较重要，这个字段用于识别是哪一个节点，在配置项目的时候会用到

### 使用

正在完善中...

#### 1、常见问题

（1）和fastlane一起使用的时候报字符集错误

```
export LANG=en_US.UTF-8
```

（2）执行脚本的时候报错

```
security unlock-keychain "-p" "YOUR_KEYCHAIN_PWD" "/Users/YOUR_NAME/Library/Keychains/login.keychain" 
```

### 参考资料

1、[持续集成是什么？](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)

2、[Jenkins 官网](https://jenkins.io)
