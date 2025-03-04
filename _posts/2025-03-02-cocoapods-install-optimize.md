---
layout: post
title: "CocoaPods install自定义优化"
date: 2025-03-02 12:00:00 +0800
comments: true
tags: Note
---

CocoaPods 是一个用于管理 iOS 项目中第三方开源库的工具。它能够让 iOS 开发人员通过一些简单的指令就可以下载第三方开源库，并可以管理所下载的第三方框架。这些年一直在使用 CocoaPods，但是也遇到了一些问题，这里记录下一些问题及解决方案。

![cocoapods](/images/cocoapods-install-optimize/cocoapods.png)

> 关于 CocoaPods 源码调试

可以参考 [VSCode搭建CocoaPods调试环境](https://blog.devzeng.com/blog/vscode-debug-cocoapods.html)，方便快捷定位到实现源码的位置并进行自定义的验证。

### CocoaPods 自定义缓存目录

CocoaPods 中可以在 Podfile 中配置从指定的仓库指定的分支下载对应的 Pod 库。如：

```
pod 'Xxx', :git => 'https://xxx.com/xxx/Xxx.git', :branch => 'feature/dev-v1.0'
```

下载的 Pod 库会存放在 `~/Library/Caches/CocoaPods/Pods` 目录下缓存起来 (对应放在 External 下面），目录结构如下：

```
Pods
├── External
│   └── Xxx
│       └── 7a2ddd624b9f4e3ad7df801a11bad5a0
│           ├── README.md
│           └── Xxx
├── Release
│   ├── AFNetworking
│   │   └── 4.0.1-7864c
│   │       ├── AFNetworking
│   │       ├── LICENSE
│   │       ├── README.md
│   │       └── UIKit+AFNetworking
├── Specs
│   ├── External
│   │   └── Xxx
│   │       └── 7a2ddd624b9f4e3ad7df801a11bad5a0.podspec.json
│   └── Release
│       ├── AFNetworking
│       │   └── 4.0.1-7864c.podspec.json
└── VERSION
```

因为有缓存，在某些场景下，同时打包相关项目，但是分别指定不同的分支，可能出现代码对应不上的问题。如果使用 Jenkins 打包，可以设置节点同时只能运行一个任务，在执行之前可以先清理掉 External 目录下的内容。

```
rm -rf ~/Library/Caches/CocoaPods/Pods/External/*
rm -rf ~/Library/Caches/CocoaPods/Pods/Specs/External/*
```

但是这样打包效率太低了，着急的时候就出现任务等待的情况。其实只需要想办法让不同的项目能够自定义的指定不同的缓存目录，这样就可以避免同时打包的问题。其实 CocoaPods 已经提供了 `CP_CACHE_DIR` 环境变量，可以设置不同的缓存目录。但是这个设置的是整个CocoaPods 的缓存目录，如果仅需要将 External 针对不同的项目设置不同的缓存目录就需要修改 CocoaPods 源码了。

通过断点调试，改动的文件为 `CocoaPods/lib/cocoapods/downloader/cache.rb`, 仓库地址: https://github.com/CocoaPods/CocoaPods.git 。设计一个环境变量 `CP_EXT_CACHE_DIR`，通过设置当前项目扩展缓存路径可以实现不同项目自定义缓存目录。

改动代码如下：

```
# @return [Pathname] The ext root directory where this cache store its
#         downloads.
#
attr_reader :ext_root

# Initialize a new instance
#
# @param  [Pathname,String] root
#         see {#root}
#
def initialize(root)
    @root = Pathname(root)
    unless ENV['CP_EXT_CACHE_DIR'].nil?
        ext_path = Pathname.new(ENV['CP_EXT_CACHE_DIR']).expand_path
        ext_path.mkpath unless ext_path.exist?
        @ext_root = ext_path
    else
        @ext_root = Pathname(root)
    end
    ensure_matching_version
end

# @param  [Request] request
#         the request to be downloaded.
#
# @param  [Hash<Symbol,String>] slug_opts
#         the download options that should be used in constructing the
#         cache slug for this request.
#
# @return [Pathname] The path for the Pod downloaded from the given
#         `request`.
#
def path_for_pod(request, slug_opts = {})
    cache_dir = request.released_pod? ? root : ext_root
    pod_path = cache_dir + request.slug(**slug_opts)
    pod_path
end

# @param  [Request] request
#         the request to be downloaded.
#
# @param  [Hash<Symbol,String>] slug_opts
#         the download options that should be used in constructing the
#         cache slug for this request.
#
# @return [Pathname] The path for the podspec downloaded from the given
#         `request`.
#
def path_for_spec(request, slug_opts = {})
    cache_dir = request.released_pod? ? root : ext_root
    spec_path = cache_dir + 'Specs' + request.slug(**slug_opts)
    spec_path = spec_path.sub_ext('.podspec.json')
    spec_path
end
```

### CocoaPods 优化 Git 下载超时的问题

随着版本迭代，或者是提交了比较大的文件， GIT 仓库会变得越来越大，在下载的时候，会出现如下错误：

```
Cloning into '/var/folders/q4/0m_l9t6j3s38pjwfyg8v2s2mwh2j_d/T/d20250303-79709-yhqu0o'...
error: RPC failed; HTTP 504 curl 22 The requested URL returned error: 504
fatal: expected 'packfile'
```

以往类似场景一般调整一下 GIT 参数基本上能解决：

```
git config --global http.postBuffer 10000000000
git config --global http.maxRequestBuffer 500M
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
```

但其实并没能解决根本问题，从网上搜索到相关资料，可以通过在 clone 的时候指定 `--depth 1` 来解决，但是 CocoaPods 并没有提供类似参数。可以参考 [cocoapods install加速](https://action121.github.io/2021/03/25/cocoapods-install%E5%8A%A0%E9%80%9F.html)，通过修改 CocoaPods 源码的方式解决了超时的问题。

修改 `cocoapods-downloader/lib/cocoapods-downloader/git.rb`， 仓库地址：https://github.com/CocoaPods/cocoapods-downloader.git 。改动代码如下：

```
def self.preprocess_options(options)
    return options unless options[:branch]

    input = [options[:git], options[:commit]].map(&:to_s)
    invalid = input.compact.any? { |value| value.start_with?('--') || value.include?(' --') }
    raise DownloaderError, "Provided unsafe input for git #{options}." if invalid

    command = ['ls-remote',
                '--',
                options[:git],
                options[:branch]]

    output = Git.execute_command('git', command)
    match = commit_from_ls_remote output, options[:branch]

    return options if match.nil?

    options[:commit] = match
    # options.delete(:branch)

    options
end

# The arguments to pass to `git` to clone the repo.
#
# @param  [Bool] force_head
#         If any specific option should be ignored and the HEAD of the
#         repo should be cloned.
#
# @param  [Bool] shallow_clone
#         Whether a shallow clone of the repo should be attempted, if
#         possible given the specified {#options}.
#
# @return [Array<String>] arguments to pass to `git` to clone the repo.
#
def clone_arguments(force_head, shallow_clone)
    command = ['clone', url, target_path]

    if branch = options[:branch]
        command += ['--branch', branch, '--depth', 1]
    else
        command += ['--template=']

        if shallow_clone && !options[:commit]
            command += %w(--single-branch --depth 1)
        end
  
        unless force_head
            if tag_or_branch = options[:tag] || options[:branch]
              command += ['--branch', tag_or_branch]
            end
        end
    end

    command
end
```

> 说明

- (1) 下载慢的问题，主要是因为 GIT 仓库太大导致的。如果仓库里面的大文件比较多，可以考虑将大文件移除仓库，或者将大文件放到其他地方。

- (2) 如果有条件可以将仓库转换为zip的方式使用http下载，或者是产物化之后下载。

### 参考资料

- 1、[cocoapods install加速](https://action121.github.io/2021/03/25/cocoapods-install%E5%8A%A0%E9%80%9F.html)

- 2、[iOS Git仓库及资源下载加速调研](https://action121.github.io/2022/11/23/iOS-Git%E4%BB%93%E5%BA%93%E5%8F%8A%E8%B5%84%E6%BA%90%E4%B8%8B%E8%BD%BD%E5%8A%A0%E9%80%9F%E8%B0%83%E7%A0%94.html)

- 3、[介绍git clone --depth=1的用法](https://www.cnblogs.com/cangqinglang/p/14007741.html)

- 4、[CocoaPods缓存清理之谜](https://mp.weixin.qq.com/s/dfyJxfah2VY5bQyoQQ8L3g)