---
title: iOS从通讯录中复制粘贴电话号码携带特殊字符
tags:
  - iOS
abbrlink: cb622554
date: 2018-03-30 17:16:51
---

在做收货地址的功能，其中有个场景就是用户先从手机`通讯录`中拷贝了某个电话号码，然后将其粘贴到输入框中。一切看似正常，但是这里有个问题，从通讯录中拷贝的电话号码，自带了特殊的不可见unicode字符，如果你不打断点查看变量值，压根不会看到这些特殊字符。

#### 特殊字符

这些特殊字符在OC和Swift中有不同的表现形式:

Objective-C:
```
phone	__NSCFString *	@"\U0000202d(888) 555-5512\U0000202c"	0x000060000065b7b0
```
Swift:
```
phone	String	"\u{e2}(888) 555-5512\u{e2}"
```

#### 解决方案

Swift:

```Swift
func removeSpecialCharactersForPhoneNumber(_ phone: String) -> String {
    var result = phone
    ["+86","+","-","(",")"," "].forEach {
        result = result.replacingOccurrences(of: $0, with: "")
    }
    // 过滤`通讯录`拷贝的特殊字符
    result = result.replacingOccurrences(of: "\\p{Cf}", with: "", options: .regularExpression)
    return result
}
```

Objective-C

```ObjC
[phone stringByReplacingOccurrencesOfString:@"\\p{Cf}" withString:@"" options:NSRegularExpressionSearch range:NSMakeRange(0, phone.length)]
```

参考: [https://stackoverflow.com/questions/47623828/ios-copy-paste-phone-from-contacts-to-uitextfield-adds-strange-unicode-character](https://stackoverflow.com/questions/47623828/ios-copy-paste-phone-from-contacts-to-uitextfield-adds-strange-unicode-character)
