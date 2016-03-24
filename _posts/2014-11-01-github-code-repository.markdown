---
layout: post
title: "使用GitHub进行项目托管"
date: 2014-11-01 14:55:26 +0800
comments: true
tags: Note
---

Git 是由 Linux 之父 Linus Tovalds 为了更好地管理linux内核开发而创立的分布式版本控制／软件配置管理软件。目前支持 Windows 、MacOSX 、Linux 等多种主流平台，特点为快速、高效及易于使用。

###在GitHub上创建一个新的repository

1、在浏览器中输入`https://github.com/login`登录GitHub.

![github_login.png](/images/github_code_repository/github_login.png)

2、登录完成后，点击页面的右上角的加号，选择`New repository`.

![repo-create.png](/images/github_code_repository/repo-create.png)

3、为repository创建一个简短的、易记的名称。例如"CocoaPodsSample".

![create-repository-name.png](/images/github_code_repository/create-repository-name.png)

说明：

（1）`Repository name`:表示的是代码仓库的名称.

（2）`Description`:表示对仓库的描述，可选.

（3）`Public/Private`:表示该仓库是否对外公开.

（4）勾选`Initialize this repository with a README`表示默认创建一个README文件

填写上面的内容完成之后点击`Create repository`的按钮后仓库就创建了，创建的repository的SSHClone路径是：`git@github.com:hhtczengjing/CocoaPodsSample.git`。

###Git基本操作

1、clone仓库到本地

`git clone git@github.com:hhtczengjing/CocoaPodsSample.git`

2、提交代码

（1）拷贝源码到上面克隆出来的目录下面，拷贝前需要清除git信息

`find . -type d -name ".git"|xargs rm -rf`

（2）查看文件的git状态

cd到上面的克隆的目录下面，输入下面的命令可以查看git的状态：

`git status`

git的状态示例如下：

![git_status.png](/images/github_code_repository/git_status.png)

（3）按照git status状态提示的将指定的文件添加到git管理，命令如下：

`git add DevZeng/`

（4）将代码提交到本地仓库，-m后面的是提交的注释信息

`git commit -m "第一次提交"`

（5）提交本地仓库的代码到服务器主分支

`git push origin master`

（6）更新代码

`git pull`

###Git 常用命令

1、创建版本库

（1）克隆远程版本库

`git clone <url>`

（2）初始化本地版本库

`git init`

2、修改和提交

（1）查看状态

`git status`

（2）查看变更内容

`git diff`

（3）跟踪所有改动过的文件

`git add .`

（4）跟踪指定的文件

`git add <file>`

（5）文件改名

`git mv <old> <new>`

（6）删除文件

`git rm <file>`

（7）停止跟踪文件但不删除

`git rm --cached <file>`

（8）提交所有更新过的文件

`git commit -m “commit message”`

（9）修改最后一次提交

`git commit --amend`

3、查看提交历史

（1）查看提交历史

`git log`

（2）查看指定文件的提交历史

`git log -p <file>`

（3）以列表方式查看指定文件的提交历史

`git blame <file>`

4、撤消

（1）撤消工作目录中所有未提交文件的修改内容

`git reset --hard HEAD`

（2）撤消指定的未提交文件的修改内容

`git checkout HEAD <file>`

（3）撤消指定的提交

`git revert <commit>`

5、分支与标签

（1）显示所有本地分支

`git branch`

（2）切换到指定分支或标签

`git checkout <branch/tag>`

（3）创建新分支

`git branch <new-branch>`

（4）删除本地分支

`git branch -d <branch>`

（5）列出所有本地标签

`git tag`

（6）基于最新提交创建标签

`git tag <tagname>`

（7）删除标签

`git tag -d <tagname>`

6、合并与衍合

（1）合并指定分支到当前分支

`git merge <branch>`

（2）衍合指定分支到当前分支

`git rebase <branch>`

7、远程操作

（1）查看远程版本库信息

`git remote -v`

（2）查看指定远程版本库信息

`git remote show <remote>`

（3）添加远程版本库

`git remote add <remote> <url> `

（4）从远程库获取代码

`git fetch <remote>`

（5）下载代码及快速合并

`git pull <remote> <branch>`

（6）上传代码及快速合并

`git push <remote> <branch>`

（7）删除远程分支或标签

`git push <remote> :<branch/tag-name>`

（8）上传所有标签

`git push --tags`


###参考资料

1、[《Create A Repo》](https://help.github.com/articles/create-a-repo/)

2、[《Pro Git (中文版)》](https://progit.org/book/zh/)

3、[《Git Community Book (中文版)》](http://gitbook.liuhui998.com/index.html)

4、[Git常用命令速查表](/images/github_code_repository/Git_Cheat_Sheet.png)