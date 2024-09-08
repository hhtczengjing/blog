---
layout: post
title: "Windows下制作Chrome多版本测试程序"
date: 2024-09-07 14:40:00 +0800
comments: true
tags: Note
---

前段时间基于CEF(Chromium Embedded Framework)做了一个Windows上的应用，主要能实现动态加载打包好的Web资源包，因为使用的是较低版本的Chrome内核，发现存在较多的兼容性问题，想到以前做IE兼容性测试的时候可以使用 IETester 来做IE不同版本之间的兼容性测试。

![ietester](/images/chrome-portable-app/ietester.png)

查了一些资料发现可以通过制作便携版的Chrome来实现多版本Chrome共存。因为经常要切版本，这里记录一下整个过程：

1、提取启动程序

（1）到[官网](https://portableapps.com/apps/internet/google_chrome_portable)下载 GoogleChromePortable_128.0.6613.120_online.paf.exe

![ChromiumPortable-1](/images/chrome-portable-app/ChromiumPortable-1.png)

（2）使用7zip打开，将 GoogleChromePortable.exe 文件提取出来

![ChromiumPortable-2](/images/chrome-portable-app/ChromiumPortable-2.png)

2、提取Chrome版本资源包

（1）下载Chrome离线包

如果要下载最新版本直接就到[官网](https://www.google.com/intl/en/chrome/browser/desktop/index.html?standalone=1)下载就可以。如果要下载比较旧的版本需要到去搜一下，这里列了几个可以去查找的地方，下载完成后需要仔细校验一下数字签名信息。

- 下载地址1：[https://www.chromedownloads.net/chrome64win-stable/](https://www.chromedownloads.net/chrome64win-stable/)

- 下载地址2：[https://commondatastorage.googleapis.com/chromium-browser-snapshots/index.html](https://commondatastorage.googleapis.com/chromium-browser-snapshots/index.html)

- 下载地址3：[https://vikyd.github.io/download-chromium-history-version/#/](https://vikyd.github.io/download-chromium-history-version/#/)

（2）提取程序

使用 7zip 打开下载回来的可执行程序

![chrome-1](/images/chrome-portable-app/chrome-1.png)

将 chrome.7z 里面的 Chrome-bin 解压出来

![chrome-2](/images/chrome-portable-app/chrome-2.png)

> 温馨提示：如果不是 chrome.7z 的话，说明不是离线安装包，需要重新下载

3、整合

（1）创建一个目录，这里我按照版本号的方式进行命名，比如 `Chrome_49.0.2623.75`，将 GoogleChromePortable.exe 拷贝进去，再创建一个 App 文件夹

![portable-1](/images/chrome-portable-app/portable-1.png)

（2）将 Chrome-bin 文件拷贝到 App 目录里面

![portable-2](/images/chrome-portable-app/portable-2.png)

双击 GoogleChromePortable.exe 即可启动，首次启动后会自动创建一个 Data 目录

![portable-3](/images/chrome-portable-app/portable-3.png)

> 可以将 GoogleChromePortable.exe 修改成任意名称，比如 chrome49.exe 然后创建一个桌面快捷方式。关闭浏览器窗口后 GoogleChromePortable.exe 进程不会自动结束，需要强制清理(参考exit.bat)。

上面的步骤做了一个GitHub Workflow 可以实现自动提取，完整代码见：https://github.com/hhtczengjing/portable_chrome.git `.github/workflows/builder.yml`，也可以使用 `build.bat`。

### 参考资料

- 1、[Chromium 历史版本离线安装包 - 下载方法](https://github.com/vikyd/note/blob/master/chrome_offline_download.md)

- 2、[自己制作Chrome便携版实现多版本共存](https://github.com/xiangyuecn/Docs/blob/master/Other/%E8%87%AA%E5%B7%B1%E5%88%B6%E4%BD%9Cchrome%E4%BE%BF%E6%90%BA%E7%89%88%E5%AE%9E%E7%8E%B0%E5%A4%9A%E7%89%88%E6%9C%AC%E5%85%B1%E5%AD%98.md)

- 3、[Google Chrome Portable](https://portableapps.com/apps/internet/google_chrome_portable)