---
title: 升级 Xcode15 后老项目报错
abbrlink: 7c0ab59
date: 2023-10-26 21:42:21
tags:
    - iOS
---

升级到 Xcode15 后，打开老项目，编译时会遇到如下错误：

```
Assertion failed: (false && "compact unwind compressed function offset doesn't fit in 24 bits"), function operator(), file Layout.cpp, line 5758.
```

![](/images/2023/20231026213513.png)

在 `Build Settings` -- `Other Linker Flags ` 中添加 `-ld64`，`command + shift + K` 更新缓存并重新编译即可。

![](/images/2023/20231026213944.png)

参考这里 [https://developer.apple.com/forums/thread/735426](https://developer.apple.com/forums/thread/735426)
