---
layout: post
title: "在Docker上搭建WebDAV文件共享服务"
date: 2019-02-25 18:46:37 +0800
comments: true
tags: Note
---

近期由于一些不可抗力因素导致AirDrop被禁用了，平时对文档或者是一些安装包的共享还是有比较多的需求，在此记录一下使用Docker快速搭建WebDAV环境的过程。

直接在命令行输入下面的命令即可快速完成安装：

```
docker run -d -v /Users/zengjing/docker/webdav:/var/webdav -e USERNAME=test -e PASSWORD=test -p 8888:80 morrisjobke/webdav
```

安装完成后通过浏览器：`http://ip:8888/webdav`进行访问，输入用户名密码`test/test`即可

说明：

- (1) `-v`后面的路径是共享的文件存放的路径，需要提前创建好。可以直接将需要共享的文件拷贝到该目录下面其他人就可以直接进行下载使用

- (2) `USERNAME、PASSWORD`分别是用户名密码

如果其他电脑想要上传文件到共享盘上面可以通过：

`Finder -> 前往 -> 连接服务器`，在对话框中输入: `http://用户名:密码@IP:端口/webdav`就可以将共享盘挂载到本地，然后像访问本地磁盘文件夹一样直接将需要共享的内容复制进去即可。
