---
layout: post
title: "Jenkins多节点同步CocoaPods索引"
date: 2023-09-20 23:08:37 +0800
comments: true
tags: Note
---

CocoaPods的索引库更新一直以来是一件很痛苦的事情，为了提升效率在项目中引入了镜像索引库的方案，将项目用到的第三方库的podspec配置自动抽取到一个镜像索引库里面。这个方案确实降低了大家在同步索引库的耗时，但是在构建环境下存在较多的问题。

现在是要去多台 Macos 节点 需要定期同步 CocoaPods 的索引。同步索引库直接通过命令 ` pod repo update xxx ` 就能解决，如果要多个节点同步那么使用 Pipeline 的任务每个节点执行脚本就可以了，具体的代码如下：

```
node("built-in") {
    stage('Master Update') {
        sh label: '', script: '''
            export LANG="en_US.UTF-8"
            pod repo update mirror-spec
        '''
    }
}

node("slave1") {
    stage('Slave1 Update') {
        sh label: '', script: '''
            export LANG="en_US.UTF-8"
            pod repo update mirror-spec
        '''
    }
}
```

但是运行一段时间发现由于git账号的密码会定期修改，就会出现执行命令的过程中需要输入密码导致任务结束的问题。

发现 Jenkins 在 clone git 仓库的时候会自动注入配置的凭据，从日志发现用到了一个环境变量 [`GIT_ASKPASS`](https://git-scm.com/docs/gitcredentials) ， 摘录一段官方的介绍：

```
If the GIT_ASKPASS environment variable is set, the program specified by the variable is invoked. A suitable prompt is provided to the program on the command line, and the user’s input is read from its standard output.
```

大概的意思是如果发现环境变量 `GIT_ASKPASS` 就会从这个里面获取凭据信息，环境变量配置的是一个可执行的脚本，脚本的模板代码如下：

```
#!/bin/sh
case "$1" in
    Username*) echo $GIT_USERNAME ;;
    Password*) echo $GIT_PASSWORD ;;
esac
```

从代码直接可以看到只要在 echo 后面放对应的用户名密码就行了。这里需要说明的是脚本文件需要赋予可执行权限。

那么接下来的问题就来了，怎么获取配置的凭据数据？发现了一个插件 [Credentials Binding Plugin](https://www.jenkins.io/doc/pipeline/steps/credentials-binding/) , 官方提供了示例代码

```
node {
  withCredentials([usernamePassword(credentialsId: 'mylogin', usernameVariable: 'VAR_USERNAME', passwordVariable: 'VAR_PASSWORD')]) {
    sh '''
      echo $VAR_USERNAME
      echo $VAR_PASSWORD
    '''
  }
}
```

大概的做法是将凭据ID `mylogin` 配置的用户名密码读取出来分别放到环境变量 `VAR_USERNAME` 和 `VAR_PASSWORD` 里面，然后就可以自由发挥了。最终整合后的代码如下：

```
node("built-in") {
    stage('Master Update') {
        withCredentials([usernamePassword(credentialsId: 'credentialsId', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
            sh label: '', script: '''
                export LANG="en_US.UTF-8"
                rootPath="${WORKSPACE}"
                cd ${rootPath}
                # set git credentials
                ASKPASS=${rootPath}/askpass.sh
                cat > $ASKPASS <<'EOF'
#!/bin/sh
case "$1" in
    Username*) echo $GIT_USERNAME ;;
    Password*) echo $GIT_PASSWORD ;;
esac
EOF
                chmod +x $ASKPASS
                export GIT_ASKPASS=$ASKPASS
                # update repo
                pod repo update mirror-spec
                # clean resources
                rm -rf $ASKPASS
            '''
        }
    }
}

node("slave1") {
    stage('Slave1 Update') {
        withCredentials([usernamePassword(credentialsId: 'credentialsId', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
            sh label: '', script: '''
                export LANG="en_US.UTF-8"
                rootPath="${WORKSPACE}"
                cd ${rootPath}
                # set git credentials
                ASKPASS=${rootPath}/askpass.sh
                cat > $ASKPASS <<'EOF'
#!/bin/sh
case "$1" in
    Username*) echo $GIT_USERNAME ;;
    Password*) echo $GIT_PASSWORD ;;
esac
EOF
                chmod +x $ASKPASS
                export GIT_ASKPASS=$ASKPASS
                # update repo 
                pod repo update mirror-spec
                # clean resources
                rm -rf $ASKPASS
            '''
        }
    }
}
```

部署了一段时间了，暂时一切运转正常。

### 参考资料

- [Credentials Binding Plugin](https://www.jenkins.io/doc/pipeline/steps/credentials-binding/)

- [gitcredentials](https://git-scm.com/docs/gitcredentials)
