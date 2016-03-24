---
layout: post
title: "Java使用JNDI配置WEB项目数据源"
date: 2014-08-06 21:28:49 +0800
comments: true
tags: Java
---

`JNDI(Java Naming and Directory Interface,Java命名与目录接口)`是Java提供的一种标准的命名系统接口，JNDI提供统一的客户端API，通过不同的访问提供者接口`JNDI SPI(Service Provider Interface,服务提供者接口)`的实现，由管理者将JNDI API映射为特定的命名服务和目录系统，使得Java应用程序可以和这些命名服务和目录服务之间进行交互。

##使用传统的方式

一般对于普通的项目，习惯上使用`.properties`的文件来配置数据源的一些信息。在`.properties`文件中配置数据源连接字符串、用户名、密码和驱动的类，然后在代码中读取配置文件的信息再通过`DriverManager.getConnection(url,  username, password)`的方式获取数据源连接对象，步骤如下,以SQLServer为例:

(1)注册驱动

`Class.forName(“com.microsoft.sqlserver.jdbc.SQLServerDriver”);`

(2)获取连接对象

`Connection conn = DriverManager.getConnection(url,  username, password);`

(3)示例代码

```
public static Connection getDefaultConnection() {
	try {
		String url = "jdbc:sqlserver://localhost:1433;databaseName=DBName";
		String username = "sa";
		String password = "*";
		String driver = "com.microsoft.sqlserver.jdbc.SQLServerDriver";
		//注册SQLServer的驱动
		Class.forName(driver);
		//获取数据源连接对象
		Connection conn = DriverManager.getConnection(url,  username, password);
		return conn;
	} catch (Exception e) {
		e.printStackTrace();
		return null;
	}
}
```

##使用JNDI的方式

1、获取数据源的连接对象`Connection`

```
public static Connection getConnection(String jndi) {
	DataSource datasource = null;
	Connection connection = null;
	try {
		Context context = new InitialContext();
		Context envContext = null;
		try {
			envContext = (Context) context.lookup("java:/comp/env");
		} catch (Exception e1) {
			// 无法通过java:方式获得换用/comp/env的方式
			try {
				envContext = (Context) context.lookup("/comp/env");
			} catch (Exception fff) {
				e1.printStackTrace();
			}
		}
		//如果数据源的名称不为空的话使用指定的数据源的名称来获取数据库连接对象
		if(StringUtils.isNotEmpty(jndi)) {
			datasource = (DataSource) envContext.lookup(jndi);
		} else {
			datasource = (DataSource) envContext.lookup("sqlserver/default");
		}
		connection = datasource.getConnection();
	} catch (Exception e2) {
		e2.printStackTrace();
	}
	return connection;
}
```
2、在Tomcat中的webapp中加入`Resource`配置

```
<Resource name="sqlserver/default"
	auth="Container"
	type="javax.sql.DataSource"
	maxActive="100"
	maxIdle="30"
	maxWait="10000"
	username="sa"
	password="*"
	driverClassName="com.microsoft.sqlserver.jdbc.SQLServerDriver"
	url="jdbc:sqlserver://localhost:1433;databasename=DBNAME"/>
```
常见配置属性描述:

- name：JDBC数据源的名称

- auth：认证方式，一般设置为`Container`,还有一种是`Application`

- type：当前配置资源的类别

- factory：数据源工厂,默认为"org.apache.commons.dbcp.BasicDataSourceFactory"

- driverClassName：驱动的全路径类名

- maxActive：当前数据源支持的最大并发数

- maxIdle：连接池中保留最大数目的闲置连接数

- maxWait：当连接池中无连接时的最大等待毫秒数，在等当前设置时间过后还无连接则抛出异常

- username：访问数据库的用户名

- password：访问数据库的密码

- url：JDBC驱动的连接字符串

- validationQuery：在返回应用之前，用于校验当前连接是否有效的SQL语句，如果指定了，当前查询语句至少要返回一条记录,可以写成`select top 1 * from sysobjects`

3、在Tomcat的`lib`目录下面添加数据库的驱动文件

MySQL:`mysql-connector-java-5.1.20-bin.jar`

SQLServer:`sqljdbc4.jar`

Oracle:`ojdbc14.jar`

4、在web.xml加入如下配置

```
<resource-ref>
	<res-ref-name>sqlserver/default</res-ref-name>
	<res-type>javax.sql.DataSource</res-type>
	<res-auth>Container</res-auth>
</resource-ref>
```
注:属于可选配置，如果在web.xml中加入了上面的配置，则需要在Tomcat中一定要配置对应的`Resource`，否则会报错。

5、使用方式

```
Connection conn = getConnection("sqlserver/default");
...
```

##参考资料

1、[《Java使用JNDI技术获取DataSource对象》](http://www.cnblogs.com/cyjch/archive/2012/03/28/2420806.html)

2、[《JNDI配置原理详解》](http://nything.iteye.com/blog/420018)

3、[《JAVA配置JNDI数据源》](http://blog.csdn.net/xuhuanchao/article/details/4862895)

4、[《JNDI 是什么》](http://blog.csdn.net/zhaosg198312/article/details/3979435)

5、[《Web项目开发环境中运行在Tomcat时涉及到JNDI的Datasource的解决方法》](http://blog.csdn.net/kkdelta/article/details/7301965)

6、[《tomcat下jndi的三种配置方式》](http://blog.csdn.net/lgm277531070/article/details/6711177)