---
title: 极光推送在iOS正式环境收不到通知
abbrlink: 7524f046
date: 2022-04-28 13:46:04
tags:
    - iOS
---

iOS app 中使用了极光推送，并且在 debug 模式下也能正常接收到通知消息，但是一旦切换到 release 正式环境下，则无法收到通知消息。

主要原因是因为 JAVA 后台通过 SDK 调用极光接口时，`apns_production` 参数的值不匹配导致的，在后台代码中将该值设置为 `true` 即可。

![](/images/2022/1651115235134.png)

## 参考文档

- [SDK FAQ](https://docs.jiguang.cn/jpush/client/iOS/ios_faq)
- [iOS 两种环境测试的相关知识](https://community.jiguang.cn/article/57171)