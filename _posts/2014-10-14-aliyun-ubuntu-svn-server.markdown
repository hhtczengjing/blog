---
layout: post
title: "阿里云Ubuntu系统搭建SVN服务器"
date: 2014-10-14 23:39:09 +0800
comments: true
tags: Note
---

近期入手了一台阿里云（阿里云是国内优秀的云计算服务提供商，属于阿里巴巴旗下）的云服务器（Ubuntu系统）打算用来做WEB API的服务器，另外为了便于对数据和相关文档的管理需要在服务器上搭建SVN服务器，本文主要是用于记录如何搭建SVN服务器以及在搭建过程中遇到的一些问题。

![aliyun_info.png](/images/aliyun_ubuntu_svn/aliyun_info.png)

##SVN服务器相关软件安装

1、使用SSH远程服务器

（1）对于MAC OS/Liunx的用户直接打开终端输入

`ssh 用户名@实例名`，例如 `ssh root@192.168.1.100`

执行上面的命令后终端会提示输入密码，验证通过后会出现如下信息：

```
Welcome to Ubuntu 12.04.5 LTS (GNU/Linux 3.2.0-67-generic x86_64)

 * Documentation:  https://help.ubuntu.com/
New release '14.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Welcome to aliyun Elastic Compute Service!

Last login: Tue Oct 14 21:31:10 2014
```

(2)Windows的用户需要按照系统的要求安装指定的软件或者是直接使用WEB的终端进行访问


2、安装软件

依次在终端中执行下面的命令安装如下三个软件包：

（1）`sudo apt-get install subversion`

（2）`sudo apt-get install apache2`

（3）`sudo apt-get install libapache2-svn`

其中`subversion`是SVN必须的，apache2和libapache2-svn是为了配置SVN服务器支持通过HTTP访问


##SVN服务器配置

1、相关用户、组的设定

`sudo addgroup subversion`

`sudo usermod -G subversion -a www-data`

2、创建项目的目录

`sudo mkdir /home/svn`

3、配置Subversion

（1）配置dav_svn.conf文件

`vim /etc/apache2/mods-available/dav_svn.conf`

按照下面的步骤进行配置即可

<1>删除`<Location /svn>`和`DAV svn`这两行前面的注释

![aliyun_ubuntu_svn_001.png](/images/aliyun_ubuntu_svn/aliyun_ubuntu_svn_001.png)

<2>删除`SVNParentPath ...`前面的注释，并且把后面的路径替换成创建的SVN的项目路径`/home/svn`

![aliyun_ubuntu_svn_002.png](/images/aliyun_ubuntu_svn/aliyun_ubuntu_svn_002.png)

<3>删除AuthType Basic、AuthName "Subversion Repository"前面的注释，去掉AuthUserFile所在行前面的注释，并且修改后面的路径为`/etc/subversion/dav_svn.passwd`，去掉AuthzSVNAccessFile所在行前面的注释，并且修改后面的路径为`/etc/subversion/dav_svn.authz`,删除Require valid-user和</Location>前面的注释，具体如下所示：

![aliyun_ubuntu_svn_003.png](/images/aliyun_ubuntu_svn/aliyun_ubuntu_svn_003.png)

其中：

`/etc/subversion/dav_svn.passwd`文件是SVN用户名和密码的配置，指定基本用户验证的密码文件存放的位置

`/etc/subversion/dav_svn.authz`文件是访问权限配置

（2）重启Apache 2 WEB服务器

`sudo /etc/init.d/apache2 restart`

4、创建SVN文件仓库

（1）创建指定的项目存放路径

`cd /home/svn`

`mkdir project1`

（2）更改版本库所示的用户、组

`sudo chown -R root:subversion project1`

（3）创建SVN仓库

`sudo svnadmin create /home/svn/project1`


（4）赋予组成员对所有新加入文件仓库的文件拥有相应的权限

`sudo chmod -R g+rws project1`

5、用户和权限配置

（1）创建dav_svn.passwd文件并添加用户admin001，执行命令后会提示要输入密码

`sudo htpasswd -c /etc/subversion/dav_svn.passwd admin001`

继续添加新用户，去掉-c，否则会覆盖本文件

`sudo htpasswd /etc/subversion/dav_svn.passwd admin002`

（2）创建授权文件

`vim /etc/subversion/dav_svn.authz`

里面加入project1的权限配置，如

```
[groups]
administrator=admin001,admin001

[project1:/]
@administrator=rw
```

6、重启SVN服务器

`killall svnserve`

`svnserve -d -r /home/svn/`

至此SVN服务器搭建完成，可以在终端执行

`svn checkout http://hostname/svn/project1 project1 --username 用户名 --password 密码`
来checkout项目了

##相关问题

1、Apache和Tomcat端口号冲突

（1）修改`/etc/apache2/ports.conf`文件将`Listen 80`改成`Listen 8080`表示监听8080端口或者其他可用端口

（2）修改`/etc/apache2/sites-enabled/000-default`文件，修改`<VirtualHost*:80>`改成`<VirtualHost*:8080>`

2、关闭和启动Tomcat服务器

（1）关闭：`sudo /etc/init.d/tomcat stop`

（2）启动：`sudo /etc/init.d/tomcat start`

##参考资料

1、[《关于阿里云》](http://www.aliyun.com/about/?spm=5176.383338.25.1.MVser7)

2、[《UBUNTU SVN 服务器配置》](http://www.cnblogs.com/ouuy/archive/2012/04/27/2473706.html)

3、[《阿里云ubuntu 创建svn服务器》](http://www.cnblogs.com/likwo/p/3152365.html)

4、[《Linux SVN 命令详解》](http://www.cnblogs.com/xulb597/archive/2012/07/02/2573575.html)
