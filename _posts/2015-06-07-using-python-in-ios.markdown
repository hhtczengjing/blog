---
layout: post
title: "合理使用Python助力iOS开发"
date: 2015-06-07 03:37:56 +0800
comments: true
tags: Python
---

说明：本文并不是介绍关于Python如何使用或者是Python的语法知识的，重点是分享在开发中使用Python来减少重复性的劳动的思路。

![python_logo.png](/images/python_ios/python_logo.png)

[Python](https://www.python.org)是一种面向对象、解释型计算机程序设计语言，由`Guido van Rossum`于1989年底发明，第一个公开发行版发行于1991年，Python 源代码同样遵循 GPL(GNU General Public License)协议 。Python语法简洁而清晰，具有丰富和强大的类库。它常被昵称为胶水语言，能够把用其他语言制作的各种模块（尤其是C/C++）很轻松地联结在一起。

Python就为我们提供了非常完善的基础代码库，覆盖了网络、文件、GUI、数据库、文本等大量内容，被形象地称作“内置电池（batteries included）”。用Python开发，许多功能不必从零编写，直接使用现成的即可。

除了内置的库外，Python还有大量的第三方库，也就是别人开发的，供你直接使用的东西。当然，如果你开发的代码通过很好的封装，也可以作为第三方库给别人使用。

许多大型网站就是用Python开发的，例如YouTube、Instagram，还有国内的豆瓣。很多大公司，包括Google、Yahoo等，甚至NASA（美国航空航天局）都大量地使用Python。

###安装Python第三方模块

在Python中，安装第三方模块，是通过包管理工具pip完成的，在终端中尝试运行pip如果提示未找到该命令，需要先安装pip。

(1)下载pip的安装脚本

```
wget https://bootstrap.pypa.io/get-pip.py
```

(2)执行安装脚本，安装pip

```
sudo python get-pip.py 
```

一般来说，第三方库都会在Python官方的`pypi.python.org`网站注册，要安装一个第三方库，必须先知道该库的名称，可以在官网或者pypi上搜索。比如使用比较频繁的一个用于解析HTML的库BeautifulSoup，因此安装BeautifulSoup的命令为：

```
pip install BeautifulSoup
```
只要耐心等待下载并安装后，就可以使用了。另外其他常用的第三方库还有MySQL的驱动：mysql-connector-python，用于科学计算的NumPy库：numpy，用于生成文本的模板工具Jinja2，等等。


###案例

1、抓取`http://blog.devzeng.com`网站下面的所有文章的名称和链接。适用于模拟爬虫的功能，将某些网站上面的数据抓取回来。

```
#!/usr/bin/python
#-*- coding: utf-8 -*-
#encoding=utf-8

import urllib,re,json,time,sys
from BeautifulSoup import BeautifulSoup

#获取指定的URL的内容
def getHtmlContent(url):
    page = urllib.urlopen(url)
    html = page.read()
    return html

#获取全部的文章的链接
def getAllHtmlLink(html, url):
	arr = []
	pattern = u'<a.*?href="(.+)".*?>(.*?)</a>'
	soup = BeautifulSoup(html)
	articleResult = soup.findAll('article')
	for article in articleResult:
		h1Result = article.findAll('h1')
		for h1 in h1Result:
			aResult = h1.findAll('a')
			for a in aResult:
				ret = re.search(pattern, str(a))
				if ret:
					map = {}
					group = ret.groups()
					map['title'] = group[1]
					map['link'] = url + group[0]
					arr.append(map)
	return arr

#转换成JSON字符串
def toJSONString(arr):
	return json.dumps(arr)

#保存内容到文件
def saveToFile(path, json):
	fp = open(path, 'w')
	fp.write(json)
	fp.close()

if __name__=="__main__":
	start = time.time()
	url = "http://blog.devzeng.com"
	html = getHtmlContent(url + "/blog/archives")
	result = getAllHtmlLink(html, url)
	json = toJSONString(result)
	print type(json.encode('utf-8').decode('utf-8'))
	saveToFile("blog.txt", json)
	c = time.time() - start
	print('执行完成，程序运行耗时:%0.2fs'%(c))
```

说明：

(1)依赖BeautifulSoup模块，使用前需要先安装`BeautifulSoup`

(2)思路是先读取网址的HTML内容，然后解析HTML中的指定的链接标签分析里面的数据然后整合到一起

(3)def开头的是函数，函数名后面有冒号，没有花括号的区分用代码缩进来判断是不是一个代码块的内容。

2、前段时间的项目是使用WebService作为网络API接口，为了保证请求参数的完整和顺序的一致需要将WSDL文件下载下来进行解析。但是每次手动的处理都是一件很费体力的事情，而且经常容易遗漏一些东西。使用Python之后就极大的方便了我们的开发。

```
#!/usr/bin/python
#-*- coding: utf-8 -*-
#encoding=utf-8

import urllib,re
from BeautifulSoup import BeautifulSoup

#获取指定的URL的内容
def getHtmlContent(url):
    page = urllib.urlopen(url)
    html = page.read()
    return html

#获取全部的超链接的内容
def getAllHrefLink(url, html):
        soup = BeautifulSoup(html)
        aResult = soup.findAll('a')
        pattern = u'<a.*?href="(.+)".*?>(.*?)</a>'
        for a in aResult:
            ret = re.search(pattern, str(a))
            if ret:
                group = ret.groups()
                name = group[1].replace('.asmx', '.xml')
                link = url+group[0]
                if name != u'[To Parent Directory]':
                    saveData(name, link)

#保存文本内容到指定的路径                
def saveData(path, content):
    f = open(path,'w+')
    f.write(content)

#调用函数
url = "http://192.168.1.101:8080"
content = getHtmlContent(url + "/Services/")
getAllHrefLink(url, content)
```

###参考资料

1、[《Python官网》](https://www.python.org)

2、[《Python 3.0教程》](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

3、[《Python 2.7教程》](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000)

4、[《python字符串操作》](http://www.cnblogs.com/SunWentao/archive/2008/06/19/1225690.html)
