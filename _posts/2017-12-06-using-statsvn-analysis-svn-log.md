---
layout: post
title: "使用StatSVN对SVN日志进行可视化分析"
date: 2017-12-06 13:08
comments: true
tags: Note
---

[StatSVN](http://www.statsvn.org/)是一个开源的SVN统计工具(Java语言编写，最近一次更新是`2010-01-01`)，能够从Subversion版本库中取得信息，然后生成描述项目开发的各种表格和图表（StatSVN生成的报表是一组包括表格与图表的静态HTML文档）。比如：

- 代码行数的时间线；
- 针对每个开发者的代码行数；
- 开发者的活跃程度；
- 开发者最近所提交的；
- 文件数量；
- 平均文件大小；
- 最大文件；
- 哪个文件是修改最多次数的；
- 目录大小；
- 带有文件数量和代码行数的Repository tree等。

### 使用步骤

#### 1、下载安装

到官网的`http://www.statsvn.org/downloads.html`下载最新版本(`v0.7.0`)。下载完之后获取`statsvn.jar`文件。

由于`statsvn.jar`使用的是Java编写的，使用的前提是需要有Java的环境，可以通过`java -version`查看是否安装。

#### 2、获取SVN的日志文件

```
svn log -v --xml SVN_LINK > svn.log
```

如果没有安装SVN需要先安装SVN的客户端。(`SVN_LINK`指的是SVN仓库的链接地址）

#### 2、生成统计文件

将下载的`statsvn.jar`和`svn.log`拷贝到一个单独的文件夹下如`test`,方便进行下面的操作：

```
cd test
java -jar statsvn.jar svn.log LOCAL_PATH -charset utf-8 -output-dir DIST_PATH
```
说明：

- LOCAL_PATH：替换成仓库checkout到本地的路径
- DIST_PATH: 替换成生成的统计报表文件存放路径

更多用法可以到官网去看。执行完成后在`DIST_PATH`下会自动生成一堆HTML文件双击`index.html`可以查看效果：

![demo.png](/images/statsvn-analysis-svn/demo.png)

可以使用自动化的脚本将生成的资源文件放到HTTP服务器上面就能做到在线浏览了。

### 参考资料

1、[StatSVN官网](http://www.statsvn.org)

2、[SVN的可视化日志统计工具StatSVN](https://my.oschina.net/myriads/blog/15665)

3、[使用statsvn统计svn中的代码量](http://chenzhou123520.iteye.com/blog/1436653)