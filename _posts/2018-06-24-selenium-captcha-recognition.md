---
layout: post
title: "使用Selenium获取验证码并识别"
date: 2018-06-24 11:46:29 +0800
comments: true
categories: Note
---

最近项目组提了个需求要求我这边帮他们实现一个网站的数据采集并对接到指定的数据库表里面，记录下使用的在线API识别验证码的过程：

由于验证码在每次加载页面的时候都会刷新，也就是说每次打开登录界面都是不同的验证码，所以需要将打开的登录界面截图然后从里面扣取验证码对应的内容再提交到服务器进行识别。

1、对登录界面进行截图

```
url = ''
driver = webdriver.PhantomJS()
driver.get(url)
driver.set_window_size(1200, 800) #此处一定要设置固定值，在其他的机器上面运行的时候可能会有问题
# 暂停10s确保登录界面加载完成
time.sleep(10)
# 截取登录界面的屏幕
screenshot_path = 'screenshot.png'
if os.path.exists(screenshot_path):
	os.remove(screenshot_path)
driver.save_screenshot(screenshot_path)
```

2、从截图中扣取验证码

```
# 找到验证码元素，并获取到位置坐标
element = driver.find_element_by_xpath("//*[@id=\"login-content\"]/div[2]/div[4]/img")
left = int(element.location['x'])
top = int(element.location['y'])
right = int(element.location['x'] + element.size['width'])
bottom = int(element.location['y'] + element.size['height'])
# 从截图中抠出来验证码的区域
captcha_path = 'captcha.png'
if os.path.exists(captcha_path):
	os.remove(captcha_path)
img = Image.open(screenshot_path)
img = img.crop((left, top, right, bottom))
img.save(captcha_path)
```

3、调用在线API进行验证码识别

以下代码来源于：聚合数据-验证码识别的示例代码，具体的可以参考官方的文档：

```
def captcha_recognition(appkey, codeType, imagePath):
	"""
	调用验证码在线识别的API
   :param appkey: 平台申请的appkey
   :param codeType: 验证码类型
   :param imagePath: 验证码图片路径
   :return: 查询结果
    """
    if not os.path.exists(imagePath):
        return ''
    captcha_result = ''
    submitUrl = 'http://op.juhe.cn/vercode/index'  # 接口地址
    # buld post body data
    boundary = '----------%s' % hex(int(time.time() * 1000))
    data = []
    data.append('--%s' % boundary)
    data.append('Content-Disposition: form-data; name="%s"\r\n' % 'key')
    data.append(appkey)
    data.append('--%s' % boundary)
    data.append('Content-Disposition: form-data; name="%s"\r\n' % 'codeType')
    data.append(codeType)
    data.append('--%s' % boundary)
    fr = open(imagePath, 'rb')
    data.append('Content-Disposition: form-data; name="%s"; filename="b.png"' % 'image')
    data.append('Content-Type: %s\r\n' % 'image/png')
    data.append(fr.read())
    fr.close()
    data.append('--%s--\r\n' % boundary)
    http_body = '\r\n'.join(data)
    try:
        req = urllib2.Request(submitUrl, data=http_body)
        req.add_header('Content-Type', 'multipart/form-data; boundary=%s' % boundary)
        req.add_header('User-Agent', 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.143 Safari/537.36')
        req.add_header('Referer', 'http://op.juhe.cn/')
        resp = urllib2.urlopen(req, timeout=60)
        qrcont = resp.read()
        result = json.loads(qrcont, 'utf-8')
        error_code = result['error_code']
        if (error_code == 0):
            data = result['result']
            captcha_result = data
        else:
            errorinfo = u"错误码:%s,描述:%s" % (result['error_code'], result['reason'])
            print errorinfo
    except Exception as e:
        print e
    return captcha_result
```

### 参考资料

1、[聚合数据-验证码识别](https://www.juhe.cn/docs/api/id/60)
