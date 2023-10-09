---
layout: post
title: "Docker配置JDK的Security Provider"
date: 2023-10-08 23:50:16 +0800
comments: true
tags: Note
---

最近将之前写的一个消息通知的服务支持通过Docker进行部署，以前一直都是通过命令行启动的。一般的流程就是编写Dockerfile文件编译镜像运行容器就大功告成了。

Dockerfile配置如下：

```
FROM openjdk:8
VOLUME /tmp/data
ADD app.jar app.jar
ENV JAVA_OPTS=""
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar --spring.profiles.active=docker" ]
```

编译镜像

```
docker build -t zengjing/pep_notifier:0.1.0 .
```

运行容器

```
docker run -d --name pep_notifier --link mysql57:mysqldb --restart=always -p 8080:8080 zengjing/pep_notifier:0.1.0
```

到这里一切顺利，运行过程中发现消息发送大量的报错，错误信息如下：

```
java.security.NoSuchAlgorithmException: Cannot find any provider supporting RSA/None/NoPadding
```

以前通过命令行运行的时候也有类似的报错，找到了一个大神的文章(https://www.cnblogs.com/greys/p/10950525.html)修改了JDK得以解决。主要做法就是修改 `java.security` 文件添加扩展的加密套件。现在的问题就是如何在Docker中实现JDK的修改呢？

(1) 下载 bcprov-jdk15on-1.58.jar

```
curl https://repo1.maven.org/maven2/org/bouncycastle/bcprov-jdk15on/1.58/bcprov-jdk15on-1.58.jar -o bcprov-jdk15on-1.58.jar
```

(2) 获取 java.security 并修改配置

前面已经运行起来容器，可以通过命令拷贝容器的文件到宿主机

```
docker cp pep_notifier:/usr/local/openjdk-8/jre/lib/security/java.security .
```

对拷贝出来的文件添加如下行（搜索 security.provider 跟在后面添加，注意后面的数字是连续递增的）

```
security.provider.10=org.bouncycastle.jce.provider.BouncyCastleProvider
```

(3) 修改 Dockerfile

在之前的版本基础上增加两行，主要是拷贝并替换 java.security 配置内容

```
ADD security/java.security ${JAVA_HOME}/jre/lib/security/
ADD security/bcprov-jdk15on-1.58.jar ${JAVA_HOME}/jre/lib/ext/
```

做完上面的步骤再重新构建镜像，运行容器就OK了。再次运行前需要先删除之前的容器，参考如下命令：

```
docker stop pep_notifier # 停止容器
docker rm pep_notifier # 删除容器
```

### 参考资料

- 1、[java.security.NoSuchAlgorithmException: Cannot find any provider supporting RSA](https://www.cnblogs.com/greys/p/10950525.html)