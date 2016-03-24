---
layout: post
title: "GitLab的简单使用"
date: 2015-07-12 04:05:48 +0800
comments: true
tags: Note
---

[GitLab](https://about.gitlab.com)是一个利用`Ruby on Rails`开发的开源应用程序，实现一个自托管的Git项目仓库，可通过Web界面进行访问公开的或者私人项目。

![logo.png](/images/gitlab_usage/logo.png)

它拥有与Github类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。团队成员可以利用内置的简单聊天程序(Wall)进行交流。它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。开源中国代码托管平台`git.oschina.net`就是基于GitLab项目搭建。

###添加新项目

1、选择创建新项目

登录成功后，点击导航条上的 “+” 就可以进入创建项目的页面

![new_project_1.png](/images/gitlab_usage/new_project_1.png)

2、填写项目的信息

在创建工程的页面，按照要求填写项目的名称和可见性等信息。

![new_project_2.png](/images/gitlab_usage/new_project_2.png)

（1）Project path：项目的路径，一般可以认为是项目的名称

（2）Import prject from：从哪导入项目，提供Github/Bitbucket等几个选项

（3）Description（项目的描述）：可选项，对项目的简单描述

（4）Visibility Level（项目可见级别）：提供Private（私有的，只有你自己或者组内的成员能访问）/Internal（所有登录的用户）/Public(公开的，所有人都可以访问)三种选项。

###添加和配置SSH公钥

SSH（Secure Shell）是一种安全协议，在你的电脑与GitLab服务器进行通信时，我们使用SSH密钥（SSH Keys）认证的方式来保证通信安全。你可以在网络上搜索到关于SSH密钥的更多介绍；下面我们重点讲解如何创建 SSH密钥，并将密钥中的公钥添加到GitLab，以便我们通过SSH协议来访问Git仓库。

![add_sshkey_3.gif](/images/gitlab_usage/add_sshkey_3.gif)

SSH 密钥的创建需要在终端（命令行）环境下进行，我们首先进入命令行环境。通常在OS X和Linux平台下我们使用终端工具（Terminal），在Windows平台中，可以使用Git Bash工具。

进入命令行环境后，我们执行以下操作来创建 SSH 密钥。

####1.进入SSH目录

`cd  ~/.ssh`

（1）如果还没有 ~/.ssh 目录，可以手工创建一个(`mkdir ~/.ssh`)，之后再通过`cd ~/.ssh`进入SSH目录

（2）可以通过`ls -l`命令查看SSH目录下的文件，来确认你是否已经生成过SSH密钥；如果SSH目录为空，我们开始第二步，生成 SSH 密钥；如果存在id_rsa.pub这个文件，说明你之前生成过SSH密钥，后面有介绍如何添加多个sshkey

####2.生成SSH密钥

我们通过下面的命令生成密钥，请将命令中的`YOUR_EMAIL@YOUREMAIL.COM`替换为你自己的`Email`地址。

`ssh-keygen -t rsa -C "YOUR_EMAIL@YOUREMAIL.COM"`

在SSH生成过程中会出现以下信息，按屏幕的提示操作即可；

```
$ ssh-keygen -t rsa -C "YOUR_EMAIL@YOUREMAIL.COM"
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/USERNAME/.ssh/id_rsa.
Your public key has been saved in /Users/USERNAME/.ssh/id_rsa.pub.
The key fingerprint is:
15:81:d2:7a:c6:6c:0f:ec:b0:b6:d4:18:b8:d1:41:48 YOUR_EMAIL@YOUREMAIL.COM
```

说明：

（1）一般情况下，在命令行中输入密码、口令一类的信息时是没有信息回显的。在我们这一步的操作中，输入passphrase口令时，命令行界面上不会随着键盘敲入密码而有什么反馈。

（2）当提示`Enter passphrase (empty for no passphrase) : `时，可以直接按两次回车键输入一个空的 passphrase；也可以选择输入一个 passphrase 口令，如果此时你输入了一个`passphrase`，请牢记，之后每次提交时都需要输入这个口令来确认。

####3.获取SSH公钥信息

SSH密钥生成结束后，你可以在SSH目录下看到私钥`id_rsa`和公钥`id_rsa.pub`这两个文件，不要把私钥文件`id_rsa`的信息透露给任何人。我们可以通过文本编辑器或`cat`命令来查看`id_rsa.pub`公钥信息。

（1）通过编辑器。使用你熟悉的文本编辑器，比如 记事本、Sublime Text等软件打开`id_rsa.pub`，复制里面的所有内容以备下一步使用。

（2）通过cat命令。在命令行中敲入`cat id_rsa.pub`，回车执行后命令行界面中会显示`id_rsa.pub`文件里的内容，复制后在下一步使用。

（3）通过直接使用命令将`id_rsa.pub`文件里的内容复制到剪切板中

- Windows: `clip < ~/.ssh/id_rsa.pub`

- Mac: `pbcopy < ~/.ssh/id_rsa.pub`

- GNU/Linux (requires xclip): `xclip -sel clip < ~/.ssh/id_rsa.pub`

####4.添加SSH公钥到gitlab

（1）打开`https://gitlab.com/profile`Profile配置页面，选择SSH Keys.

![add_sshkey_1.png](/images/gitlab_usage/add_sshkey_1.png)

（2）添加SSH公钥

按照要求填写Title和Key，其中Title是Key的描述信息（如My_work_computer等），Key是上面复制的SSH公钥的内容，直接粘贴到输入框中保存即可。

![add_sshkey_2.png](/images/gitlab_usage/add_sshkey_2.png)

####5.测试SSH连接

`ssh -T git@gitlab.com`

如果连接成功的话，会出现以下信息。

`Welcome to GitLab, USERNAME!`

###如何同时使用多个SSH公钥

如果你已经有了一套ssh(笔者的电脑上就有好几套如github/gitcafe/gitlab,三者各不一样)，为了保证各个服务能正常使用需要配置多个SSH Key。可以按照以下的步骤来实现多套SSH Key的共同工作：

####1.生成SSH密钥

假设你已经有了一套名为id_rsa的公秘钥，将要生成的公秘钥名称为gitlab，你也可以使用任何你喜欢的名字。记得把以下命令中的`YOUR_EMAIL@YOUREMAIL.COM`改为你的`Email`地址

`ssh-keygen -t rsa -C "YOUR_EMAIL@YOUREMAIL.COM" -f ~/.ssh/gitlab`

说明：

（1）`-f`后面的参数是自定义的SSH Key的存放路径，将来生成的公秘钥的名字分别是gitlab.pub和gitlab

（2）其他的和上面生成密钥的步骤相同，只是多了下面的配置的步骤

####2.配置自定义的公秘钥名称

在SSH用户配置文件~/.ssh/config中指定对应服务所使用的公秘钥名称，如果没有config文件的话就新建一个(`vim ~/.ssh/config`)，并输入以下内容(可以添加多个)：

```
Host gitlab.com www.gitlab.com
  IdentityFile ~/.ssh/gitlab
```

###导入项目或提交代码

1、初始上传代码

（1）可以先将项目clone到本地然后将文件拷贝到目录下面再提交上去

```
git clone git@gitlab.com:USERNAME/PROJECTNAME.git
cd WatchDemo
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```

（2）如果项目存在需要导入到gitlab可以直接将项目导入上去

```
cd existing_folder
git init
git remote add origin git@gitlab.com:USERNAME/PROJECTNAME.git
git push -u origin master
```

说明：

（1）请将上面的USERNAME和PROJECTNAME替换成用户名和项目的名称

（2）existing_folder指的是项目在本地的路径（根路径）

2、其他使用请参考[《使用GitHub进行项目托管》](http://blog.devzeng.com/blog/github-code-repository.html)，或者是到网络上查看Git的相关资料

> 感谢[GitCafe](https://gitcafe.com)，一个非常优秀的代码托管平台，本文很多资料来源于GitCafe官方提供的[帮助手册](https://help.gitcafe.com/manuals/help)。

###参考资料

1、[《GitLab Help》](https://gitlab.com/help/)

2、[《Gitcafe Help》](https://help.gitcafe.com/manuals/help/ssh-key)

3、[《Gitlab的使用》](http://blog.cnbluebox.com/blog/2014/04/15/gitlabde-shi-yong/)

4、[《用GitLab搭建自己的私有GitHub》](http://segmentfault.com/a/1190000000345686)
