---
layout: post
title: "如何创建一个私有的Gem库"
date: 2018-03-12 19:29:36 +0800
comments: true
tags: Ruby
---

近期看了下[Cocoapods](https://github.com/CocoaPods/CocoaPods)的一部分代码，结合之前做的iOS项目脚手架工具，突发奇想能不能做一个内部的工具库呢（类似于Cocoapods）。

首先要解决的问题就是怎么样发布自己写的工具库，有没有类似于[RubyGems](https://rubygems.org/)这样的托管平台呢？查了一番资料找到了一个开源的项目- [geminabox](https://github.com/geminabox/geminabox)， 可以搭建一个托管的平台。

### 搭建Gem私服

前提条件是需要有Docker的环境，如果没有的话可以参考：[CentOS安装Docker](http://blog.devzeng.com/blog/install-docker-in-centos.html)。

#### (1) 创建并运行容器

```
docker run -d -v /home/docker/geminabox:/webapps/geminabox/data --name geminabox -p 9292:9292 -P -h geminabox spoonest/geminabox:latest
```

#### (2) 安装geminabox客户端

```
gem install geminabox
```

> PS : 如果是MacOS自带ruby环境。Ruby的版本管理可以使用RVM工具

安装完成之后访问: `http://ip:9292`即可打开：

![geminabox.png](/images/create-a-private-gem/geminabox.png)

### Gem模块开发与发布

#### (1) 创建模板工程

使用`bundler gem [name]`命令可以一键创建项目的结构，相当的方便：

```
⇒  bundler gem demo

Creating gem 'demo'...
MIT License enabled in config
      create  demo/Gemfile
      create  demo/lib/demo.rb
      create  demo/lib/demo/version.rb
      create  demo/demo.gemspec
      create  demo/Rakefile
      create  demo/README.md
      create  demo/bin/console
      create  demo/bin/setup
      create  demo/.gitignore
      create  demo/LICENSE.txt
Initializing git repo in /Users/zengjing/demo/demo
```

如果是第一次使用可能会有些提示问题需要填写。

#### (2) 编译打包

发布gem库需要先进行打包，使用`gem build [gemspecname]`命令可以打包：

```
⇒  gem build demo.gemspec

  Successfully built RubyGem
  Name: demo
  Version: 0.1.0
  File: demo-0.1.0.gem
```

出现`Successfully built RubyGem`表示编译打包完成。

(3) 上传

接下就是之前安装的`geminabox`派上用场的时候了，使用`gem inabox [gemfile]`可以一键上传。

```
gem inabox [gemfile]
```

### 使用

在Gemfile中加入如下代码:

```
source "http://192.168.3.18:9292" do
	gem 'demo', '~> 0.1.0'
end
```

然后安装

```
bundle install
```

### 参考资料

1、[如何开发一个自己的 RubyGem？](http://code.oneapm.com/ruby/2015/07/02/how-to-create-a-gem/)

2、[geminabox](http://tomlea.co.uk/posts/gem-in-a-box/)