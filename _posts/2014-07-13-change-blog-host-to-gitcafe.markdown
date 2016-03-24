---
layout: post
title: "将博客从GitHub迁移到GitCafe"
date: 2014-07-13 14:51:57 +0800
comments: true
tags: Note
---

最近一直使用[Github Pages功能](https://pages.github.com/)以及[Octopress](http://octopress.org/)来记录自己在学习和生活中的琐事，之前也写了一篇文章[《Hello Octopress》](/blog/hello-octopress.html)来分享如何在GitHub使用Octopress来搭建博客的技术细节。

但是最近发现一个很头疼的问题就是博客的访问速度实在是很慢，虽然做了一部分的优化，比如将GOOGLE的字体和jQuery的链接移除，但整体来看速度还是没有明显的提升，最近看到网上有资料提到可以将博客转移到GitCafe的方法，周末闲来无事就将博客内容镜像到[GitCafe](http://gitcafe.com/signup?invited_by=zengjing)上。

以下为大家介绍详细的迁移过程。

## 迁移教程

### 注册

如果你还没有注册过[GitCafe](http://gitcafe.com/signup?invited_by=zengjing)，首先需要[点这里](https://gitcafe.com/signup)注册一下。

![注册用户](/images/gitcafe/gitcafe-signup.png)

### 在GitCafe上新建一个博客项目

然后我们需要先在GitCafe上新建一个博客项目。GitCafe的博客搭建官方教程藏得比较深，所以我第一次还没有找到，教程地址在[这里](https://gitcafe.com/GitCafe/Help/wiki/Pages-%E7%9B%B8%E5%85%B3%E5%B8%AE%E5%8A%A9#wiki)。具体来说，就是创建一个与用户名(如果是组织，就是组织名)相同名称的项目。如果你创建的项目名与用户名相同，GitCafe会自动识别成这是一个Page项目，如下所示：

![创建新项目](/images/gitcafe/gitcafe-create-page.png)

### 设置多个Git Remote源

接下来我们需要将原本提交到Github上的博客内容同步提交到GitCafe。因为我的博客是基于[Octopress](http://octopress.org/)的，我介绍一下Octopress的做法，其它博客引擎的做法类似。
为简化操作，便于部署，可以直接修改`Rakefile`文件。在其大约第269行左右增加如下代码，也可以达到同样的目的，这样你每次就仍然只需要执行`rake deploy`即可同时将博客同步到github和gitcafe【替换YOUR_GITCAFE_SOURCE为你对应的GitCafe源地址】：

``` ruby
system "git remote add gitcafe YOUR_GITCAFE_SOURCE >> /dev/null 2>&1"
system "git push -u gitcafe master:gitcafe-pages"
```

插入代码的示例位置如下：

![设置Rakefile](/images/gitcafe/gitcafe-edit-rakefile.png)


### 设置域名

GitCafe的自定义域名设置比github要友好得多，它不但提供了图形界面设置，并且支持同时设置多个域名。在`项目管理`->`域名管理`中，我们可以找到相应的设置项，如下所示：

![自定义域名](/images/gitcafe/custom_domains.png)

在设置完之后，我们需要去域名解析的服务商那儿，将对应的域名用`A记录`类型，解析到`117.79.146.98`即可。

![域名解析](/images/gitcafe/domain_dns.png)


##参考资料

1、[《将博客从GitHub迁移到GitCafe》](http://devtang.com/blog/2014/06/02/use-gitcafe-to-host-blog/)

2、[《Pages相关帮助》](https://gitcafe.com/GitCafe/Help/wiki/Pages-相关帮助#wiki)


