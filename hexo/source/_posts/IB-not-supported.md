---
title: Compiling IB documents for earlier than iOS 7 is no longer supported
abbrlink: effc9d9
date: 2021-09-02 15:45:44
tags:
  - iOS
---

最近在苹果官方网站下载了一个 Demo，由于年代久远，Xcode 打开编译后，直接提示如下错误信息：

```
Compiling IB documents for earlier than iOS 7 is no longer supported.
```

只需要打开对应的 Storyboard 文件，在右侧面板中把 `File inspector` - `Interface Builder Document` - `Builds for` 改成 `iOS 7.0 and Later` 即可。

![png](/images/2021/09/103.png)

