---
layout: post
title: "在 iOS 模拟器里预览 H5：WKWebView 工程脚手架的实现"
date: 2026-09-15 12:00:00 +0800
comments: true
tags: iOS
---

做 H5 的同学经常要反复确认页面在 iPhone 上的真实渲染效果。打开模拟器、新建一个空工程、塞一个 `WKWebView`、写死 URL、编译、跑起来——这套动作每次都要几分钟，而且不同的本地服务（`localhost:3000`、`8080`、`5173`……）URL 还不一样。

我把它做成了一个 **Claude Code 技能**（Agent Skill），一句话就能预览：

```bash
/ios-webview-preview http://localhost:3000 --device "iPhone 16 Pro"
```

背后是一个完整的 iOS App 模板 + 一组 Shell 脚本：编译、安装、启动全部自动完成。这篇文章把它从「URL」到「模拟器上跑的 App」这条链路的实现细节完整拆开讲一遍。

<!--more-->

## 一、整体架构：一个「技能」如何打包一个 iOS App

这个技能本质上是三块东西：

```
skills/ios-webview-preview/
├── SKILL.md            # 技能入口，写给 coding agent 的说明
├── scripts/            # 编排脚本（初始化 / 编译 / 运行 / 测试）
│   ├── init.sh         # 拷模板 → xcodegen 生成工程
│   ├── build.sh        # xcodebuild 编译
│   ├── run.sh          # simctl install + launch
│   ├── update-url.sh   # 写死目标 URL 到 Config.swift
│   └── run-example.sh  # 一键跑完整流程
└── template/           # 完整的 iOS App 源码
    ├── project.yml     # xcodegen 工程描述
    └── WebViewPreview/
        ├── App/            # AppDelegate / SceneDelegate / Config
        ├── WebView/        # WKWebView 容器
        ├── UserScripts/    # 油猴式用户脚本系统
        ├── NativeExtensions/ # $app 原生扩展
        └── Resources/      # Monaco 编辑器 + 内置脚本
```

关键设计决策有两个：

**一是用 `xcodegen` 而不是手工维护 `.xcodeproj`。** `.xcodeproj` 是二进制 + 一堆 `pbxproj` 文本，改动频繁时 merge 和手工维护都很痛苦。这里只维护一份 `project.yml`，每次 `xcodegen generate`（不到 1 秒）就重新生成工程。`run-example.sh` 里干脆每次都重新 generate，加新 Swift 文件、改 `Resources` 目录都不用再管工程文件。

**二是「模板 + 工作副本」分离。** `template/` 是单一数据源，运行时会完整拷贝到 `~/.ios-webview-preview/` 再编译。技能升级时重新拷贝即可，不会污染源码仓库。

## 二、从 URL 到模拟器：构建流水线

用户给一个 URL，到 App 真正跑起来，串起来的是四个脚本：

**1. `init.sh` —— 一次性初始化。** 把 `template/*` 拷到 `~/.ios-webview-preview/`，然后 `xcodegen generate`。里面有个细节：用 `${BASH_SOURCE[0]}` 解析脚本自身路径，而不是 `$0`，这样无论技能被装在插件市场、手动 clone 还是 symlink，`template/` 都能被正确找到。

**2. `update-url.sh` —— 写死目标 URL。** 这是最有意思的一步。`Config.swift` 里只有一个枚举：

```swift
enum AppConfig {
    static var targetURL = "http://localhost:3000"
}
```

脚本不是用 `sed` 替换，而是直接 `cat >` 整个文件重写：

```bash
cat > "$CONFIG_FILE" << EOF
import Foundation

enum AppConfig {
    static let targetURL = "$NEW_URL"
}
EOF
```

原因是 URL 里可能包含 `&`、`?`、`/` 这些对 `sed` 替换有特殊含义的字符。直接 heredoc 重写整个文件，天然规避了转义地狱。注意这里有个**易踩的坑**：模板里是 `static var`，`update-url.sh` 生成的是 `static let`——因为用户还能在 App 内长按导航栏标题现场改 URL（后面讲），运行时改需要 `var`，而脚本写入的编译期常量用 `let` 即可。

**3. `build.sh` —— 解析设备名并编译。** 用户传的是 `"iPhone 16 Pro"` 这种名字，但 `xcodebuild` 需要 UDID。脚本先用 `simctl list devices` 把名字反解成 UDID：

```bash
DEVICE_UDID=$(xcrun simctl list devices | grep "$DEVICE_NAME" | grep -v "unavailable" | head -1 | grep -oE '[A-Fa-f0-9-]{36}')
```

如果直接传的就是 UDID（36 位十六进制格式），就原样用。最后 `xcodebuild -destination "id=$DEVICE_UDID" build`，并把 UDID 落到 `/tmp/webview-preview-device-udid.txt` 供下一步复用。

**4. `run.sh` —— 安装并启动。** `simctl boot`（已 boot 就跳过）→ `simctl install` → `simctl launch`。`open -a Simulator` 把模拟器窗口带到前台，`sleep 2` 等它就绪。

这四个脚本在 `run-example.sh` 里被收敛成一条命令，用 `set -euo pipefail` 保证任何一步失败立即中断。开发技能时改完 Swift 或示例脚本，跑这一条就够了。

## 三、WKWebView 容器

`WebViewController` 是核心容器。除了常规的 `WKWebView` 配置，有几个值得展开的点：

### 3.1 用 KVO 而不是 raw KVO

导航栏标题要跟随网页的 `<title>`，返回按钮要跟随 `canGoBack`。最直接的做法是 `addObserver(_:forKeyPath:)`。但这里用 Swift 4 的 `NSKeyValueObservation`：

```swift
canGoBackObservation = webView.observe(\.canGoBack, options: [.new]) { [weak self] _, _ in
    self?.updateBackButtonState()
}
titleObservation = webView.observe(\.title, options: [.new]) { [weak self] _, _ in
    self?.updateTitle()
}
```

注释里点明了原因：**raw KVO 的 `observeValue(forKeyPath:...)` 需要手动判断 keyPath 并转发 `super`**，如果 `WKWebView` 内部触发了一个我们没处理的 keyPath，转发到 super 时 context 为 nil 会直接崩溃。`NSKeyValueObservation` 是每个 keyPath 一个独立的、类型安全的观察者，天然没有这个问题，而且 `deinit` 里 `invalidate()` 清理引用关系更清晰。

### 3.2 长按导航栏现场改 URL

预览本地服务时经常要换端口。这里给导航栏挂了一个 `UILongPressGestureRecognizer`（0.5 秒），长按弹 `UIAlertController`，输入新 URL 后校验 scheme 必须是 `http` / `https`，然后更新 `targetURL` 并 reload。这个小功能让「改端口」从「重新跑脚本」变成「长按 + 改文字」。

### 3.3 非 Web scheme 交给系统

`mailto:`、`tel:`、`itms-apps:` 这类 URL 不能让 `WKWebView` 吞掉。在 `decidePolicyFor` 里判断：

```swift
let webSchemes: Set<String> = ["http", "https", "about", "data"]
if !webSchemes.contains(scheme) {
    UIApplication.shared.open(url)
    decisionHandler(.cancel)
    return
}
```

另外 `createWebViewWith` 里处理了 `window.open`：当 `targetFrame == nil`（新窗口）时，直接在当前 `webView` 里 `load` 那个 request，而不是真的弹新窗口。

## 四、油猴式用户脚本系统

App 内置了一套类似 Tampermonkey 的用户脚本管理：可以新建、编辑、启用/禁用、删除脚本，脚本在页面加载时自动注入。

### 4.1 数据模型与持久化

`Script` 结构体包含 `id / name / version / source / enabled / createdAt / updatedAt`。`name` 和 `version` 是**派生字段**——每次保存时从源码头部 `// ==UserScript==` 块里解析出来覆盖，而不是独立输入框。

持久化到 `Documents/scripts/`：

```
Documents/scripts/
  index.json     # [Script] 的 JSON 数组
  <id>.js        # 每个脚本源码单独落一个文件
```

写文件采用「先写 `.tmp` 再 `rename`」的原子写策略，避免写一半崩溃留下损坏文件。`index.json` 和 `<id>.js` 是冗余的：`.js` 方便人直接打开看和外部修改，`index.json` 是程序读取的唯一索引。

一个容易被忽略的坑在 **Date 解码**上。`JSONDecoder` 默认 `.deferredToDate` 只认 epoch `Double`，如果有人写成了 ISO 8601 字符串，整个 `index.json` decode 失败，catch 后静默返回 `[]`——用户脚本全部「失踪」。所以这里自定义了 `dateDecodingStrategy`，两种格式都吃：

```swift
decoder.dateDecodingStrategy = .custom { dec in
    let c = try dec.singleValueContainer()
    if let s = try? c.decode(String.self), let d = ISO8601DateFormatter().date(from: s) {
        return d
    }
    let n = try c.decode(Double.self)
    return Date(timeIntervalSince1970: n)
}
```

### 4.2 注入时的错误隔离

用户脚本的源码是不可信输入（用户自己写，或从 URL 拉取）。如果直接塞进 `WKWebView`，脚本里一个 `throw` 会让页面后续的 JS 整体挂掉。所以每个脚本都套一层 IIFE + try/catch：

```swift
static func wrappedSource(_ userCode: String) -> String {
    return "(function(){\ntry{\n\(userCode)\n}catch(e){console.error('[userscript error]',e)}})();"
}
```

错误统一打到 console（Xcode console 可见），不影响宿主页面。

### 4.3 内置脚本：vConsole

除了用户自己加的脚本，App 还内置了 [vConsole](https://github.com/Tencent/vConsole)（H5 调试面板）。内置脚本和用户脚本共用同一套 `Script` 模型，但有几个区别：

- **source 从 bundle 读**（`Resources/Scripts/vconsole.user.js`），不写沙箱，用户删不掉；
- **启用状态存 `UserDefaults`**（key `builtin-<id>-enabled`），和用户脚本的 `enabled` 字段独立；
- **默认关闭**，需要在列表里手动打开；
- 编辑器打开内置脚本时设 `readOnly`，右上角是「关闭」而不是「保存」。

`BuiltinScripts.isBuiltin(id)` 用 `builtin-` 前缀判定，`SettingsViewController` 据此禁掉左滑删除和编辑保存。

## 五、原生扩展：`$app` 命名空间与跨域 Cookie 桥

这是整个项目里最有技术含量的一块——**让 H5 通过 JS 调用 iOS 原生能力**，并解决了一个前端原生做不到的痛点：跨域 cookie 读写。

### 5.1 插件协议与注册表

核心是一个 `@MainActor` 协议：

```swift
@MainActor
protocol AppNativePlugin {
    var name: String { get }
    func jsAPI() -> String                    // 返回要挂到 $app.<name> 上的 JS 字面量
    func handle(action: String, payload: [String: Any],
                webView: WKWebView, reply: @escaping (AppNativeReply) -> Void)
}
```

每个插件（目前是 `CookieBridgePlugin`）在 `AppNativeRegistry.plugins` 数组里登记一行。注册表负责三件事：

1. **shim 注入**：把每个插件的 `jsAPI()` 拼成 `window.$app.<name> = { ... }`，注入到 `.atDocumentStart`（早于用户脚本的 `.atDocumentEnd`）；
2. **dispatch**：收到 native 消息时按 `module` 字段路由到对应插件的 `handle`；
3. **reply**：把 `(ok, value, error)` 序列化成 JSON，调 `window.__appNativeReply(id, ...)` 回传。

**新增插件只需两步**：写一个 `struct XxxPlugin: AppNativePlugin`，然后在 `plugins` 数组加一行。`WebViewController` 完全不用改——shim 自动重拼，handler 自动 dispatch。

### 5.2 把异步消息包装成 Promise

`WKScriptMessageHandler` 的 `postMessage` 天生是「fire and forget」的，没有返回值。但插件调用者想要 `await $app.cookie.get(...)` 这样的同步风格。shim 里做了关键的一层转换——**给 `postMessage` 包一层 Promise**：

```javascript
native.postMessage = function(envelope) {
  return new Promise(function(resolve, reject) {
    var id = nextId++;
    var timer = setTimeout(function() {
      pending.delete(id);
      reject(new Error('timeout'));
    }, TIMEOUT_MS);                       // 30s 超时
    pending.set(id, { resolve: resolve, reject: reject, timer: timer });
    nativePost({ id: id, module: envelope.module, action: envelope.action, payload: envelope.payload || {} });
  });
};
```

每个请求分配一个递增的 `id`，存进 `pending` Map 并启动 30 秒超时定时器；native 处理完后调 `window.__appNativeReply(id, result)`，shim 找到对应的 Promise 决定 resolve 还是 reject。原生侧 30 秒不回就 reject，避免 Promise 永远 pending。

### 5.3 跨域 Cookie 桥

前端 `document.cookie` 受同源策略限制，只能读写当前域的 cookie。而 `WKHTTPCookieStore`（iOS 11+）是原生 API，对域不做 superdomain 校验。`CookieBridgePlugin` 把它桥成 `$app.cookie.set/get/list/delete` 四个方法。

比如 H5 想给 `example.com` 写一个带 `sameSite` 的 cookie，`document.cookie` 在 `localhost` 页面上做不到，而：

```javascript
await $app.cookie.set({
  name: '__cb_xd', value: 'xd_val',
  domain: 'example.com', path: '/',
  expires: 1735689600, secure: true, httpOnly: true, sameSite: 'lax'
});
```

跨域读写、`HttpOnly`、`SameSite`、后缀匹配（`.example.com` 命中 `api.example.com`）全都支持。`CookieBridgePlugin` 里对 `sameSite` 做了字符串到 `HTTPCookieStringPolicy` 的映射，对域做了精确 + 后缀两段匹配。

值得注意的是 `@MainActor` 在这里的价值：`WKHTTPCookieStore` 是 `WK_SWIFT_UI_ACTOR`，所有方法都在主线程。协议标了 `@MainActor` 后，插件里直接同步访问这些 API 即可，不用写一堆 `DispatchQueue.main.async` 嵌套。

## 六、Monaco 编辑器：file:// 下踩过的坑

用户脚本编辑器用 [Monaco Editor](https://microsoft.github.io/monaco-editor/)（VS Code 同款内核），跑在一个 `WKWebView` 里。这是整个项目调试成本最高的部分，几个坑都值得记下来。

### 6.1 loadFileURL 而不是 loadHTMLString

早期实现用 `loadHTMLString + file:// baseURL` 加载 Monaco。结果 `WKWebView` 在 `loadHTMLString` 下会让页面拿到 **opaque origin**，AMD loader 抛错时只能拿到 `Script error. :0`——跨域细节被浏览器吞掉，根本没法定位。

改成 `loadFileURL + allowingReadAccessTo:` 后，页面拿到真正的 `file://` origin，相对路径下的 `loader.js` 和 AMD 模块全部同源加载，错误信息完整可见。

### 6.2 没有 Web Worker：getWorker 必须返回 Promise

Monaco 的 TS/JSON/HTML 智能提示依赖 Web Worker，但 `file://` 下 `WKWebView` 无法创建 Worker。所以这里裁掉了所有 worker 相关的语言服务，`getWorker` 返回一个 noop worker。

**坑在于：必须返回 Promise，而不是同步 `null`。** Monaco 的 `WorkerManager` 拿到返回值后直接 `.then(...)`（不是 `await`），同步返回 `null` 会被解析成：

```
null is not an object (evaluating 'i.then')
```

把整个 worker 加载链路炸掉，编辑器事件循环被破坏、无法输入。正确的写法：

```javascript
self.MonacoEnvironment = {
  getWorker: function () {
    return new Promise(function (resolve) {
      resolve({ postMessage: function () {}, terminate: function () {}, /* ... */ });
    });
  }
};
```

### 6.3 手动注册内联 tokenizer

JS 语法高亮没有走 `require('vs/basic-languages/javascript')`，而是在 `editor.html` 里内联了一个 Monarch tokenizer。原因是那个 AMD 模块的 `define` 写成了 `["require","require"]`，loader 会报错导致回调永远不触发。内联是同步的，Monarch 也在主线程同步跑，对油猴脚本这种 JS-only 场景完全够用。

### 6.4 尺寸与 reflow

`WKWebView` 里 Monaco 首次 layout 计算高度异常，早期实现把宽高缓存到闭包里，拿到的是 0，后续 reflow 一直传 0，Monaco 只渲染 1 行高。修法是：始终读 container 的**当前**尺寸 + `automaticLayout: true` + 创建后多次 `setTimeout(reflow, [0, 50, 200, 500, 1000])` 兜底，把时序问题全部覆盖。

### 6.5 裁剪打包

`monaco-bundler.sh` 把 `monaco-editor` npm 包裁剪后写入模板。裁掉了带 worker 的语言服务（typescript/json/css/html）、非英文 NLS 包（约 1.7MB）、以及除 javascript 外的所有 basic-languages（约 636KB → 8KB）。只保留 JS 语法高亮所需的最小集合。

还有个细节：`NoAccessoryWebView` 重写了 `inputAccessoryView` 返回空 UIView，屏蔽键盘顶部那排系统附件栏（地球键、听写键）。

## 七、测试自动化

因为跑在模拟器上，测试分为「能自动化的」和「必须手动的」两部分。

`test-flow.sh` 自动完成：同步模板 → 卸载清沙箱 → 编译 → 安装 → 注入示例脚本 → 截图 → **抓运行日志做断言**。其中日志断言最有价值——它把历史上踩过的四个具体 bug 都固化成反例：

```bash
assert_no "uncaught.*Script error"            "无跨域脚本错误"
assert_no "null is not an object.*i\\.then"    "worker Promise 正确"
assert_no "Could not create a sandbox extension" "无 sandbox 扩展失败"
assert_no "unhandledrejection.*P\\.comments"   "无 tokenization 失败"
```

每修一个坑，就加一条断言，保证它不会回归。

注入示例脚本的 `install-test-scripts.sh` 里有个巧妙点：**用 MD5 文件名生成 stable UUID**，保证多次运行幂等——脚本 id 稳定，`index.json` 合并时能正确识别「已存在」而不是重复追加。

触控和键盘输入没法用 `simctl` 模拟，这部分落在 `scripts/TEST.md` 的 14 步人工验收清单里（点齿轮、点 +、打字、保存、左滑删除、开关切换……）。

## 小结

这个技能的价值不在于单个技术点有多难，而在于把一整套 iOS 工程实践打包成了一个可复用的自动化工具。几个值得复用的思路：

1. **用 `xcodegen` 让工程文件从「手工维护」变成「可再生的产物」**，模板 + 工作副本分离；
2. **用 heredoc 重写代替 sed 替换**，天然规避转义问题；
3. **KVO 用 `NSKeyValueObservation` 替代 raw KVO**，从根源上避免 context 崩溃；
4. **Promise 化的 `postMessage` + 超时**，把「fire-and-forget」的 bridge 变成可 `await` 的 API；
5. **`loadFileURL` + noop worker + 内联 tokenizer**，是 Monaco 跑进 `file://` 环境的一套完整解法；
6. **把每个踩过的坑固化成日志断言**，让测试脚本成为活的「防回归清单」。

完整代码在 [ai-skills/skills/ios-webview-preview](https://github.com/zengjing/ai-skills/tree/master/skills/ios-webview-preview)，感兴趣可以直接拿来用或参考。
