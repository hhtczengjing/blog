---
layout: post
title: "Windows环境下OpenSSL的编译和使用"
date: 2017-04-08 11:00:00 +8000
comments: true
tags: C++
---

[OpenSSL](https://www.openssl.org/)是目前使用的非常广泛的加密算法库，基本上我们日常使用到的HTTPS、SSH都离不开它的身影。本文就在Windows环境下面编译最新版本的OpenSSL的步骤进行整理。

![oepnssl-logo.png](/images/openssl-windows-compile/oepnssl-logo.png)

### 编译OpenSSL

#### 1.编译环境准备

##### (1) perl

OpenSSL的编译需要使用到perl的环境，如果之前安装过可以跳过此步骤。

1) 下载perl安装包

根据操作系统的版本下载对应最新版本的perl（当前最新的版本是`5.22.3.2204`），下载地址是：`https://www.activestate.com/activeperl/downloads`。

![windows-perl-download-page.png](/images/openssl-windows-compile/windows-perl-download-page.png)

2) 配置环境变量

前往“`计算机` -> `右键-属性` -> `高级系统设置` -> `环境变量`”将`C:\Perl\site\bin;C:\Perl\bin;`（Perl安装路径）添加到环境变量（如果前面有其他的配置使用`;`进行拼接）

配置完成之后再cmd中输入`perl -version`,如果正确输出如下信息表示成功安装。

```
This is perl 5, version 22, xxxxxx
```

##### (2) openssl

前往OpenSSL的[官网](https://www.openssl.org/)下载最新最新版本的源码（当前最新的版本是`openssl-1.1.0e`），下载完成之后解压到D盘。下载界面如下：

![](/images/openssl-windows-compile/openssl-source-download-page.png)

##### (3) IDE安装

本文使用的是`Visual Studio 2010`版本

#### 2.编译OpenSSL

打开命令行工具，cd到OpenSSL源码所在路径。

##### (1) 配置编译模式

```
perl Configure VC-WIN32 no-asm --prefix=d:\openssl_lib
```

说明：

- Configure 后面的选项可选值有` VC-WIN32(32位) | VC-WIN64A(64位AMD) | VC-WIN64I(64位Intel) | VC-CE(Windows CE)`
- prefix: 表示生成的lib文件存放路径

##### (2) 编译生成

1) 编译源码

```
nmake
```

2) 测试

```
nmake test
```

3) 生成可执行文件

```
nmake install
```

执行完成上面的三个步骤之后在`d:\openssl_lib`这个目录下面会生成四个文件夹（include/lib/bin/html)：

- `include`目录下面存放的shi头文件
- `lib`目录是生成的静态库文件,文件的后缀是`.lib`
- `bin`目录下面存放的是dll文件和exe文件
- `html`目录下面存放的是文档

### OpenSSL的简单使用

#### 1.注册dll文件

执行下面两个步骤实现dll文件注册：

1) 将bin目录下面的`libcrypto-1_1.dll`文件拷贝到`C:\Windows\System32`目录下面

2) 在运行(`win+R`)中输入`regsvr32 libcrypto-1_1.dll`

#### 2.创建示例项目

使用`Visual Studio 2010`创建一个C++的`CLR命令行控制台程序`。

#### 3.配置OpenSSL依赖

需要配置两个内容包含目录和库目录，`项目名称右键 -> 配置属性 -> VC++目录`按照下面的配置方式进行配置：

![visual-studio-config.png](/images/openssl-windows-compile/visual-studio-config.png)

#### 4.示例代码

下面以SHA256加密算法为例进行测试

##### (1) 头文件

```
#include <openssl/sha.h>
```

##### (2) 链接库

```
#pragma comment(lib, "libcrypto.lib")
```

##### (2) 示例代码

```
void sha256(char* string, char outputBuffer[64])
{
    unsigned char hash[SHA256_DIGEST_LENGTH];
    SHA256_CTX sha256;
    SHA256_Init(&sha256);
    SHA256_Update(&sha256, string, strlen(string));
    SHA256_Final(hash, &sha256);
    int i = 0;
    for(i = 0; i < SHA256_DIGEST_LENGTH; i++)
    {
        sprintf(outputBuffer + (i * 2), "%02x", hash[i]);
    }
}
```

### 参考资料

1、[VS2010中编译openssl的步骤和使用设置](http://blog.163.com/xiaoting_hu/blog/static/50464772201310415042524/)

2、[OpenSSL在Windows下的编译安装](http://blogger.org.cn/blog/more.asp?name=OpenSSL&id=18972)

3、[在 Windows下用 Visual Studio 编译 OpenSSL 1.1.0](http://www.cnblogs.com/chinalantian/p/5819105.html)

4、[OpenSLL](https://github.com/openssl/openssl)

5、[在VS2010项目中引用Lib静态库（以Openssl为例）](http://kb.cnblogs.com/page/94467/)
