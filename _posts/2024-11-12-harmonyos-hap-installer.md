---
layout: post
title: "HarmonyOS NEXT HAP安装工具"
date: 2024-11-12 23:00:00 +0800
comments: true
tags: Note
---

HarmonyOS NEXT 上的产物有 APP/HAP 两种，其中 APP 上架使用，HAP 用于非上架的场景（企业内部分发和调试包）。

- APP 仅用于提交上架，不可以直接安装，只能通过AppStore下载

- HAP 非上架发布的场景，可以本地安装，如果是企业发布的应用release模式下的hap无法直接安装

![agc](/images/harmonyos-hap-installer/agc.png)

在实际项目中日常提测不可能每次都让开发直接给测试的机器build安装一下，还是需要提供安装包来使用。

可以通过 [Command Line Tools for HarmonyOS](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-commandline-get-V5) + jenkins 做自动化任务，生成hap。

```
hvigorw assembleHap --mode module -p product=default -p buildMode=debug --no-daemon
```

生成的产物路径：${PROJECT_PATH}/{moduleName}/build/{productName}/outputs/{targetName}/xxx.hap 将 HAP 拷出来用 DevEco Testing 安装就好了。

![deveco-testing.png](/images/harmonyos-hap-installer/deveco-testing.png)

> 说明：DevEco Testing 需要使用开发者账号登录才能使用

使用命令行：

```
hdc file send "entry-default-signed.hap" "data/local/tmp/entry-default-signed.hap"
hdc shell bm install -p "data/local/tmp/entry-default-signed.hap"
```

如果项目中包含HSP动态库，需要每个模块构建生成HSP

```
hvigorw assembleHsp --mode module -p module=library@default -p product=default --no-daemon
```

生成的产物路径：${PROJECT_PATH}/{moduleName}/build/{productName}/outputs/{targetName}/xxx.hsp

将生成的hap/hsp拷贝出来，压缩成zip使用 DevEco Testing 安装。当然可以使用命令：

```
remote_path="data/local/tmp/8be7b3fc662b4b8a9d91f52d39632989"
hdc shell mkdir "${remote_path}"

for file in $dir_path/*
do
  if [ -f "$file" ]; then
    echo "hdc file send $file ${remote_path}"
    hdc file send $file "${remote_path}"
  fi
done

hdc shell bm install -p "${remote_path}"

hdc shell rm -rf "${remote_path}"
```

Command Line Tools 大约2G左右，下载起来也是比较费劲，其实只用到了hdc这个命令，想着能不能只把hdc提取出来，没想到竟然可以（研究了一下DevEco Testing其实也是用的简化的hdc命令）。项目源码地址：https://github.com/hhtczengjing/hdc_cli.git

当然命令行还是有点麻烦，初步实现了一个GUI可视化的工具，支持拖动zip包直接安装到手机上面，项目地址： https://github.com/hhtczengjing/hap_installer.git

### 参考资料

- 1、[HAP/HAR/HSP的关系是什么？](https://developer.huawei.com/consumer/cn/doc/harmonyos-faqs/faqs-package-structure-19-V5)

- 2、[搭建流水线](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-command-line-building-app-V5#section1518833762214)

- 3、[下载中心](https://developer.huawei.com/consumer/cn/download/)