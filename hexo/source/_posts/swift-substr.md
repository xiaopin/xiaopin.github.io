---
title: Swift截取子字符串
date: 2018-03-29 14:57:02
tags:
	- Swift
---

在Swift中截取子串，一直是个头疼的问题，毕竟牵涉到`String.Index`这个鬼东西，而且API还不好用。现在连`substring(to:)`、`substring(from:)`、`substring(width:)`这几个方法也被废弃了。

### ***One-sided Slicing***

好在Swift4之后，苹果为String增加了一个语法糖`...`，可以对字符串进行单侧边界取子串。注意返回的类型是`Substring`而不是`String`。

```Swift
extension String {

    /// 截取子字符串
    ///
    /// - Parameters:
    ///   - start: 要抽取的子串的起始下标。如果是负数，那么该参数声明从字符串的尾部开始算起的位置。
    ///            也就是说，-1 指字符串中最后一个字符，-2 指倒数第二个字符，以此类推。
    ///   - length: 截取长度。如果省略了该参数，那么返回从`start`开始到结尾的子串。
    /// - Returns: 子字符串
    func substr(_ start: Int, length: UInt? = nil) -> String? {
        if abs(start) >= count { return nil }
        let offset = (start >= 0) ? start : count+start
        let s = self.index(startIndex, offsetBy: offset)
        guard let length = length else {
            // length==nil,直接截取到字符串结尾
            return String(self[s...])
        }
        let e = self.index(startIndex, offsetBy: min(offset+Int(length), count))
        return String(self[s..<e])
    }

}
```

如果懂JavaScript的话，一看就懂，这函数和JS的substr是一样的用法。