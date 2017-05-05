---
layout: post
title: "Jersey整合Swagger自动生成API文档"
date: 2017-05-04 13:51:00 +8000
comments: true
tags: Java
---

之前写过一篇文章[《使用Jersey开发REST服务》](http://blog.devzeng.com/blog/java-restful-with-jersey.html)，里面简单介绍了使用Jersey来快速创建REST的API服务。

REST API都是要对外提供服务的，那么文档是必须的。经常要给其他人员提供文档，每次都是要不断的维护word/excel的文件，挺麻烦的。能不能做到自动生成呢？答案是可以的，swagger就是这样的一个组件帮助我们快速生成，让开发人员只需要关注功能的开发即可，后续的工作就交给Swagger就好了。

下面简单介绍下如何在Jersey的项目中集成Swagger。

### 1、pom.xml加入swagger的依赖

```
<dependency>
    <groupId>io.swagger</groupId>
    <artifactId>swagger-jersey2-jaxrs</artifactId> 
    <version>1.5.0</version>
</dependency>
```

### 2、修改web.xml配置

#### (1) 修改Jersey配置

```
<!-- jersey -->
<servlet>
    <servlet-name>jersey</servlet-name>
    <servlet-class>org.glassfish.jersey.servlet.ServletContainer</servlet-class>
    <init-param>
        <param-name>jersey.config.server.provider.packages</param-name>
        <param-value>io.swagger.jaxrs.listing,com.devzeng.service.schedule.api</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>jersey</servlet-name>
    <url-pattern>/api/*</url-pattern>
</servlet-mapping>
```

#### （2）新增swagger配置

```
<!-- swagger -->
<servlet>
    <servlet-name>Jersey2Config</servlet-name>
    <servlet-class>io.swagger.jersey.config.JerseyJaxrsConfig</servlet-class>
    <init-param>
        <param-name>api.version</param-name>
        <param-value>1.0.0</param-value>
    </init-param>
    <init-param>
        <param-name>swagger.api.basepath</param-name>
        <param-value>http://localhost:8080/api</param-value>
    </init-param>
    <load-on-startup>2</load-on-startup>
</servlet>
```

说明：

- `swagger.api.basepath`:这个是api访问的baseurl

### 3、配置注解

### 4、配置swagger-ui

#### (1) 下载swagger-ui

`https://github.com/swagger-api/swagger-ui`

推荐使用2.x版本

#### (2) 拷贝dist目录下面的文件到webroot下面

#### (3) 修改index.html页面的url地址

```
url = "http://localhost:8080/api/swagger.json";
```

生成的文档效果如下：

![swagger-demo.jpg](/images/jersey-swagger/swagger-demo.jpg)

### 参考资料

1、[Swagger Core Jersey 2.X Project Setup 1.5](https://github.com/swagger-api/swagger-core/wiki/Swagger-Core-Jersey-2.X-Project-Setup-1.5#configure-and-initialize-swagger)

2、[Swagger-Core Annotations](https://github.com/swagger-api/swagger-core/wiki/Annotations-1.5.X)
