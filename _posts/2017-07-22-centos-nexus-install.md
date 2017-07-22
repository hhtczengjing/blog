---
layout: post
title: "Maven私有库Nexus的安装和使用"
date: 2017-07-22 00:08
comments: true
tags: Note
---

在进行Java开发的时候，通常会使用[Maven](http://maven.apache.org/)进行第三方库的管理，类似于iOS中的Cocoapods。我们在使用Cocoapods的时候都感受过更新索引库Specs的痛苦，使用Maven的时候依赖库也是从中央库(Central Repository)下载，速度可想而知会很慢。另外如果我们内部开发了一些基础的工具库，又不太方便托管到Central Repository的时候怎么办呢？

![thenexus-header-final2.jpg](http://www.sonatype.org/nexus/content/uploads/2014/09/thenexus-header-final2.jpg)

参考Cocoapods我们可以搭建内部的私有库来解决这些问题。[Nexus](http://www.sonatype.org/nexus/) 是Maven仓库管理器，如果你使用Maven，你可以从Maven中央仓库 下载所需要的构件（artifact），但这通常不是一个好的做法，通常会在本地架设一个Maven仓库服务器，在代理远程仓库的同时维护本地仓库，以节省带宽和时间，Nexus就可以满足这样的需要。这里记录下使用Nexus来搭建Maven的私有库的过程，本文基于`CentOS 6.5`。

### 手动安装Nexus 2.0

#### 1、下载nexus

到官网下载最新版本的nexus，下载地址`http://www.sonatype.org/nexus/archived/`，本文使用的版本是`Nexus 2.12.0-01`

```
wget http://download.sonatype.com/nexus/oss/nexus-2.12.0-01-bundle.tar.gz
```

#### 2、创建一个nexus文件夹将所有的东西放在下面

```
mkdir -p /home/nexus
```

#### 3、解压

```
tar zxvf nexus-2.12.0-01-bundle.tar.gz
```

将解压后的文件拷贝到刚才创建的nexus目录下面

#### 4、配置防火墙

nexus的默认端口是`8081`, 如果启动后无法正常访问需要配置下防火墙策略：

```
iptables -I INPUT -p tcp --dport 8081 -j ACCEPT
/etc/init.d/iptables save
service iptables restart
```

#### 5、启动

```
cd /home/nexus/nexus-2.12.0-01/bin
./nexus start
```

启动成功后访问地址：`http://localhost:10081/nexux/`。

### nexus配置说明

#### 1.Repository类型

关于Repository的类型有如下几种：

- group: 仓库组，用来合并多个hosted/proxy仓库，通常我们配置maven依赖仓库组
- hosted：本地仓库，通常我们会部署自己的构件到这一类型的仓库。
- proxy：代理仓库，它们被用来代理远程的公共仓库，如maven中央仓库。
- virtual: 虚拟组，暂未使用

#### 2.默认Repository

nexus会默认创建如下几种Repository：

- 1、`Public Repositories`, 这是一个Repository Group，该Repository  Group包含了多个Repository，其中包含了Releases、Snapshots、ThirdParty和Central。
- 2、`3rd party`，该Repository即是存放你公司所购买的第三方软件库的地方，它是一个由Nexus自己维护的一个Repository。 
- 3、`Apache Snapshots`，这是一个代理Repository，即最终的依赖还是得在Apache官网上去下载，然后缓存在Nexus中。
- 4、`Central`，这就是代理Maven Central Repository的Repository。
- 5、`Releases`，你自己的项目要发布时，就应该发布在这个Repository，他也是Nexus自己维护的Repository，而不是代理。
- 6、`Snapshots`，你自己项目Snapshot的Repository。

### 使用Docker安装Nexus 3.0

#### 1、安装Docker

参考[《CentOS安装Docker》](http://blog.devzeng.com/blog/install-docker-in-centos.html)

#### 2、安装Nexus

##### （1）下载镜像

```
docker pull sonatype/nexus3
```

##### （2）创建nexus的数据存储目录

创建存储文件目录，并修改目录拥有者，容器里面运行的uid是`200`

```
mkdir -p /opt/nexus-data
chown -R 200 /opt/nexus-data
``` 

##### （3）创建并启动服务

```
docker run -d -p 8081:8081 --name nexus3 -v /opt/nexus-data:/nexus-data sonatype/nexus3
```

参数说明：

- `-d`：指的是后台运行
- `-p 8081:8081`：指把容器`8081`端口映射到主机`8081`端口，格式为`“主机端口:容器端口”`
- `-v /opt/nexus-data:/nexus-data`：把容器里的`/nexus-data`目录，映射到主机的`/opt/nexus-data`目录
- `–-name nexus3`：容器的名称
- `sonatype/nexus3`：需要下载的镜像

### 配置Maven

#### 1、Mac安装Maven

```
brew install maven
```

#### 2、修改配置文件

安装完成后Maven的配置文件在：`/usr/local/Cellar/maven/{版本号}/libexec/conf`这个目录下面。找到`settings.xml`文件在`mirrors`节点下面新增如下配置:

```
<mirror>
	<id>nexus</id>
	<mirrorOf>central</mirrorOf>
	<name>central repository</name>
	<url>http://192.168.3.18:8081/repository/maven-public/</url>
</mirror>
```
表示新增了一个镜像配置，第三方库从这个镜像地址下载。

#### 3、编写Maven示例程序

使用Maven的模板快速创建一个webapp的项目：

```
mvn archetype:generate -DgroupId=com.devzeng.demo -DartifactId=demo-app -DpackageName=com.devzeng.demo -Dversion=1.0
```

如果在控制台看到有jar包从`http://192.168.3.18:8081`这个地址下载表示配置成功。

### 参考资料

1、[《Nexus私服使Maven更加强大》](http://blog.csdn.net/liujiahan629629/article/details/39272321)

2、[《CentOS上搭建私有maven仓库，提供jcenter镜像》](http://www.chengyong.net/linux-study/centos-install-sonaType-nexus.html)

3、[《CentOS安装nexus(Maven仓库管理器)》](http://blog.csdn.net/typa01_kk/article/details/49228873)

4、[《Centos 基础开发环境搭建之Maven私服nexus》](http://www.cnblogs.com/dingyingsi/p/3776557.html)

5、[《在 Docker 搭建 Maven 私有库》](http://beyondvincent.com/2016/09/23/2016-09-23-use-nexus-with-docker/)

6、[《使用sonatype nexus搭建jcenter mirror》](https://zhuanlan.zhihu.com/p/27660819)

7、[《Docker 搭建Nexus OSS3私服》](https://wendyeq.me/2016/11/20/nexus-oss-3-in-docker/)

8、[《Nexus入门指南（图文）》](http://juvenshun.iteye.com/blog/349534)