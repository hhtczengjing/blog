---
layout: post
title: "使用Kettle导入Excel数据"
date: 2017-08-20 10:08
comments: true
tags: Note
---

ETL（Extraction, Transformation, and Loading），在日常的工作中我们经常会遇到各种数据的处理，转换，迁移。比如将Excel的数据导入到数据库，将SQLServer里面的数据转换后存到Oracle，将数据库的数据提取到文本等。

最开始都是使用写代码然后进行处理，多了几次之后就觉得麻烦了。后来了解到[Kettle](http://community.pentaho.com/projects/data-integration/)这个工具，首先无需安装直接就能使用，支持图形化的GUI设计界面，然后可以以工作流的形式流转，在做一些简单或复杂的数据抽取、质量检测、数据清洗、数据转换、数据过滤等方面有着比较稳定的表现，通过熟练的使用能在数据处理方面减少不少的工作量，提高工作效率。

![kettle-logo.png](/images/load-excel-data-with-kettle/kettle-logo.png)

### 下载安装

Kettle是使用Java编写的，所以需要安装Java的运行环境。Kettle支持跨平台能在各种系统下使用，下面以在MacOS上面为例介绍如何配置。

#### 1、环境准备

在命令行执行`java -version`，查看当前是否安装JDK。如果出现如下的内容：

```
java version "1.8.0_91"
Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)
```

表示Java环境已经安装，如果没有的话需要先安装JDK。

#####（1）下载安装JDK

到Oracle的官网上下载最新的版本JDK，网址如下：

```
http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
```

选择需要的版本下载即可，下载完成直接双击下一步安装：

![jdk-download.png](/images/load-excel-data-with-kettle/jdk-download.png)

##### （2）配置环境变量

安装完成JDK之后，一般会自动安装在`/Library/Java/JavaVirtualMachines/`这个目录下面，可以通过`/usr/libexec/java_home`这个查看路径。

```
vim ~/.bash_profile
```

在里面新增如下记录(当前安装的JDK版本是1.8)：

```
export JAVA_8_HOME=`/usr/libexec/java_home -v 1.8`
export JAVA_HOME=$JAVA_8_HOME
```

保存，然后使用`source ~/.bash_profile`。

#### 2、下载

打开Kettle官网：`http://community.pentaho.com/projects/data-integration/` 在网页的下面有下载的入口，当前的稳定版本是`7.1`:

![kettle-download.png](/images/load-excel-data-with-kettle/kettle-download.png)

#### 3、启动

下载完成后解压压缩包，在命令行进入到kettle的目录（文件夹的名字一般是`data-integration`）。然后执行`./spoon.sh`即可启动Kettle。

![hello-kettle.png](/images/load-excel-data-with-kettle/hello-kettle.png)

### 使用示例

近期工作中经常需要协助同事将Excel里面的数据进行处理之后保存到SQLServer数据里面去，最开始是使用Python脚本解析Excel然后生成SQL语句到服务器上面执行的，每次有字段的调整都要改代码（代码如下）：

```
def parse_excel_file(file, index=1):
    workbook = xlrd.open_workbook(file)
    sheets = workbook.sheets()
    sheets_data = []
    for sheet in sheets:
        print "[解析]", sheet.name
        sheet_data = []
        for i in range(index, sheet.nrows):
            sheet_row_data = []
            for j in range(0, sheet.ncols):
                cell = sheet.cell(i, j)
                # 数据类型 0 empty,1 string, 2 number, 3 date, 4 boolean, 5 error
                ctype = cell.ctype 
                # 数据转换
                cvalue = cell.value
                if ctype == 2:
                    if isinstance(cvalue, float):
                        cvalue = str(long(cvalue))
                    elif isinstance(cvalue, int):
                        cvalue = str(cvalue)
                elif ctype == 1:
                    cvalue = cvalue.encode('UTF-8')
                sheet_row_data.append(cvalue)
            sheet_data.append(sheet_row_data)
        sheets_data.append(sheet_data)
    return sheets_data
```

下面以如何使用Kettle通过配置的方式简化操作，具体的操作步骤如下：

#### 1、创建数据转换

执行`./spoon.sh`后启动Kettle的界面，在`主对象树` -> `转换`菜单上面单击右键新建转换。

![kettle-demo-01.png](/images/load-excel-data-with-kettle/kettle-demo-01.png)

#### 2、配置数据转换

#####（1）从左边的`核心对象`里面选中控件直接拖到右边的区域，各个控件之间可以用箭头连起来(按住Shift直接拖即可)

![kettle-demo-03.png](/images/load-excel-data-with-kettle/kettle-demo-03.png)

#####（2）双击Excel输入配置解析

![kettle-demo-04.png](/images/load-excel-data-with-kettle/kettle-demo-04.png)

1) 点击`浏览`选择Excel文档的路径(xls格式），选择完成后点击`增加`。

2）选择`工作表`的选项卡，在此页面配置要解析的Sheet和起始行列信息。

![kettle-demo-05.png](/images/load-excel-data-with-kettle/kettle-demo-05.png)

3）选择`字段`的选项卡，在此页面点击`获取来自头部数据的字段`，Kettle会自动获取表头生成字段信息。

![kettle-demo-06.png](/images/load-excel-data-with-kettle/kettle-demo-06.png)

#####（3）使用JavaScript代码自动生成主键

由于业务方需要在将数据保存到数据库的时候需要指定一个主键，这里可以直接使用JavaScript的代码来自动生成一个UUID作为主键。

![kettle-demo-07.png](/images/load-excel-data-with-kettle/kettle-demo-07.png)

#####（4）输出数据到目标数据库

1）配置数据源

![kettle-demo-08.png](/images/load-excel-data-with-kettle/kettle-demo-08-1.png)

配置完成之后点击测试，如果报错(缺少驱动包)，需要下载对应数据库的驱动包放到Kettle目录下面的`lib`目录下。

![kettle-demo-08.png](/images/load-excel-data-with-kettle/kettle-demo-08-1-error.png)

2）配置目标表和字段映射

配置好数据源后需要配置目标表的信息，如表名、数据库字段对应等。

![kettle-demo-08-2.png](/images/load-excel-data-with-kettle/kettle-demo-08-2.png)

点击输入字段映射可以将输入的数据字段和目标表的数据库字段进行一一对应起来。

#### 3、运行

配置完成后点击左上角的运行按钮直接就可以运行转换任务。

![kettle-demo-run.png](/images/load-excel-data-with-kettle/kettle-demo-run.png)

### 参考资料

1、[Kettle 官网](http://community.pentaho.com/projects/data-integration/)

2、[kettle转换中使用javascript例子整理（1）](http://blog.csdn.net/man_earth/article/details/39525651)

3、[kettle JavaScript脚本](http://www.cnblogs.com/melodyluo/p/3374382.html)
