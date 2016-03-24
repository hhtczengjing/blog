---
layout: post
title: "在StartSSL上申请免费SSL证书"
date: 2014-10-20 22:18:54 +0800
comments: true
tags: Note
---

   跟VeriSign一样，StartSSL（`http://www.startssl.com`，公司名：StartCom）也是一家CA机构，它的根证书很久之前就被一些具有开源背景的浏览器支持（Firefox浏览器、谷歌Chrome浏览器、苹果Safari浏览器等）。在去年9月份，StartSSL竟然搞定了微软：微软在升级补丁中，更新了通过Windows根证书认证程序（Windows Root Certificate Program）的厂商清单，并首次将StartCom公司列入了该认证清单，这是微软首次将提供免费数字验证技术的厂商加入根证书认证列表中。现在，在Windows 7或安装了升级补丁的Windows Vista或Windows XP操作系统中，系统会完全信任由StartCom这类免费数字认证机构认证的数字证书，从而使StartSSL也得到了IE浏览器的支持。
   
   ![startssl_logo.jpg](http://www.startssl.com/img/top-logo1.jpg)

##StartSSL个人证书登录申请

1、在浏览器中输入：`https://www.startssl.com/?app=12`，如果是第一次使用需要进行注册，选择Sign-up,如下图所示：

![startssl_free_certificate_001](/images/startssl_free_certificate/startssl_free_certificate_001.png)

2、在注册表单中填写指定的内容，需要填写姓名（First/Last Name）、家庭住址（Home Address）、邮编（Zip）等，具体如下图所示：

![startssl_free_ certificate_002](/images/startssl_free_certificate/startssl_free_certificate_002.png)

注意：

（1）所示字段都需要完整填写；

（2）Complete Home Address：需要填写的是完整的家庭住址，最好是和IP所在地一致。比如是在深圳注册的，那么家庭住址需要填写一个深圳的地址；

（3）填写的家庭住址不要明显的表面是一个商业的住址，如`14/F, BAK Tech Buld`。这种类型的地址会被打回来提示需要用邮件补一份家庭住址；

（4）邮箱不要使用QQ邮箱，这里QQ不支持。

3、填写完成后点击Continue按钮，接下来StartSSL官方会发送一封邮件到上面填写的邮箱地址上，里面有一个验证码，把验证码输入到下面的框中点击Continue即可。

![startssl_free_ certificate_003.png](/images/startssl_free_certificate/startssl_free_certificate_003.png)

4、验证成功后，要么是等待人工审核开通，要么是直接提示下载个人操作证书。我申请的时候是因为前面填写的地址和注册的IP所在地不一致直接被拒绝，第二次是因为家庭住址不规范被要求重新填了一次。通过审核后就会出现下面的页面下载个人操作的证书，这里是选择证书加密的位数默认选择`2048(高级)`，如图所示：

![startssl_free_ certificate_004.png](/images/startssl_free_certificate/startssl_free_certificate_004.png)

5、点击Continue后就会进入到下载安装的界面，如下图所示：

![startssl_free_ certificate_005.png](/images/startssl_free_certificate/startssl_free_certificate_005.png)

6、点击Install安装,然后在出现的界面中选择Finish就能开始下载证书，稍等几分钟就会自动在浏览器上安装StartSSL操作证书了

![startssl_free_certificate_006.png](/images/startssl_free_certificate/startssl_free_certificate_006.png)


##StartSSL免费证书创建

1、点击Validations Wizard，选择Domain name validation，点击Continue。

![startssl_free_certificate_007](/images/startssl_free_certificate/startssl_free_certificate_007.png)

2、输入你想要使用SSL的域名，点击继续

![startssl_free_ certificate_008](/images/startssl_free_certificate/startssl_free_certificate_008.png)

3、接下来选择你的域名所有者的邮箱，如果你的域名设置了Whois保护，可能还要先进入域名注册商那里取消保护，否则无法使用域名管理员邮箱通过域名所有权验证

![startssl_free_ certificate_009](/images/startssl_free_certificate/startssl_free_certificate_009.png)

4、提交了域名后，到邮箱中收取激活邮件，填入验证码，通过验证。

![startssl_free_certificate_010](/images/startssl_free_certificate/startssl_free_certificate_010.png)

验证通过后会有如下提示：

![startssl_free_certificate_011](/images/startssl_free_certificate/startssl_free_certificate_011.png)

5、上面已经提交了域名并通过了所有权验证，点击Certificates Wizard，选择WEB Server SSL/TSL Certifites。

![startssl_free_certificate_012](/images/startssl_free_certificate/startssl_free_certificate_012.png)

6、上面点击Continue后接下来会提示输入生成私钥需要的密码和指定位数。其中密码，最少10位，最大32位。

![startssl_free_ certificate_013](/images/startssl_free_certificate/startssl_free_certificate_013.png)

7、将显示内容保存为ssl.key（这个私钥是加密的），继续点击下一步。

![startssl_free_ certificate_014](/images/startssl_free_certificate/startssl_free_certificate_014.png)

8、接下来会提示需要选择要生成的域名下面的二级域名，如下这里我选择的是devzeng.com下面的ssl.devzeng.com这个二级域名，如下图示：

（1）选择前面绑定的域名：

![startssl_free_ certificate_015](/images/startssl_free_certificate/startssl_free_certificate_015.png)

（2）输入二级域名：

![startssl_free_ certificate_016](/images/startssl_free_certificate/startssl_free_certificate_016.png)

提交后还得再等待StartSSL审核，一般是几分钟后就可以收到邮件通知。当然也有审核不通过的，

![startssl_free_ certificate_017](/images/startssl_free_certificate/startssl_free_certificate_017.png)

9、这时进入Tool Box点击Retrieve Certificate，选择申请证书的域名，将框中的内容保存为.crt文件，这个就是域名证书了。

![startssl_free_certificate_018](/images/startssl_free_certificate/startssl_free_certificate_018.png)

##参考资料

1、[《全球唯一免费HTTPS证书颁发机构：StartSSL》](http://www.chinaz.com/free/2010/0114/103945.shtml)

2、[《StartSSL免费SSL证书成功申请-HTTPS让访问网站更安全》](http://www.freehao123.com/startssl-ssl/)
