---
layout: post
title: "Hello Octopress"
date: 2014-03-01 01:08
comments: true
tags: Note
---

![hello-octopress.png](/images/hello-octopress/hello-octopress.png)

   折腾了这么久由最开始的新浪博客这类，然后到独立的WordPress，到目前的Octopress。因为各种的麻烦（空间、域名、流量、审核）弄得无力打理博客。关于Octopress（A blogging framework for hacker）是基于Jekyll博客引擎开发的一个博客系统，能方便的生成静态页面在Github page上展示。

####1.安装Ruby
   Octopress需要Ruby环境的支持，而且据官方说明最少需要在1.9.3以上（经过验证最好是在1.9.3，本人在Mac OS X 10.9上试了很长时间发现2.0都安装不成功）。为了比较方便安装和管理Ruby的环境，在这里使用了RVM(Ruby Version Manager)
#####(1)安装RVM
在终端输入如下命令即可实现安装RVM

```
$ curl -L https://get.rvm.io | bash -s stable --ruby
```
#####(2)安装Ruby 1.9.3
直接使用rvm 的install可安装指定版本的ruby，安装完成后输入ruby --version如果出现1.9.3的字样就表示安装成功了。

```
$ rvm install 1.9.3
```
#####(3)错误说明
安装rvm1.9.3需要gcc46,可以使用ln的命令来创建一个llvm的连接。
   
####2.安装Octopress
  在安装Octopress之前，请确保你的电脑上已经安装有git了，在终端输入git --version，应该可以看到电脑中的git版本(我电脑上输出:git version 1.7.12.4 (Apple Git-37))，如果没有显示相关内容，请先安装git。
######(1)git安装之后，利用git命令将octopress从github上clone到本机，如下命令：
```
$ git clone git://github.com/imathis/octopress.git octopress
$ cd octopress # If you use RVM, You'll be asked if you trust the .rvmrc file (say yes).
```
######（2）接着安装相关依赖项：
```
$ gem install bundler
$ rbenv rehash # If you use rbenv, rehash to be able to run the bundle command
$ bundle install
```
######(3)最后安装默认的Octopress 主题。
```
rake install
```
####3.部署到Github
由于Github提供了Pages的功能，可以显示静态的html,所以需要把Octopress生成的静态html文件提交到GitHub的Master目录下。
#####(1)生成部署的文件
```
$ rake generate #生成静态的HTML文件等，会创建_deploy的文件夹
``` 
#####(2)将部署生成的静态文件到Master
```
$ cd _deploy
$ git init
$ rake generate
$ git add .
$ git commit -m "first commit" 
$ git remote add origin 工程目录
$ git push -u origin master
``` 
#####(3)提交Source到source分支
```
$ cd source
$ git add .
$ git commit -am "Some comment here." 
$ git push origin source
``` 
#####(4)创建新文章更新等
其实如果需要发布新文章，创建部署文件提交到Github，可以使用更为简单的方式，如下：

```
$ rake new_post["New Post"] #创建新文章
$ rake generate
$ git add .
$ git commit -am "Some comment here." 
$ git push origin source
$ rake deploy
``` 
