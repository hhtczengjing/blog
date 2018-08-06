---
layout: post
title: "Python使用pyexecjs执行JS代码"
date: 2018-07-20 11:46:29 +0800
comments: true
categories: Python
---

近期在采集一个网站的时候遇到一部分的页面是使用JS代码来填充数据的，代码如下：

```
<tr>
    <td width="150px" class="success">案件级别:</td>
    <td colspan="1">
        <script>
            var s = getDictLabel([{"id":"4191c4842b3842749dd467655f90b1fa","isNewRecord":false,"remarks":"案件级别","createDate":"2016-11-29 15:19:34","updateDate":"2016-11-29 15:19:34","value":"1","label":"一般事件","type":"case_grade","description":"案件级别","sort":1,"parentId":"0"},{"id":"5e24fd12f3384f38ab10898013fe25d7","isNewRecord":false,"createDate":"2016-11-29 15:19:35","updateDate":"2016-11-29 15:19:35","value":"2","label":"紧急事件","type":"case_grade","description":"案件级别","sort":2,"parentId":"0"},{"id":"f793a78eb44c405b9fd007359df09579","isNewRecord":false,"createDate":"2016-11-29 15:19:36","updateDate":"2016-11-29 15:19:36","value":"3","label":"一般重复","type":"case_grade","description":"案件级别","sort":3,"parentId":"0"},{"id":"3f2605e3f6e648f688b79da5f730aedb","isNewRecord":false,"createDate":"2016-11-29 15:19:34","updateDate":"2016-11-29 15:19:34","value":"4","label":"紧急重复","type":"case_grade","description":"案件级别","sort":4,"parentId":"0"}], 1, '', true);
            document.write(s);
        </script>
    </td>
    <td width="150px" class="success">受理时间:</td>
    <td colspan="1">2018-08-05 13:39:22</td>
</tr>
```

对上下文的源码进行分析，找到`getDictLabel`这个JavaScript的函数实现代码如下：

```
function getDictLabel(data, value, defaultValue){
	for (var i=0; i<data.length; i++){
		var row = data[i];
		if (row.value == value){
			return row.label;
		}
	}
	return defaultValue;
}
```

通过一番了解，可以获取到`td`标签中间的`script`对应的JS代码，通过执行JS代码获取对应的数据。找到了一个Python的库`pyexecjs`可以实现，可以通过如下代码进行安装：

```
pip install PyExecJS
```

核心代码如下：

```
import execjs

def exec_js_function(js):
    # 编译JS代码
    ctx = execjs.compile("""
        function getDictLabel(data, value, defaultValue){
            for (var i = 0; i < data.length; i++){
                var row = data[i];
                if (row.value == value){
                    return row.label;
                }
            }
            return defaultValue;
        }
    """)
    # 删除一些无关的字符
    jscode = js.replace('document.write(s);', '').replace(', true);', ')').replace('var s =', '')
    # 执行代码
    return ctx.eval(jscode)
```

更多用法可以从参考链接获取

### 参考资料

1、[官方主页](https://github.com/doloopwhile/PyExecJS)

2、[Python执行Js语句之ExecJs](https://www.jianshu.com/p/729be9639ac7)

3、[pyexecjs速度慢与pyv8的安装](http://www.wisedream.net/2017/11/27/traps/pyexecjs-and-pyv8/)
