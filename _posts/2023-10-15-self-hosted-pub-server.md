---
layout: post
title: "搭建Dart Pub镜像服务"
date: 2023-10-15 17:00:00 +0800
comments: true
tags: Note
---

和其他的语言一样 Dart 也有自己的包管理工具，[Pub](https://pub.dev/) 是 Dart 官方的包管理器。

![pub_dev](/images/pub_host/pub_dev.png)

pub 默认从 `https://pub.dartlang.org/` 下载依赖包，如果需要修改可以通过设置环境变量 `PUB_HOSTED_URL`。如修改为清华的镜像源可以使用如下命令进行修改：

```
export PUB_HOSTED_URL="https://mirrors.tuna.tsinghua.edu.cn/dart-pub"
```

`PUB_HOSTED_URL` 设置的地址其实是需要能访问互联网的，那么对于在内网的构建机器来说就需要给每台机器设置需要访问网络，或者是单独给这个地址开启代理服务。

现在的诉求就是希望能搭建一个pub服务的镜像，部署的机器能访问互联网，其他的打包机通过设置环境变量指向这个服务。

发现 [pub-mirror](https://github.com/tuna/pub-mirror) 这个开源库可以满足我的要求，这个就是一个镜像服务会自动将[https://pub.dartlang.org/](https://pub.dartlang.org/) 网站上面的内容全量同步到本地。但是存在一个问题就是会下载太多的包到本地，本地的磁盘有点扛不住。

那就只能自己去实现了，找到一个私有托管 dart packages 的开源项目 [unpub](https://github.com/bytedance/unpub)，可以实现将项目内部用到的私有库发布然后供其他项目进行使用。

![unpub_screenshot](/images/pub_host/unpub_screenshot.png)

其实 PUB 托管服务主要是提供如下几个api（以 dio 为例）：

```
https://pub.dartlang.org/api/packages # 获取所有的package，JSON字符串
https://pub.dartlang.org/api/packages/dio # 获取指定名称的package版本信息，JSON字符串
https://pub.dartlang.org/api/packages/dio/versions/5.3.3 # 获取指定名称的package的指定版本信息，JSON字符串
https://pub.dartlang.org/packages/dio/versions/5.3.3.tar.gz # 安装包（路径里面没有api）
```

查看 unpub 的源码发现其内部实现了上面的几个api，主要实现流程如下：

```
// 1、先查找是否是私有库
var package = await metaStore.queryPackage(name);

if (package == null) {
    // 2、如果不是私有库，直接重定向到上游的服务
    return shelf.Response.found(Uri.parse(upstream).resolve('/api/packages/$name').toString());
}

// 3、使用私有库
....
```

那么只要处理 package 找不到的时候，调用上游的地址先请求然后把结果返回即可（替换成自己的服务地址）：

(1) 修改 `@Route.get('/api/packages/<name>')`

```
if (package == null) {
    Uri upstreamUri = Uri.parse(Uri.parse(upstream).resolve('/api/packages/$name').toString());
    final response = await http.get(upstreamUri);
    if (response.statusCode != 200) {
        return shelf.Response.found(upstreamUri);
    }
    String responseText = response.body;
    responseText = responseText.replaceAll(upstream, host);
    return _okWithJsonString(responseText);
}
```

(2) 修改 `@Route.get('/api/packages/<name>/versions/<version>')`

```
if (package == null) {
    Uri upstreamUri = Uri.parse(Uri.parse(upstream).resolve('/api/packages/$name/versions/$version').toString());
    final response = await http.get(upstreamUri);
    if (response.statusCode != 200) {
        return shelf.Response.found(upstreamUri);
    }
    String responseText = response.body;
    responseText = responseText.replaceAll(upstream, host);
    return _okWithJsonString(responseText);
}
```

(3) 修改 `@Route.get('/packages/<name>/versions/<version>.tar.gz')`

```
if (package == null) {
    // 如果本地没有缓存先下载
    var file = File(path.join(baseDir, name, '$name-$version.tar.gz'));
    var file_exists = await file.exists();
    if (!file_exists) {
        Uri upstreamUri = Uri.parse(Uri.parse(upstream).resolve('/packages/$name/versions/$version.tar.gz').toString());
        http.Client client = new http.Client();
        var req = await client.get(upstreamUri);
        var content = req.bodyBytes;
        await file.create(recursive: true);
        await file.writeAsBytes(content);
    }
    return shelf.Response.ok(
        file.openRead(),
        headers: {HttpHeaders.contentTypeHeader: ContentType.binary.mimeType},
    );
}
```

其中 `_okWithJsonString` 是新增的一个私有方法，主要是返回JSON字符串：

```
static shelf.Response _okWithJsonString(String data) => shelf.Response.ok(
    data,
    headers: {
        HttpHeaders.contentTypeHeader: ContentType.json.mimeType,
        'Access-Control-Allow-Origin': '*'
    },
);
```

如果要调试代码，需要有 mongo 环境，使用 docker 快速安装一个：

```
docker run -d --name mongo -p 27017:27017 mongo
```

代码修改完重新编译运行，找一个 Flutter 项目在执行：

```
export PUB_HOSTED_URL="http://127.0.0.1:4000"
dart pub get --verbose
```

如果控制台出现如下内容表示部署成功：

![pub_log](/images/pub_host/pub_log.png)

### 支持Docker部署

Dockerfile 配置：

```
FROM google/dart

WORKDIR /app
COPY ./unpub ./
RUN dart pub get
ENTRYPOINT ["dart", "run", "bin/unpub.dart", "--database", "mongodb://mongo:27017/dart_pub"]
```

docker-compose.yml 配置：

```
version: '3.3'
services:
  unpub:
    build: ./
    container_name: unpub_server
    environment:
     - UNPUB_UPSTREAM=https://pub.flutter-io.cn
     - UNPUB_HOST=http://pub.devzeng.com
    restart: always
    ports:
      - 4000:4000
    volumes:
      - ~/docker/unpub/unpub_packages:/app/unpub-packages
      - ~/docker/unpub/mirror_packages:/app/mirror-packages
    depends_on:
      - mongo
  mongo:
    image: mongo:4.2.19
    container_name: unpub_mongo 
    restart: always
    volumes:
      - ~/docker/unpub/unpub_mongo:/data/db
```

运行：

```
docker-compose up --build -d
```

推荐使用 nginx 做一个域名的代理，示例配置如下：

```
http {
  server {
    listen 80;
    server_name pub.devzeng.com;

    location / { 
        proxy_pass   http://127.0.0.1:4000;
        index  index.html index.htm;  
    }
  }
}
```

> 附：

- (1) 代码在 https://github.com/hhtczengjing/unpub 的 develop 分支
- (2) 关于私有库的托管部分后续完善

### 参考资料

- 1、[Configuring pub environment variables](https://dart.dev/tools/pub/environment-variables)