---
layout: post
title: "使用Anaconda管理Python环境"
date: 2018-01-05 13:08
comments: true
tags: Note
---

Python好用但是在使用过程中发现还是有很多问题的，其中一个就是版本管理（Python2和Python3的切换）。相比于Ruby的版本管理有`rvm`，可以使用`rvm use 2.4.0`这样的命令来快速切换Ruby的版本。

出于历史原因目前还是有很多Python的程序是运行在`Python2.7`，经常需要在Python3的环境下面执行一些实例切换起来非常麻烦，刚好最近了解到Anaconda，Anaconda 是一个可用于科学计算的 Python 发行版，支持 Linux、Mac、Windows系统，内置了常用的科学计算包）解决了官方 Python 的两大痛点：

- 提供了包管理功能，Windows 平台安装第三方包经常失败的场景得以解决，
- 提供环境管理的功能，功能类似 Virtualenv，解决了多版本Python并存、切换的问题。

![python-anaconda](/images/python-anaconda/anaconda-logo.png)

### 安装配置

(1) 下载安装

直接在[官网](https://www.anaconda.com/download/)下载安装包， 选择 Python3.6 的安装包进行下载，下载完成后直接安装，安装过程选择默认配置即可。

国内用户可以从：`https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/` 下载安装包。

(2) 修改镜像地址

可以修改`~/.condarc`文件为如下配置设置下载package的镜像地址：

```
channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
  - defaults
show_channel_urls: true
ssl_verify: true
```

或者使用命令的方式设置：

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --set show_channel_urls yes
```

### conda命令工具使用

conda 是 Anaconda 下用于包管理和环境管理的命令行工具，是 pip 和 vitualenv 的组合。安装成功后 conda 会默认加入到环境变量中，因此可直接在命令行窗口运行 conda 命令

（1）查看帮助

```
conda -h 

usage: conda [-h] [-V] command ...

conda is a tool for managing and deploying applications, environments and packages.

Options:

positional arguments:
  command
    info         Display information about current conda install.
    help         Displays a list of available conda commands and their help
```

（2）创建管理Python环境

```
(1) 基于python3.6版本创建一个名字为python36的环境
conda create --name python36 python=3.6

(2) 激活此环境
source activate python36

(3) 退出当前环境
deactivate python36 

(4) 删除该环境
conda remove -n python36 --all
```

同理如果需要创建Python2.7的环境可以直接创建

```
conda create --name python27 python=2.7
```

使用的时候`source activate python27`即可切换到2.7的版本。

(4) 查看所有安装的环境

```
conda info -e

# conda environments:
#
python27                 /Users/zengjing/Anaconda3/anaconda3/envs/
python36                 /Users/zengjing/Anaconda3/anaconda3/envs/
root                  *  /Users/zengjing/Anaconda3/anaconda3
```

(5) 包管理

conda实现了相当于pip的包管理功能，类似于pip可以直接通过如下方式管理依赖包：

```
# 安装包 
conda install requests

# 查看已安装的包
conda list 

# 包更新
conda update requests

# 删除包
conda remove requests
```

### 参考资料

1、[Anaconda 入门安装教程](https://foofish.net/anaconda-install.html)

2、[用 Anaconda 完美解决 Python2 和 python3 共存问题](https://foofish.net/compatible-py2-and-py3.html)

3、[Anaconda使用总结](https://www.jianshu.com/p/2f3be7781451#)