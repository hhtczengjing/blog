---
layout: post
title: "如何将自己的Node.js包发布到npm上面"
date: 2019-05-31 18:46:37 +0800
comments: true
tags: Note
---

早前写过一篇[《使用Verdaccio搭建npm仓库》](https://blog.devzeng.com/blog/npm-repo-with-verdaccio.html)介绍如何搭建私有的npm包托管的环境的文章，比较适合将私有的Node.js包发布上去。本文主要记录一下发布一个公开的package到npm的过程。

![npm-intro.png](/images/npm-publish-package/npm-intro.png)

1、注册账号

前往`https://www.npmjs.com`注册账号，并按照要求验证邮箱。

2、Node.js包

创建`package.json`文件，如下：

```
{
  "name": "gitlab-systemhook-handler",
  "version": "0.1.0",
  "description": "Web handler / middleware for processing Gitlab System hooks",
  "main": "gitlab-systemhook-handler.js",
  "scripts": {
    "test": "node demo.js"
  },
  "keywords": [
    "gitlab",
    "systemhook"
  ],
  "author": "zengjing <hhtczengjing@gmail.com>",
  "repository": {
    "type": "git",
    "url": "https://github.com/hhtczengjing/gitlab-systemhook-handler.git"
  },
  "license": "MIT",
  "dependencies": {
    "bl": "~1.1.2",
    "buffer-equal-constant-time": "~1.0.1"
  }
}
```

说明：

- （1）name: 包的名字
- （2）version：版本号
- （3）main：入口的JS文件名称
- （4）repository：源码路径
- （5）dependencies：依赖库

3、登录

命令行输入：`npm login`，如果设置了第三方的registry，可以在后面加上`--registry https://registry.npmjs.com/`，然后按照要求输入用户名、密码和邮箱即可完成登录。

```
➜ npm login --registry https://registry.npmjs.com/
Username: zengjing
Password: 
Email: (this IS public) hhtczengjing@gmail.com
Logged in as zengjing on https://registry.npmjs.com/.
```

4、发布

命令行输入：`npm publish`，同上如果设置了第三方的registry，可以在后面加上`--registry https://registry.npmjs.com/`。

```
➜ npm publish --registry https://registry.npmjs.com/
npm notice 
npm notice 📦  gitlab-systemhook-handler@0.1.0
npm notice === Tarball Contents === 
npm notice 582B  package.json                
npm notice 762B  demo.js                     
npm notice 2.5kB gitlab-systemhook-handler.js
npm notice 1.3kB README.md                   
npm notice === Tarball Details === 
npm notice name:          gitlab-systemhook-handler               
npm notice version:       0.1.0                                   
npm notice package size:  1.9 kB                                  
npm notice unpacked size: 5.1 kB                                  
npm notice shasum:        115e54761497edeb3187617f6b683f2300f877b4
npm notice integrity:     sha512-zNm/3Tzps0HUJ[...]ZoHOcyTeckDuA==
npm notice total files:   4                                       
npm notice 
+ gitlab-systemhook-handler@0.1.0
```

然后到npm的后台可以看到发布成功的package了：

![packages.png](/images/npm-publish-package/packages.png)

如果遇到如下的错误：

![403_error.png](/images/npm-publish-package/403_error.png)

表示没有验证邮箱，请先到注册的邮箱验证，然后重新发布即可

### 参考资料

1、[如何使用npm发布自己的npm包](https://blog.csdn.net/zyg1515330502/article/details/81112653)

2、[npm 官网](https://www.npmjs.com)