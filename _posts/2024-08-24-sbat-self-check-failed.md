---
layout: post
title: "ThinkPad无法启动系统问题解决办法"
date: 2024-08-24 17:20:00 +0800
comments: true
tags: Note
---

最近几天一台Windows电脑(ThinkPad)无法启动进入系统，开机有个错误提示界面，然后就自动关机，错误界面如下图：

![error](/images/sbat-self-check-failed/error.jpg)

上面的文字内容如下：

```
Verifying shlm SBAT data failed: Security Policy Violation
Something has gone seriously wrong: SBAT self-check failed: Security Policy Violation 
```

开始怀疑是系统硬件问题，使用系统自带的自检工具查了一下没有发现有啥异常问题。简单用错误提示在网上查了一下都是一些和Ubuntu相关的错误，就没怎么仔细去看了。

觉得可能要重做系统，手头上也没有安装系统的工具拿到电脑维修店去检查了一下，说是主板有问题要维修（电脑比较旧了感觉维修没啥必要就没管了，还好没有修要不然就被忽悠了，水真的很深）。

查了一下资料，发现网上有很多类似的问题，简单按照步骤对bios设置了一下，竟然搞定了，这里记录一下解决的过程。

开机按 `F1` (和电脑有关，具体要看每个机型的设置) 进入 bios 设置界面。

![bios-home](/images/sbat-self-check-failed/bios-home.png)

1、关闭 `Restart -> Load Setup Defaults -> OS Optimized Defaults`

![restart-settings-1](/images/sbat-self-check-failed/restart-settings-1.png)

弹框选择`Yes`保存设置

![restart-settings-2](/images/sbat-self-check-failed/restart-settings-2.png)

2、关闭 `Security -> Secure Boot -> Secure Boot`

![security-settings-1](/images/sbat-self-check-failed/security-settings-1.png)

![security-settings-2](/images/sbat-self-check-failed/security-settings-2.png)

按 `F10` 保存设置

3、再次开机就可以正常进入系统了。

### 参考资料

- [1、调整BIOS Setup中的OS Optimized Defaults设置项](https://robotrs.lenovo.com.cn/ZmptY2NtYW5hZ2Vy/p4data/Rdata/Rfiles/701.html)

- [2、Disable and Enable Secure Boot in BIOS - Lenovo Support Quick Tips](https://support.lenovo.com/nz/en/solutions/nvid500424-disable-and-enable-secure-boot-in-bios-lenovo-support-quick-tips)