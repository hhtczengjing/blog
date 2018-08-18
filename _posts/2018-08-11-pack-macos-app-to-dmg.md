---
layout: post
title: "如何将macOS应用程序打包为dmg文件"
date: 2018-08-11 11:46:29 +0800
comments: true
tags: macOS
---

之前改写过网上开源的一个JSON转Model的Mac APP，当时是直接使用的是将`.app`格式的文件直接拖到应用里面进行安装的，最近刚好有空就了解了一下dmg文件是怎么打包的，记录下整个的过程方便以后查找。

### 操作步骤

#### 1.准备相关文件

- (1) 打包生成的`.app`文件
- (2) 一张背景图
- (3) `Applications`文件夹的替身文件(可以到其他的dmg里面去拷贝一个)

#### 2.创建空白镜像文件

(1) 打开`磁盘工具`，选择`文件` -> `新建映像` -> `空白映像`：

![1.png](/images/macos-build-dmg-file/1.png)

(2) 在弹出框中填写相关的信息

![2.png](/images/macos-build-dmg-file/2.png)

(3) 填写完成后点击保存，即可生成一个空白的dmg文件

#### 3.配置

(1) 拷贝文件

双击前面创建的DMG文件，在Finder中打开，直接将之前准备好的相关文件拖进去就行了

![3.png](/images/macos-build-dmg-file/3.png)

(2) 设置背景图片和图标大小

在打开的镜像文件中(Finder)的空白地方右键选择`查看显示选项`

1) 设置图标大小为`100*100`（具体可以根据实际需要进行调整)

2）设置背景为`图片`，将背景图片拖到右边的框里面

![8.png](/images/macos-build-dmg-file/8.png)

(3) 隐藏背景图片

隐藏背景图片文件夹的方式就是将其重命名为`.`开头的

```
mv /Volumes/YoudaoNote_3.3.2/background /Volumes/YoudaoNote_3.3.2/.background
```

或者是

```
chflags hidden /Volumes/YoudaoNote_3.3.2/background
```

(4) 排列图标

直接拖动图标到指定位置，拖完的效果如下

![4.png](/images/macos-build-dmg-file/4.png)

(5) 关闭镜像

打开`磁盘工具`将左侧的对应的`磁盘映像`关闭即可

![5.png](/images/macos-build-dmg-file/5.png)

#### 4.转换

(1) 打开`磁盘工具`，选择`映像` -> `转换`：

![6.png](/images/macos-build-dmg-file/6.png)

(2) 填写要保存的文件的名称点击转换即可，生成的文件就是最终的dmg文件

![7.png](/images/macos-build-dmg-file/7.png)

### 参考资料

1、[Mac OS 开发 － 聊聊如何打包dmg文件](https://www.jianshu.com/p/c6cd257676bf)

2、[制作映像(dmg)文件详细步骤](https://bbs.feng.com/read-htm-tid-6724285.html)