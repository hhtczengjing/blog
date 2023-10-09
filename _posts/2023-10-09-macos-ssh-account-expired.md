---
layout: post
title: "macOS SSH 登录账号过期问题解决"
date: 2023-10-09 08:00:10 +0800
comments: true
tags: Note
---

近期对构建环境的几台 Mac Mini 做了一些“标准化”处理，处理完成后发现通过ssh访问输入密码无法正常连接了（按回车会立即提示连接结束）。通过查看控制台日志发现有如下的记录：

![error_log](/images/macos-ssh-account-expired/error_log.png)

错误信息：

```
error: PAM: user account has expired for xxxx from 127.0.0.1
```

各种查找资料没有找到能讲清楚这个问题原因的，经过一番测试发现一种解决办法：

(1) 查看是否打开共享的远程登录

> 旧版的系统(<= macOS 12)

![share_settings_01](/images/macos-ssh-account-expired/share_settings_01.png)

> 新版的系统(>= macOS 13)

![share_settings_03](/images/macos-ssh-account-expired/share_settings_03.png)

(2) 如果开启了远程登录且设置"允许访问"为"所有用户"还是不行，可以考虑设置为仅部分用户访问并将需要访问的用户添加进来

![share_settings_02](/images/macos-ssh-account-expired/share_settings_02.png)

完美解决问题。

> 附：

Jenkins的macOS节点如果无法通过SSH访问的情况可以通过 `Launch agent by connecting it to the controller` 的方式进行连接。在节点上面直接启动一段命令即可。在设置页面会提供启动的代码，代码示例如下：

```
curl -sO http://xxxx/jnlpJars/agent.jar
java -jar agent.jar -jnlpUrl http://xxxx/computer/yyyy/jenkins-agent.jnlp -secret zzzz -workDir "~/jenkins"
```