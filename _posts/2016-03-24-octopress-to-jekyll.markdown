---
layout: post
title: "将博客从Octopress迁移到Jekyll"
date: 2016-01-23 18:49:35 +0800
comments: true
tags: Note
---

一直想把Octopress的主题给换了，没有找到满意的主题。最近发现[喵神的博客(OneV's Den)](https://onevcat.com)由ghost迁移到了[Jekyll](https://jekyllrb.com)了。

![](/images/octopress_jekyll/jekyll.png)

这些都不是重点，重点是他把之前使用的主题也一起弄到Jekyll上面来了，而且代码开源在GitHub上面了(项目地址：`https://github.com/onevcat/vno-jekyll`),效果如下：

![onevcat.png](/images/octopress_jekyll/onevcat.png)

###安装Vno Jekyll

####（1）从GitHub把项目的源码clone下来

```
git clone https://github.com/onevcat/vno-jekyll.git blog_site
```

####（2）安装相关依赖工具

``` 
sudo gem install bundler
bundle install
```

如果之前安装过bundler，可以先使用`sudo gem uninstall bundler`卸载然后再执行上面的命令安装。

####（3）启动服务

```
bundler exec jekyll serve
```

启动成功后如下所示：

![jekyll_server.png](/images/octopress_jekyll/jekyll_server.png)

###部署到GitHub和Coding

部署的过程可以参考我之前写的文章：

（1）[《Hello Octopress》](http://blog.devzeng.com/blog/hello-octopress.html)

（2）[《将博客从GitHub迁移到GitCafe》](http://blog.devzeng.com/blog/change-blog-host-to-gitcafe.html)

为了便于将文件和源码同步推送到github和coding上，我写了一个脚本：

```bash
#! /bin/bash

#提交源码到Github
rm -rf _site
git add .
git commit -m "update at `date` "

git remote add origin git@github.com:hhtczengjing/MyBlogSourceCode.git >> /dev/null 2>&1
echo "### Pushing Source to Github..."
git push origin master -f
echo "### Done"

#提交编译后的代码到Github
bundler exec jekyll clean
bundler exec jekyll build
cd _site

git init
git add .
git commit -m "update at `date` "

git remote add origin git@github.com:hhtczengjing/hhtczengjing.github.com.git >> /dev/null 2>&1
echo "### Pushing Source to Github..."
git push origin master -f
echo "### Done"

git remote add coding git@git.coding.net:zengjing2016/zengjing2016.git >> /dev/null 2>&1
echo "### Pushing to coding..."
git push coding master:coding-pages -f
echo "### Done"
```

###参考资料

1、[《Vno Jekyll源码》](https://github.com/onevcat/vno-jekyll)

2、[《Hello World - Vno》](http://vno.onevcat.com/2016/02/hello-world-vno/)

3、[《Jekyll Documentation》](http://jekyllrb.com)