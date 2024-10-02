---
layout: post
title: "macOS Sequoia剪切板获取文件路径的问题"
date: 2024-09-29 22:00:00 +0800
comments: true
tags: Note
---

![macos_sequoia](/images/macos-sequoia-pastboard-file-url/macos_sequoia.jpg)

收到一个问题，说升级到 macOS Sequoia，客户端复制文件粘贴到程序中通过代码获取的路径是 `file:///.file/id=xxxx`，而不是实际文件路径。之前的版本是正常的。

从剪切板获取文件路径的代码：

```
void get_file_names(void) {
    NSPasteboard* pasteboard = [NSPasteboard generalPasteboard];
    NSArray* tempArray = [pasteboard pasteboardItems];
    for(NSPasteboardItem *tmpItem in tempArray){
        NSString *pathString = [tmpItem stringForType:@"public.file-url"];
        NSLog(@"pathString: %@", pathString);
    }
}
```

通过 `public.file-url` 方式获取的地址不再是文件的真实路径，而是一个 `fileid` 形式的地址，其实兼容的代码很简单就是使用 `[NSURL path]` 重新获取一下地址就OK了。

修改后代码：

```
void get_file_names(void) {
    NSPasteboard* pasteboard = [NSPasteboard generalPasteboard];
    NSArray* tempArray = [pasteboard pasteboardItems];
    for(NSPasteboardItem *tmpItem in tempArray){
        NSString *pathString = [tmpItem stringForType:@"public.file-url"];
        if (pathString && [pathString isKindOfClass:[NSString class]]) {
            NSURL *url = [NSURL URLWithString:pathString];
            NSString *realPathString = [url path];
            NSLog(@"pathString: %@", realPathString);
        }
    }
}
```

其实还是代码写的不是很规范导致的，在 `Drag and Drop` 等场景下面早就是 `fileid` 形式的地址了。

### 参考资料

- 1、[How to get coppied file's path](https://www.alfredforum.com/topic/14680-how-to-get-coppied-files-path/)

- 2、[macOS 剪切板数据查看与修改](https://conradsun.github.io/2023/08647ac6c0.html)

- 3、[NSURL returns file's id instead of file's path](https://stackoverflow.com/questions/31320947/nsurl-returns-files-id-instead-of-files-path)

- 4、[What Do You Get When You Drag and Drop a PNG File From Finder Into an NSTextView?](https://christiantietze.de/posts/2021/11/nstextview-image-file-pasteboard/)

- 5、[NSURL File ID](https://www.macscripter.net/t/nsurl-file-id/71792)