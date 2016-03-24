---
layout: post
title: "使用SQLCipher加密SQLite数据库"
date: 2015-07-19 20:02:54 +0800
comments: true
tags: iOS
---

在iOS应用程序开发中经常需要使用SQLite来存储数据，很多时候需要加密保存一部分的数据。常见的做法是将要存储的内容先加密然后存到数据库中，使用的时候将数据进行解密，这样就会有大量的性能消耗在数据的加解密上（性能主要取决于加解密的算法和CPU的能力）。

> SQLCipher is an open source extension to SQLite that provides transparent 256-bit AES encryption of database files.

SQLite本身是支持加密功能的（免费版本不提供加密功能，商业版本是支持加密模块）。SQLCipher是一个开源的SQLite加密扩展，支持对db文件进行256位的AES加密。

###集成SQLCipher

集成SQLCipher有有两种方法一种是按照[官方](https://www.zetetic.net/sqlcipher/ios-tutorial/)的方式一步步的执行，这里就不过多的介绍。配置过程很麻烦，推荐使用下面的方式集成。

1、获取SQLite加密模块(SQLCipher)

在终端(Terminal)中使用`pod search FMDB`，在查询的结果中可以看到有个`FMDB/SQLCipher`的Sub spec。

![sqlcipher-search-fmdb.png](/images/sqlcipher-sqlite/sqlcipher-search-fmdb.png)

如果使用FMDB和CocoaPods的话直接在你的Podfile中添加`pod 'FMDB/SQLCipher'`

![sqlcipher-podfile.png](/images/sqlcipher-sqlite/sqlcipher-podfile.png)

如果没有使用CocoaPods的话建议还是安装一个吧，或者是新建一个测试项目安装FMDB和SQLCipher。安装CocoaPods可以参考[《使用CocoaPods管理iOS项目中的依赖库》](http://blog.devzeng.com/blog/ios-cocoapods-dependency-manager.html)

2、导入SQLCipher

执行`pod install`之后会自动获取SQLCipher，其实SQLCipher只有两个文件`sqlite3.h`和`sqlite3.m`。

拷贝sqlite3.h/sqlite3.m文件到项目中，如果使用CocoaPods方式获取SQLCipher的话，这一步骤就不需要了。

3、配置Xcode设置项

通过查询资料SQLite是否开启加密模块是通过宏(`SQLITE_HAS_CODEC`)来配置的。那么就需要在Xcode中配置开启SQLite加密组件的宏（如使用CocoaPods方式则不需要配置）。

![sqlite-extension-marco.png](/images/sqlcipher-sqlite/sqlite-extension-marco.png)

(1)target -> Build Setting -> Other C Flags添加-DSQLITE_HAS_CODEC、-DSQLITE_TEMP_STORE=2、-DSQLITE_THREADSAFE、-DSQLCIPHER_CRYPTO_CC几项配置

(2)target -> Build Setting -> Other Linker Flags添加-framework Security配置

4、如何使用

(1)引入sqlite3加密模块，然后在打开数据库之后加入如下代码

```
const char *key = [@"devzeng" UTF8String];
sqlite3_key(_db, key, (int)strlen(key));
```

如下图：

![sqlcipher-sqlite3-open.png](/images/sqlcipher-sqlite/sqlcipher-sqlite3-open.png)

说明：

1）如果没有添加-DSQLITE_HAS_CODEC配置上面的代码会报错

2）sqlite3_key函数需要指定加密使用的key，推荐使用UUID(可以进行salt处理)并存储到KeyChain中。

3）如使用FMDB，可以在FMDB的open方法之后添加上面的两行代码。

(2)使用了加密模块在提交到App Store时需要指明，具体的操作方法可以参考StackOverflow上面的做法。
[Does my application “contain encryption”?](http://stackoverflow.com/questions/2135081/does-my-application-contain-encryption)

###参考资料

1、[《Adding SQLCipher to Xcode Projects》](https://www.zetetic.net/sqlcipher/ios-tutorial/)

2、[《ios开发FMDB导入SQLCipher加密数据库》](http://www.2cto.com/kf/201407/315727.html)

3、[《SQLite数据库加密研究》](http://blog.itpub.net/14466241/viewspace-752861/)
