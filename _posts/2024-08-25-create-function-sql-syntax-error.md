---
layout: post
title: "MySQL创建函数语法报错的解决过程"
date: 2024-08-25 11:40:00 +0800
comments: true
tags: Note
---

近期想把之前做的一个系统在本地跑起来做一些测试，结果在执行创建函数语句的时候报错了，解决的方式很简单，免得下次再遇到，记录一下解决过程。SQL语句如下：

```
DROP FUNCTION IF EXISTS fn_version_format;
CREATE FUNCTION fn_version_format(version VARCHAR(50))
    RETURNS VARCHAR(255)
BEGIN
    DECLARE v_con VARCHAR(255);
    SET v_con = version;
    // ... 省略代码 ...
    RETURN v_con;
END;
```

运行报错，错误信息如下：

```
Error Code: 1064. You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'END' at line 1
```

从错误信息来看其实是很明显的语法的错误，但是看代码，并没有发现什么错误。折腾了好一会，最后发现还是语法的问题。

> MySQL使用分号来分割SQL语句，但是函数体中包含分号，所以会被拆分成多个SQL语句，最后导致语法错误。

解决办法：就是临时切换一下分隔符(DELIMITER)，把函数体中的分号替换成其他的字符，比如`$`。

```
DROP FUNCTION IF EXISTS fn_version_format;
DELIMITER $ -- 临时切换分隔符
CREATE FUNCTION fn_version_format(version VARCHAR(50))
    RETURNS VARCHAR(255)
BEGIN
    DECLARE v_con VARCHAR(255);
    SET v_con = version;
    // ... 省略代码 ...
    RETURN v_con;
END$
DELIMITER ; -- 还原分隔符
```

以前都是使用 [DataGrip](https://www.jetbrains.com/datagrip/) 来连接 Mysql 数据库，挺好用的。但是由于一些政策方面的原因不能在当前的电脑上面安装，用了官方的[MySQL Workbench](https://www.mysql.com/products/workbench/)还是不太习惯，发现可以使用 [IntelliJ IDEA](https://www.jetbrains.com/idea/) 的 `Database Navigator` 插件，简单配置一下就可以直接使用，可以满足日常使用。这里记录一下过程：

（1）`Plugins -> Marketplace` 搜索 `Database Navigator` 插件，安装后重启IDEA。

![install-plugin](/images/create-function-sql-syntax-error/install-plugin.png)

（2）`View -> Tool Windows -> DB Browser` 打开数据库窗口

![db-browser-viewer](/images/create-function-sql-syntax-error/db-browser-viewer.png)

（3）点击`DB Browser -> + -> MySQL` 添加数据源

![add-source-1](/images/create-function-sql-syntax-error/add-source-1.png)

按照要求填写数据库连接信息，点击 `Test Connection` 测试连接

![add-source-2](/images/create-function-sql-syntax-error/add-source-2.png)

连接成功后，点击`OK` 添加数据源

（4）`Console -> mysql` 打开MySQL控制台，可以执行SQL语句

![execute-sql](/images/create-function-sql-syntax-error/execute-sql.png)

### 参考资料

- [1、idea社区版连接mysql数据库](https://blog.csdn.net/u010711495/article/details/111414259)

- [2、MySQL的自定义函数和存储过程](https://www.cnblogs.com/wenxuehai/p/15934125.html)

- [3、mysql创建函数报1064错误的解决方案](https://blog.csdn.net/tuolingss/article/details/121234411)