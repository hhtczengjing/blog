---
layout: post
title: "使用Jersey开发REST服务"
date: 2017-03-19 10:40:00 +8000
comments: true
tags: Java
---

REST 是英文 `Representational State Transfer` 的缩写，有中文翻译为“`表述性状态转移`”。REST 这个术语是由 Roy Fielding 在他的博士论文 [《 Architectural Styles and the Design of Network-based Software Architectures 》](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)中提出的。REST 并非标准，而是一种开发 Web 应用的架构风格，可以将其理解为一种设计模式。REST 基于 HTTP，URI，以及 XML 这些现有的广泛流行的协议和标准，伴随着 REST，HTTP 协议得到了更加正确的使用。

Jersy是一个业内使用非常广泛的Java Rest框架，本文就Jersey（2.13版本）的快速使用进行简单介绍，如需要了解更多的高级用法请查看官方的文档。

### 1、在pom.xml中加入jersey相关依赖

```
<dependency>
    <groupId>org.glassfish.jersey.containers</groupId>
    <artifactId>jersey-container-grizzly2-servlet</artifactId>
    <version>2.13</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.containers</groupId>
    <artifactId>jersey-container-servlet-core</artifactId>
    <version>2.13</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-json-jackson</artifactId>
    <version>2.13</version>
</dependency>
```

### 2、配置web.xml文件

```
<servlet>
    <servlet-name>Jersey REST Service</servlet-name>
    <servlet-class>org.glassfish.jersey.servlet.ServletContainer</servlet-class>
    <init-param>
        <param-name>jersey.config.server.provider.packages</param-name>
        <param-value>com.devzeng.demo.api</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>Jersey REST Service<rvlet-name>
	<url-pattern>/api/*</url-pattern>
</servlet-mapping>
```

### 3、开发

```
@Path("/api")
public class HelloApiService {

    @POST
	@Path("save")
	@Consumes(MediaType.APPLICATION_FORM_URLENCODED)
	@Produces(MediaType.APPLICATION_JSON)
	public String save(@FormParam("data") String data) {
		return "{\"message\":\"save\"}";
	}
	
	@GET
	@Path("list")
	@Produces(MediaType.APPLICATION_JSON)
	public String list(@QueryParam("from") String from, @QueryParam("to") String to) {
		return "{\"message\":\"list\"}"; 
	}
	
	@GET
	@Path("detail/{id}")
	@Produces(MediaType.APPLICATION_JSON)
	public String detail(@PathParam("id") String id) {
		return "{\"message\":\"detail\"}"; 
	}
}
```

说明：

#### (1)跨域问题解决

如果编写的API接口需要给前端进行调用，通常会遇到跨域的问题，可以使用下面的方式进行解决：

```
@Provider
public class SceduleApiServiceCorsFilter implements ContainerResponseFilter {

	public void filter(ContainerRequestContext creq, ContainerResponseContext cres) throws IOException {
		cres.getHeaders().add("Access-Control-Allow-Origin", "*");
        cres.getHeaders().add("Access-Control-Allow-Headers", "origin, content-type, accept, authorization");
        cres.getHeaders().add("Access-Control-Allow-Credentials", "true");
        cres.getHeaders().add("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS, HEAD");
        cres.getHeaders().add("Access-Control-Max-Age", "1209600");
	}
	
}
```

#### (2)JSON & XML处理

对于REST的接口通常需要返回的数据格式是JSON、XML。如果每次都是使用JSONObject这样的库来进行拼接，也是一件很麻烦的事情，为何不能直接返回对应的POJO对象呢。Jersey就支持这样的处理，为了让项目结构比较清晰，推荐建立一个单独的package（如com.devzeng.rest.pojo）,在该package创建一个POJO对象`MyCustomBean`。

```
public class MyCustomBean {

    private String name;
    private int age;

    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public int getAge() {
        return age;
    }
    
    public void setAge(int age) {
        this.age = age;
    }
}
```

##### 1)JSON处理

```
@GET
@Path("hellojson")
@Produces(MediaType.APPLICATION_JSON)
public MyCustomBean sayHelloWithJson() {
    MyCustomBean bean = new MyCustomBean();
    bean("tom");
    bean.setAge(20);
    return bean;
}
```

说明：

① `Produces`注解需要指定返回的数据格式是JSON格式(`MediaType.APPLICATION_JSON`)。

② 如果启动之后报如下错误：

```
org.glassfish.jersey.message.internal.WriterInterceptorExecutor$TerminalWriterInterceptor aroundWriteTo
```

表示POJO对象没有被序列化成JSON对象，需要添加相关的库，推荐使用`jersey-media-json-jackson`模块：

```
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-json-jackson</artifactId>
    <version>2.13</version>
</dependency>
```

##### 2) XML处理

```
@GET
@Path("helloxml")
@Produces(MediaType.APPLICATION_XML)
public MyCustomBean sayHelloWithXML() {
    MyCustomBean bean = new MyCustomBean();
    bean("tom");
    bean.setAge(20);
    return bean;
}
```

① `Produces`注解需要指定返回的数据格式是XML格式(`MediaType.APPLICATION_XML`)。

② 启动项目之后如果报如下错误：

```
org.glassfish.jersey.message.internal.WriterInterceptorExecutor$TerminalWriterInterceptor aroundWriteTo
```

需要在POJO对象上面加上`@XmlRootElement`，`@XmlRootElement`表示将一个类或者是枚举类型映射成为一个XML元素。

#### （3）中文乱码问题

1) 推荐将项目的所有格式设置为UTF-8;

2) 如果还存在中文乱码的问题，需要将

```
@Produces(MediaType.APPLICATION_JSON + ";charset=utf-8")
```

### 参考资料

1.[《Jersey官方文档》](https://jersey.java.net/)

2.[《REST 实战》](https://www.gitbook.com/book/waylau/rest-in-action)