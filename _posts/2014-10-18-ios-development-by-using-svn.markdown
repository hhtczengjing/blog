---
layout: post
title: "iOS开发中SVN管理工具的使用"
date: 2014-10-18 15:58:25 +0800
comments: true
tags: iOS
---

SVN是Subversion的简称，是一个开放源代码的版本控制系统，相较于RCS、CVS，它采用了分支管理系统，它的设计目标就是取代CVS。互联网上很多版本控制服务已从CVS转移到Subversion。

![tortoisesvn_logo.png](/images/ios_svn/tortoisesvn_logo.png)

##SVN命令行工具

1、从本地导入代码到服务器(第一次初始化导入)，在终端中输入：

`svn import /Users/apple/Documents/workspace/project svn://hostname/svn/project --username=mj --password=123 -m "初始化导入"`

说明：将/Users/apple/Documents/workspace/project中的所有内容，上传到服务器svn仓库的project目录下，后面双引号中的"初始化导入"是注释
 
2、从服务器端下载代码到客户端本地，在终端中输入：

`svn checkout svn://hostname/svn/project --username=用户名 --password=密码 /Users/apple/Documents/code`

说明：将服务器中svn仓库中的project目录下的内容下载到/Users/apple/Documents/code目录中
 
3、提交更改过的代码到服务器

在步骤2中已经将服务器端的代码都下载到/Users/apple/Documents/code目录中，现在修改下里面的一些代码，然后提交这些修改到服务器

（1）打开终端，先定位到/Users/apple/Documents/code目录，输入：

`cd /Users/apple/Documents/code`

（2）输入提交指令：

`svn commit -m "修改了文件"`

执行上面的命令会将/Users/apple/Documents/code下的所有修改都同步到服务器端。
 
4、更新服务器端的代码到客户端

这个应该是最简单的指令了，在终端中定位到客户端代码目录后，比如上面的/Users/apple/Documents/code目录，然后再输入指令：`svn update`
 
获取SVN的其他用法，可以在终端输入：`svn help`

使用命令行的工具操作SVN确实不太方便，因此下面介绍在MAC下比较优秀的SVN客户端的使用方法：

##Cornerstone客户端的使用

1、从SVN服务器导出项目

（1）打开Cornerstone客户端，在右边的界面中选择Add Respository的按钮，如下图所示：

![cornerstone_client_001.png](/images/ios_svn/cornerstone_client_001.png)

（2）填写项目SVN的路径信息

如果项目的SVN的路径是`http://hostname:port/...`形式的话，就选择HTTP Server,如下图所示：

![cornerstone_client_002.png](/images/ios_svn/cornerstone_client_002.png)

其中上面的Protocol可以选择是使用HTTP还是HTTPS。

如果项目的SVN的路径是`svn://hostname:port/...`形式的话，就选择SVN Server，如下图所示：

![cornerstone_client_003.png](/images/ios_svn/cornerstone_client_003.png)

说明：

Server：指的是服务的名称，如IP地址或者是域名

Port:指的是端口号

Repository：指的的项目SVN地址的相对路径，如项目的地址是`http://192.168.1.100/svn/project`,那么对应的Repository为svn/project

（3）导出项目到本地

选择左上角的Check out,如下图所示：

![cornerstone_client_004.jpg](/images/ios_svn/cornerstone_client_004.jpg)

然后会提示需要选择导出到指定的路径信息：

![cornerstone_client_005.jpg](/images/ios_svn/cornerstone_client_005.jpg)

Check Out As:指明项目导出的文件夹的名称

Where:指的是本地存储的目录，另外下面可以选择导出的SVN版本的格式，填写完成后点击Check Out即可。

2、提交修改到SVN服务器或者更新代码到本地

前提条件是已经导出项目到本地，提交代码时选择Commit，更新代码选择Update即可，如下：

![cornerstone_client_006.png](/images/ios_svn/cornerstone_client_006.png)

3、其他说明

（1）如果提交代码的时候部分文件或者文件夹的状态是一个`?`的图标，需要单击右键选择`Add to working copy`添加到SVN的管理然后再提交。

（2）可以通过查看Log来比较不同版本的修改情况。

##SmartSVN客户端的使用

1、从SVN服务器导出项目到本地

（1）打开SmartSVN，在启动界面中选择`Check out project from repository`,如下所示：

![smartsvn_client_001.png](/images/ios_svn/smartsvn_client_001.png)

（2）填写SVN路径的地址，和选择需要导出到的路径，如下图所示：

![smartsvn_client_002.png](/images/ios_svn/smartsvn_client_002.png)

确定后会提示需要输入用户名和密码，然后一直点击Finish就可以导出项目到本地了。

2、更新/提交代码

关于如何提交和更新代码，可以选择指定的项目然后点击左上角的Commit和Update，对应提交本地修改的代码到服务器和从服务器更新最新的代码：

![smartsvn_client_003.jpg](/images/ios_svn/smartsvn_client_003.jpg)

##参考资料

1、[《Mac环境下svn的使用》](http://blog.csdn.net/q199109106q/article/details/8655204)

2、[《用CornerStone配置SVN，HTTP及svn简单使用说明》](http://blog.csdn.net/xiaohulunb/article/details/20627995)
