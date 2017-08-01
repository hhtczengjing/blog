---
layout: post
title: "使用Dockerfile构建Docker镜像"
date: 2017-08-01 08:08
comments: true
tags: Note
---

![docker-logo.png](/images/install-docker-in-centos/docker-logo-compressed.png)

Docker中有个非常重要的概念叫做——镜像（Image）。Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

### Dockerfile语法说明

##### 1、FROM: 指定基础镜像。

定制镜像的时候都是以一个镜像为基础，在这个基础上面进行定制。`FROM`在`Dockerfile`中是必须的指令，而且必须是第一条指令。

- 1）在Docker Hub上有非常多的官方镜像，比如服务类(nginx/redis)、语言类(node/openjdk/python)、操作系统类(ubuntu/debian/centos)等，我们可以直接拿来使用。
- 2）除了选择现有的镜像作为基础镜像外，Docker还存在一个特殊的镜像，名为`scratch`。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。

##### 2、RUN: 执行命令

run指令是用来执行命令行命令的，由于命令行的强大能力，run指令在定制镜像时是最常用的指令之一。其格式有两种：

- shell格式： `RUN 命令`，就像直接在命令行中输入的命令一样，如`RUN echo 'hello, world!' > hello.txt`
- exec格式：`RUN ['可执行文件', '参数1', '参数2']`，类似于函数调用，将可执行文件和参数分开，如`RUN [ "sh", "-c", "echo $HOME" ]`

> `Dockerfile`中每一个指令都会建立一层，`RUN`也不例外。每一个RUN的行为，就和刚才我们手工建立镜像的过程一样:新建立一层，在其上执行这些命令，执行结束后，commit这一层的修改，构成新的镜像。所以我们在使用的时候尽可能将指令进行整合（可以使用&&将各个所需命令串联起来）。

#### 3、CMD：容器启动命令

CMD 指令的格式和 RUN 相似，也是两种格式：

- shell 格式：`CMD <命令>`
- exec 格式：`CMD ["可执行文件", "参数1", "参数2"...]`

在指定了 `ENTRYPOINT` 指令后，用 CMD 指定具体的参数。参数列表格式：`CMD ["参数1", "参数2"...]`

#### 4、COPY：复制文件

和 RUN 指令一样，也有两种格式，一种类似于命令行，一种类似于函数调用：

- `COPY <源路径>... <目标路径>`
- `COPY ["<源路径1>",... "<目标路径>"]`

COPY 指令将从构建上下文目录中`<源路径>`的文件复制到新的一层的镜像内的`<目标路径>`位置。

#### 5、ENV: 设置环境变量

格式有两种：

- `ENV <key> <value>`
- `ENV <key1>=<value1> <key2>=<value2>...`

这个指令很简单，就是设置环境变量而已，无论是后面的其它指令，如`RUN`，还是运行时的应用，都可以直接使用这里定义的环境变量。

```
ENV VERSION=1.0 DEBUG=on \
    NAME="Happy Feet"
```

这个例子中演示了如何换行，以及对含有空格的值用双引号括起来的办法，这和 Shell 下的行为是一致的。

#### 6、EXPOSE: 声明端口

格式为 `EXPOSE <端口1> [<端口2>...]`。

`EXPOSE`指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不会因为这个声明应用就会开启这个端口的服务。在`Dockerfile`中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是`docker run -P`时，会自动随机映射`EXPOSE`的端口。

要将`EXPOSE`和在运行时使用`-p <宿主端口>:<容器端口>`区分开来。`-p`，是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而`EXPOSE`仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。

#### 7、WORKDIR: 指定工作目录

格式为 `WORKDIR <工作目录路径>`。

使用 `WORKDIR` 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，`WORKDIR` 会帮你建立目录。

### 构建Node web服务镜像

![docker-node-logo](/images/docker-dockerfile/docker-node-logo.jpg)

#### 1、使用Express创建一个示例项目

##### (1) 安装express应用生成器工具

```
npm install express-generator -g
```

##### (2) 创建示例项目

```
express myapp
```

#### 2、创建`.dockerignore`文件

在myapp文件夹下面，新建一个`.dockerignore`的文件，相当于`.gitignore`，可以将一些不需要的文件进行忽略。示例代码如下：

```
node_modules/  
.git
.gitignore
.idea
.DS_Store
*.swp
*.log
```

#### 3、创建Dockerfile文件

在myapp文件夹下面，新建`Dockerfile`文件，添加如下代码：

```
FROM daocloud.io/node:5
MAINTAINER hhtczengjing@gmail.com
ENV PORT 3000
COPY . /app
WORKDIR /app
RUN npm install --registry=https://registry.npm.taobao.org
EXPOSE 3000
CMD ["npm", "start"]
```

#### 4、构建镜像

```
docker build -t myapp:1.0.0 .
```

![](/images/docker-dockerfile/docker-build.png)

在终端执行这行命令之后，使用`docker images`可以查看到当前所有的镜像：

![](/images/docker-dockerfile/docker-images.png)

#### 5、创建容器

```
docker run -d -p 3000:3000 myapp:1.0.0
```

浏览器访问：http://localhost:3000， 如果出现下面的界面表示成功：

![](/images/docker-dockerfile/hello-express.png)

### 参考资料

1、[《Dockerfie 官方文档》](https://docs.docker.com/engine/reference/builder/)

2、[《Dockerfile 最佳实践文档》](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)

3、[《入门实战：使用Docker构建一个nodejs服务》](https://onbing.com/first-blog/)

4、[《在Docker中运行Node.js的Web应用》](http://blog.shiqichan.com/Dockerizing-a-Node-js-Web-Application/)

5、[《Docker — 从入门到实践》](https://www.gitbook.com/book/yeasy/docker_practice/details)
