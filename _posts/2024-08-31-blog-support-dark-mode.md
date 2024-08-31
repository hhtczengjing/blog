---
layout: post
title: "博客支持暗黑模式"
date: 2024-08-31 14:51:00 +0800
comments: true
tags: Note
---

去年支持了 github action 的方式实现代码push到master分支后能自动发布到 [github pages](https://docs.github.com/en/pages) 上面，确实比之前使用脚本强制提交发布的方式方便了不少，主要还是不需要本地配置环境了，切换到其他设备的只要写完提交就好了。

配置支持 github action 也是非常简单，从官方市场上面选择需要的action，简单配置一下就好了

![github-actions](/images/blog-support-dark-mode//github-actions.png)

最近想把博客适配支持一下暗黑模式，需要在本地调试一下，按照之前的文档安装了一下环境，结果是各种报错，折腾了个把小时，最后还是没搞定。最终采用的是使用 Docker 搭建环境映射本地markdow的路径在容器内进行编译的方式搞定，使用方式比较简单：

将 [jekyll-docker](https://github.com/hhtczengjing/jekyll-docker) 仓库的3个文件拷贝到 blog 源码根目录下面，然后按照下面的命令操作即可：

```
# 启动服务
docker-compose up --build -d
# 停止服务
docker-compose down
```

![docker-jekyll-1](/images/blog-support-dark-mode/docker-jekyll-1.png)

*** 注意 ***：

1、容器创建完成后，第一次访问 http://localhost:4000 可能会失败，因为容器还需要等jekyll服务初始化完成。

![docker-jekyll-2](/images/blog-support-dark-mode/docker-jekyll-2.png)

2、 [jekyll-docker](https://github.com/hhtczengjing/jekyll-docker) 主要实现参考 [BretFisher/jekyll-serve](https://github.com/BretFisher/jekyll-serve) 这个项目，对初始化环境做了一些调整

支持暗黑模式原理比较简单，利用 `prefers-color-scheme` 这个媒体查询，实现深色和浅色模式下的颜色调整，示例代码如下：

```
// 深色模式
@media (prefers-color-scheme: dark) {
  body {
    background: black;
    color: white;
  }
}

// 浅色模式
@media (prefers-color-scheme: light) {
  body {
    background: white;
    color: black;
  }
}
```

博客使用的是 [vno-jekyll](https://github.com/onevcat/vno-jekyll) 这个主题，样式代码在 `_sass` 这个目录下面，使用的是 scss 实现。

首先创建两个 mixin (`dark-scheme/light-scheme`)，用于在里面定义不同模式下的颜色变量：

```
// file vno-light.scss
@mixin light-scheme {
    --bg-default-color:     #FFFFFF;

    --text-primary-color:   #666666;
    --text-regular-color:   #999999;
    --text-secondary-color: #c7c7c7;

    // ... 省略代码 ...
}

// file vno-dark.scss
@mixin dark-scheme {
    --bg-default-color:     #141414;

    --text-primary-color:   #F2F2F2;
    --text-regular-color:   #999999;
    --text-secondary-color: #999999;

    // ... 省略代码 ...
}
```

然后利用 mixin 的特性和 `prefers-color-scheme` 在不同模式下 include 不同的 mixin：

```
html,
body {
  height: 100%;

  @media (prefers-color-scheme: light) {
    @include light-scheme;
  }
  
  @media (prefers-color-scheme: dark) {
    @include dark-scheme;
  }
}
```

在需要适配的地方使用变量(`var(变量名)`)替换:

```
html {
  height: 100%;
  max-height: 100%;
  background: var(--bg-default-color);
}

body {
  font-family: $sans-font;
  font-size: 1em;
  color: var(--text-primary-color);
  -webkit-font-smoothing: antialiased;
}
```

最后效果如下：

![blog-preview](/images/blog-support-dark-mode/blog-preview.png)

算是勉强能用把，后续还要对细节进行调整。

### 参考资料

- [1、主题适配夜间模式](https://blog.tmaize.net/posts/2021/01/11/%E4%B8%BB%E9%A2%98%E9%80%82%E9%85%8D%E5%A4%9C%E9%97%B4%E6%A8%A1%E5%BC%8F.html)

- [2、jekyll-serve](https://github.com/BretFisher/jekyll-serve)