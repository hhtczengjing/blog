---
layout: post
title: "使用Ant实现iOS项目的自动编译打包"
date: 2014-08-02 00:13:17 +0800
comments: true
tags: iOS
---

Apache Ant，是一个将软件编译、测试、部署等步骤联系在一起加以自动化的一个工具，大多用于Java环境中的软件开发。由Apache软件基金会所提供维护，目前最新的版本是1.9.4。本文主要介绍如何在iOS开发中使用Ant来提高开发效率，重点涉及到ant的安装、build配置文件的基本语法和iOS build脚本等内容。

![ant logo](/images/ant_for_ios/apache_ant_logo.png)

##安装ANT工具

1、到Apache Ant的官网上下载最新的ant工具包

可以直接使用浏览器下载，或者是其他下载工具。[下载地址](http://apache.dataguru.cn//ant/binaries/apache-ant-1.9.4-bin.zip)

也可以使用如下的命令:

`wget http://apache.dataguru.cn//ant/binaries/apache-ant-1.9.4-bin.zip`

2、将下载并解压后的`apache-ant-1.9.4`目录拷贝到`/usr/local`下

`unzip apache-ant-1.9.4-bin.zip`

`cp apache-ant-1.9.4 /usr/local/`

3、设置环境变量

(1)获取root权限，会提示要输入密码

`sudo -s`

(2)修改bashrc文件的读写权限

`chmod +w /etc/bashrc`

(3)用vim打开`/etc/bashrc`按`i`进入编辑,在文件的末尾添加下面两行

`export ANT_HOME=/usr/local/apache-ant-1.9.4`

`export PATH=${PATH}:${ANT_HOME}/bin`

(4)编辑完成后按`ESC`，输入`:wq!`退出vim


3、退出终端重新打开输入`ant -version`,如果能看到`Apache Ant(TM) version 1.9.3`表示安装成功，如果看到`No Java runtime present`这样也表示安装成功，但是需要安装`JAVA JRE`环境.

4、其实也可以通过Homebrew来安装，直接在终端输入`brew install ant`，但是本人从未尝试正常安装每次都是下载失败，如果有兴趣可以试试。

##Ant build.xml说明

   当开始一个新的项目时，首先应该编写Ant构建文件。构建文件定义了构建过程，并被团队开发中每个人使用。Ant构建文件默认命名为 build.xml，也可以取其他的名字。只不过在运行的时候把这个命名当作参数传给Ant。构建文件可以放在任何的位置。一般做法是放在项目顶层目录中，这样可以保持项目的简洁和清晰。
   
   Ant构建文件是XML文件。每个构建文件定义一个唯一的项目(Project元素)。每个项目下可以定义很多目标(target元素)，这些目标之间可以有依赖关系。当执行这类目标时，需要执行他们所依赖的目标。每个目标中可以定义多个任务，目标中还定义了所要执行的任务序列。Ant在构建目标时必须调用所定义的任务。
   
   下面简单介绍下property、project和target标签的基本用法：

1、project标签

   每个构建文件对应一个项目。<project>标签时构建文件的根标签。它可以有多个属性，其各个属性的含义分别如下：
   
(1)default表示默认的运行目标，这个属性是必须的
 
(2)basedir表示项目的基准目录

(3)name表示项目名

(4)description表示项目的描述

2、target标签
    
  一个project可以有多个target标签，另外一个target标签可以依赖其他的target标签。例如，有一个target用于编译程序，另一个target用于声称可执行文件。在生成可执行文件之前必须先编译该文件，因策可执行文件的target依赖于编译程序的 target。Target的所有属性如下：
  
(1)name表示标明，这个属性是必须的

(2)depends表示依赖的目标
 
(3)if表示仅当属性设置时才执行

(4)unless表示当属性没有设置时才执行
 
(5)description表示项目的描述

Ant的depends属性指定了target的执行顺序。Ant会依照depends属性中target出现顺序依次执行每个target。在执行之前，首先需要执行它所依赖的target。程序中的名为run的target的depends属性compile，而名为compile的target的 depends属性是prepare，所以这几个target执行的顺序是prepare->compile->run。一个target只能被执行一次，即使有多个target依赖于它。如果没有if或unless属性，target总会被执行

3、property标签

property是用来定义属性的，或者是全局变量，其主要有name和value两个参数，其中name指的是属性的名字，value表示属性的值。

在需要使用属性的地方使用`${name}`即可调用该属性的值。

##Xcode命令行工具

1、查看当前系统支持的SDK的版本

`xcodebuild -showsdks`

2、查看Xcode的version和build

`xcodebuild -version`

3、显示工程项目的信息，需要先cd到*.xcodeproj的目录

`xcodebuild -list`

4、clean工程

`xcodebuild clean -sdk iphoneos7.0 -configuration Release`

5、编译工程

`xcodebuild -sdk iphoneos7.0 -configuration Release -target Demo`

6、打包ipa

`xcrun -sdk iphoneos PackageApplication -v ./build/Release-iphoneos/Demo.app -o Demo.ipa`

##范例

下面是本人参考资料编写的一个简单的ant build.xml文件示例：

```
<project name="Demo" default="debug" basedir=".">

    <property environment="env"/>
    
    <!-- 属性配置 -->
    <property name="app.name"               value="Demo"/>
    <property name="app.plist"              value="Demo/Demo-Info.plist"/>
    <property name="script.command.build"   value="xcodebuild"/> 
    
    <!-- 时间格式配置 -->
    <tstamp>
        <format property="time.now" pattern="yyyyMMddhhmmss" locale="zh"/>
    </tstamp>
    
    <!-- debug模式 -->
    <target name="debug" description="debug">
        <antcall target="compile">
            <param name="compile.configuration" value="Debug"/>
            <param name="compile.sdk" value="iphonesimulator"/>
        </antcall>
    </target>
    
    <!-- release模式 -->
    <target name="release" description="release">
        <antcall target="compile">
            <param name="compile.workspace" value="${app.workspace}"/>
            <param name="compile.scheme" value="${app.scheme}"/>
            <param name="compile.configuration" value="Release"/>
            <param name="compile.sdk" value="iphoneos"/>
        </antcall>
    </target>
    
    <!-- 发布模式 -->
    <target name="deploy" description="deploy">
        <echo message="开始发布应用程序:${app.name}"/>
        <antcall target="plist"/>
        <antcall target="release"/>
        <antcall target="ipa"/>
        <antcall target="zip"/>
        <echo message="发布应用程序${app.name}完成"/>
    </target>

    <!-- 在原来数值的基础上加1 -->
    <scriptdef name="increase.number" language="javascript">
        <attribute name="value" />
        <attribute name="property"/>
        <![CDATA[
            var initVal = attributes.get("value");
            var finalVal = parseInt(initVal) + 1;
            project.setProperty(attributes.get("property"), finalVal);
        ]]>
    </scriptdef>
    
    <!-- 压缩dSYM文件 -->
    <target name="zip" description="zip dsym">
        <exec executable="zip" failOnError="true">
            <arg line="-r build/${app.name}.app.dSYM.zip build/${app.name}.app.dSYM"/>
        </exec>
    </target>
    
    <!-- 编译源码 -->
    <target name="compile" description="">
        <exec executable="${script.command.build}" failOnError="true">
            <arg line="-verbose -configuration ${compile.configuration} -sdk ${compile.sdk}"/>
            <arg value="clean"/>
            <arg value="build"/>
            <arg value="CONFIGURATION_BUILD_DIR=${basedir}/build"/>
        </exec>
    </target>
    
    <!-- 生成IPA文件 -->
    <target name="ipa" description="">
        <!-- 获取版本号 -->
        <exec executable="/usr/libexec/PlistBuddy" outputproperty="CurrentVersion" errorproperty="PListError" failOnError="true">
            <arg value="-c"/>
            <arg value="Print :CFBundleShortVersionString"/>
            <arg value="${app.plist}"/>
        </exec>
        <!-- 获取Build号 -->
        <exec executable="/usr/libexec/PlistBuddy" outputproperty="CurrentBuild" errorproperty="PListError" failOnError="true">
            <arg value="-c"/>
            <arg value="Print :CFBundleVersion"/>
            <arg value="${app.plist}"/>
        </exec>
        <!-- 打包IPA -->
        <exec executable="xcrun" failOnError="true">
            <arg line="-sdk iphoneos PackageApplication"/>
            <arg line="-v &quot;${basedir}/build/${app.name}.app&quot;"/>
            <arg line="-o &quot;${basedir}/build/${app.name}_Release_${CurrentVersion}_${CurrentBuild}_${time.now}.ipa&quot;"/>
            <arg line="--embed ${provisioning.file}"/>
        </exec>
    </target>
    
    <!-- plist增加版本号 -->
    <target name="plist" description="">
        <exec executable="/usr/libexec/PlistBuddy" outputproperty="CurrentVersion" errorproperty="PListError" failOnError="true">
            <arg value="-c"/>
            <arg value="Print :CFBundleVersion"/>
            <arg value="${app.plist}"/>
        </exec>
        
        <echo message="Fetched the last version in the plist: ${CurrentVersion}" />
        <increase.number value="${CurrentVersion}" property="result"/>
        
        <exec executable="/usr/libexec/PlistBuddy" outputproperty="PListOutput" errorproperty="PListError" failOnError="true">
            <arg value="-c"/>
            <arg value="Set :CFBundleVersion ${result}" />
            <arg value="${app.plist}"/>
        </exec>
        
        <echo message="Output: ${PListOutput}"/>
        <echo message="Errors: ${PListError}"/>
        <echo message="Old version number: ${CurrentVersion} New Version Number: ${result}" />
    </target>

</project>

```
关于如何调用，直接在终端cd到build.xml文件所在的目录，记住将build.xml文件放置在.xcodeproj文件的同级目录下，输入如下命令即可一键打包编译：

Debug工程：`ant debug`

Release工程：`ant release`

发布打包项目：`ant deploy`

##参考资料

1、[《Apache Ant》](http://zh.wikipedia.org/zh-cn/Apache_Ant)

2、[《Ant之build.xml详解》](http://www.cnblogs.com/clarkchen/archive/2011/03/10/1980194.html)

3、[《Mac OS X，下载并安装ant》](http://blog.csdn.net/crazybigfish/article/details/18215439)

4、[《使用命令行实现iOS持续集成》](http://networking.ctocio.com.cn/481/12534981.shtml)

5、[参考项目实例工程Kimera](https://github.com/maxoly/Kimera)
