---
layout: post
title: "macOS iTerm2 环境配置"
date: 2023-09-29 15:07:20 +0800
comments: true
tags: Note
---

近今年开始在macOS上面不再使用自带的终端(Terminal.app)，开始使用 iTerm2 替代。整个安装配置比较简单，记录一下个性化的配置的步骤（主要是配置主题和字体），免得再次配置的时候又需要到处去找。

![preview](/images/macos_iterm2/preview.png)

### 下载安装

到官网下载最新的安装包，下载地址：[https://iterm2.com/downloads.html](https://iterm2.com/downloads.html)

![download](/images/macos_iterm2/download.png)

点链接下载完成后直接拖 `iTerm.app` 文件到 `/Applications` 即可。

### 主题配置

主题使用的是 `Snazzy`，直接到 [https://github.com/sindresorhus/iterm2-snazzy](https://github.com/sindresorhus/iterm2-snazzy) 下载 `Snazzy.itermcolors` 文件即可。

下载完成后 `iTerm2 -> Settings -> Profiles -> Colors` 右下角的 `Color Presets` 选择 `Import...` 导入 `Snazzy.itermcolors` 

![colors-setting](/images/macos_iterm2/colors-setting.png)

导入成功后选中主题就可以了

![theme-selected](/images/macos_iterm2/theme-selected.png)

### 安装 Oh My Zsh

[Oh My Zsh](https://ohmyz.sh/) 是一个开源的、社区驱动的命令行工具。Oh My Zsh 是基于 zsh 命令行的一个扩展工具集，提供了丰富的扩展功能。

![omz-update](/images/macos_iterm2/omz-update.png)

先确保已经安装zsh，打开终端，在终端上输入如下命令: 

```
➜  ~ zsh --version

zsh 5.9 (x86_64-apple-darwin22.0)
```

> 安装

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

目前就只用了这些，后续遇到了就再补充吧....