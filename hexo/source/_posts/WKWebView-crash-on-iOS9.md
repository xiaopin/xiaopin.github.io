---
title: WKWebView在iOS9上崩溃
tags:
  - iOS
abbrlink: be7d7959
date: 2018-07-06 18:49:00
---

最近项目重构，所以采用了 WKWebView 来展示网页内容，但是发现在 iOS9 上程序会出现 crash，以前也用过 WKWebView 但是都没遇到这么坑的情况啊。

具体操作：通过 push 网页控制器，然后在 pop 之后，程序就崩了。


打断点也定位不到 crash 代码行，打开 Xcode 的僵尸模式之后，控制台也仅输出以下信息：
```
-[XPWebViewController retain]: message sent to deallocated instance 0x7fa1974a33f0
```

经过排查发现是因为 WKWebView 的 UIScrollView 代理惹的祸，因为我需要实时监听网页的滚动区域来处理一些事情，所以我把 `WKWebView.scrollView.delegate` 设置为当前控制器。

```ObjC
self.webView.scrollView.delegate = self;
```

既然知道了症结所在，那么只需要在控制器销毁的时候及时清掉代理就行了：
```ObjC
- (void)dealloc {
    self.webView.scrollView.delegate = nil; // Fix iOS9 crash.
}
```
