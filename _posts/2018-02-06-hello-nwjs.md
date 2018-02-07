---
layout: post
title: "使用NW.js开发桌面应用程序"
date: 2018-02-06 21:28:24 +0800
comments: true
tags: Note
---

前段时间要写一个桌面的应用，做一个简单的输入框供用户输入验证码的小功能，程序最开始是用Python写的，那么GUI一开始就考虑使用`wxPython`，虽然实现了功能但是总觉得太过于麻烦。

之前了解过关于Node.js开发桌面应用的技术，目前使用的比较多的都是[`nw.js`](https://nwjs.io) 和 [`electron`](https://electronjs.org)，由于下载electron的时候出现了一些问题所以就选择了`nw.js`来学习。

![nwjs_logo](/images/hello-nwjs/nwjs_logo.png)

NW.js（之前叫做node-webkit)能够通过DOM直接调用Node.js模块，实现通过Web技术来编写应用程序。

### 环境搭建

- 安装node.js 和 npm

### 测试、发布配置

#### （1）文件目录结构

```
demo
|____package.json //①
|____src
| |____app
| | |____main.js
| |____styles
| | |____main.css
| |____views
| | |____main.html
| |____assets
| | |____icon.png
| |____package.json //②
| |____README.md
| |____LICENSE
```

#### （2）创建发布配置文件

在源码上一级目录(上面对应的位置`①`)下面创建一个`package.json`文件，配置内容如下：

```
{
  "name": "demo",
  "version": "1.0.0",
  "devDependencies": {
    "nw": "^0.26.6",
    "nw-builder": "^3.5.1"
  },
  "scripts": {
    "dev": "nw src/",
    "prod": "nwbuild --platforms win32,win64,osx64 --buildDir dist/ src/"
  }
}
```

然后在命令行执行`npm install`安装相关依赖项。

#### （3）创建nw.js的配置文件

在源码所在src目录(上面对应的位置`②`)下面创建一个`package.json`文件，配置内容如下:

```
{
  "name": "demo",
  "main": "views/main.html",
  "version": "1.0.0",
  "description": "示例",
  "window": {
    "title": "示例",
    "width": 320,
    "height": 400,
    "max_width": 320,
    "max_height": 400,
    "min_width": 320,
    "min_height": 400,
    "as_desktop": true,
    "resizable": false,
    "show_in_taskbar": true,
    "icon": "icon.png"
  }
}
```

#### （4）示例页面

1) `main.html`：

```
<!DOCTYPE html>
<html>
  <head>
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    <span id="message"></span>
  </body>
  <script type="text/javascript" src="../app/main.js"></script>
</html>
```

2) `main.js`:

```
var os = require('os');
var message = document.getElementById("message");
message.innerHTML = "You are running on " + os.platform();
```

更多使用方式参考：`http://docs.nwjs.io`

#### （5）运行

```
#开发运行测试
npm run dev

#打包
npm run prod
```

运行效果如下：

![demo](/images/hello-nwjs/demo.jpg)

### 参考资料

1、[nw.js官网](https://nwjs.io)

2、[使用 NW.js 构建跨平台桌面应用程序](http://www.oschina.net/translate/cross-platform-desktop-app-nw-js)

3、[随笔分类 - nw.js](http://www.cnblogs.com/xuanhun/category/568577.html)