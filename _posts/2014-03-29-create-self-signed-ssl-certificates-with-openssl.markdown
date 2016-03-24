---
layout: post
title: "使用OpenSSL创建自签名证书"
date: 2014-03-29 21:28:24 +0800
comments: true
tags: Note
---

近期苹果把iOS系统更新到了7.1，如果用户更新到这个版本后，用原来的方式下载企业级应用，如果应用的plist文件不是用HTTPS方式部署的，那么会提示服务器上的证书无效，具体的如下图所示：

![ios-https-error.png](/images/ssl-https/ios-https-error.png)

在iOS7.1的企业级部署中需要将plist文件的URL路径设置为HTTPS才能正常下载安装，如果之前使用的是HTTP部署的，那么就需要对服务器的配置做一些改动。网上有提示说把plist文件上传到DropBox类似的文件存储服务器可以解决没有HTTPS服务器的缺憾，这个经过本人测试确实可行。但是对于那些部署在企业内网的服务来说，确实带来了很多不必要的麻烦。

在网上找了很多参考资料，都说使用自签名的SSL证书是可以的，但是需要把证书（最好是CA证书）通过Email或者是配置工具安装到iOS设备上（经过测试其实也是可以直接放在网页上提供用户直接下载的），然后根据之前创建的CA证书创建服务器上用到的证书。其实网上也有很多资料来介绍怎么创建SSL证书的，但是对于对服务器配置不是很熟悉的开发人员来说确实是一个噩梦，这里我整理了一下我这边在配置的时候用的几个步骤只要按照这几个步骤一步步的进行下去就没有什么问题了。

####Step 1.生成CA根证书
```
openssl genrsa -out myCA.key 2048
openssl req -new -x509 -key myCA.key -out myCA.cer -days 36500
```

执行上面的两个操作之后会提示输入以下几个内容(为了显示正常尽量使用英文)：

* Country Name (2 letter code) [AU]:CN //国家简称
* State or Province Name (full name) [Some-State]:GuangDong //州或省的名字
* Locality Name (eg, city) []:ShenZhen //区或市县的名称
* Organization Name (eg, company) [Internet Widgits Pty Ltd]:Comapny //公司或组织名
* Organizational Unit Name (eg, section) []:Mobile //单位或者是部门名称
* Common Name (e.g. server FQDN or YOUR name) []:zengjing //服务器名称或者昵称
* Email Address []:hhtczengjing@gmail.com  //Email地址

####Step 2.生成服务器端私钥和证书请求【CN的内容是服务器的域名或者是IP地址】
```
openssl genrsa -out server.key 2048
openssl req -new -out server.req -key server.key -subj /CN=192.168.1.101
```

####Step 3.通过CA签发证书
```
openssl x509 -req -in server.req -out server.cer -CAkey myCA.key -CA myCA.cer -days 36500 -CAcreateserial -CAserial serial
```

####Step 4.生成pem格式证书
```
cat server.cer server.key > server.pem
```

####Step 5.生成pkcs12格式证书【这个步骤会提示输入密码的请牢记自己输入的密码后面会用到的】
```
openssl pkcs12 -export -in server.cer -inkey server.key -out tomcat.p12 -name tomcat -CAfile myCA.cer -caname root -chain
```

【把证书转换成X509格式,用于IIS，这个可选对于部署在IIS上面的服务才有效】

```
openssl pkcs12 -export -clcerts -in server.cer -inkey server.key -out iis.pfx
```

####Step 6.加入信任证书【需要JAVA JRE环境，如果是iis不需要这个步骤】
```
keytool -keystore truststore.jks -keypass 123456 -storepass 123456 -alias ca -import -trustcacerts -file server.pem
```

按下回车之后会提示是否信任的，直接输入一个y即可（可能出现中文无法显示?????????֤?? [??]??  y）

####Step 7.配置Tomcat和IIS
* Tomcat<br/>
  需要到tomcat的conf目录下面修改server.xml的文件中的部分内容，增加以下部分，端口号8443按照具体的需要来进行设置
  
  ```
   <Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"           maxThreads="150" scheme="https" secure="true"           clientAuth="false" sslProtocol="TLS"            keystoreFile="tomcat.p12" keystorePass="888888" keystoreType="PKCS12"           truststoreFile="truststore.jks"  truststorePass="888888" truststoreType="JKS"/>
```           
* IIS<br/>
  IIS的配置是相当的简单直接在服务器证书那里选择导入pfx就行了，其他的就是创建HTTPS的问题了，在此就不作过多的描述。