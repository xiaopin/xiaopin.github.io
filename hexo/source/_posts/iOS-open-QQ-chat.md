---
title: iOS打开QQ临时会话窗口
tags:
  - iOS
abbrlink: 3c37c29d
date: 2018-01-16 15:03:50
---

> 前提条件：接收临时会话的QQ号码必须开通[QQ推广](http://shang.qq.com)功能，否则不能打开临时会话窗口。

公司项目在商品详情页面中，有一个客服按钮，点击后需要打开客服MM的QQ聊天窗口，在此可以向客服MM咨询商品相关的问题。这里就记录下iOS如何跳转到QQ的聊天窗口。

- 在Info.plise中加入白名单

```
<key>LSApplicationQueriesSchemes</key>
<array>
    <string>mqq</string>
</array>
```

- 示例代码

```Swift
if UIApplication.shared.canOpenURL(URL(string: "mqq://")!) {
    let qq = "12345678"
    if let url = URL(string: "mqq://im/chat?chat_type=wpa&uin=\(qq)&version=1&src_type=web") {
        if #available(iOS 10.0, *) {
            UIApplication.shared.open(url, options: [:], completionHandler: nil)
        } else {
            UIApplication.shared.openURL(url)
        }
    }
}
```
