---
title: Swift选项集合(OptionSet)
date: 2018-04-13 08:46:38
tags:
	- Swift
---

### Objective-C中的NS_OPTION宏

对于iOS开发者来说，`NS_OPTION`这个宏应该都不会陌生吧，这个宏给我们的开发带来了极大的便利。

```ObjC
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
    UIViewAutoresizingNone                 = 0,
    UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
    UIViewAutoresizingFlexibleWidth        = 1 << 1,
    UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
    UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
    UIViewAutoresizingFlexibleHeight       = 1 << 4,
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};
```

Objective-C中可以通过`|`来组合选项，通过`&`来判断给定的选项中是否包含某个成员。

```ObjC
UIViewAutoresizing autoresizing = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
if (autoresizing & UIViewAutoresizingFlexibleWidth) {
    ...
}
```

通过`|=`来追加选项。

```ObjC
autoresizing |= UIViewAutoresizingFlexibleLeftMargin;
```


### OptionSet协议

Swift使用struct来遵从`OptionSet`协议，以引入选项集合，而非enum。为什么这样处理呢？当枚举成员互斥的时候，比如说，一次只有一个选项可以被选择的情况下，枚举是非常好的。但是和 C 不同，在 Swift 中，你无法把多个枚举成员组合成一个值，而 C 中的枚举对编译器来说就是整型，可以接受任意整数值。

```Swift
struct Hobbies: OptionSet {
    var rawValue: UInt
    
    static let none         = Hobbies(rawValue: 0) // 这人很懒,居然没任何爱好😂
    static let basketball   = Hobbies(rawValue: 1 << 0)
    static let football     = Hobbies(rawValue: 1 << 1)
    static let baseball     = Hobbies(rawValue: 1 << 2)
    static let swim         = Hobbies(rawValue: 1 << 3)
    static let badminton    = Hobbies(rawValue: 1 << 4)
    static let all: Hobbies = [.basketball, .football, .baseball, .swim, .badminton]
}
```

和C一样，Swift中的选项集合结构体使用了高效的位域来表示，但是这个结构体本身表现为一个集合，它的成员则为被选择的选项。这允许你使用标准的集合运算来维护位域，比如使用`contains(_:)`来检验集合中是否有某个成员，或者是用`union(_:)`来组合两个位域，通过`remove(_:)`来移除某个成员。另外，由于OptionSet继承于ExpressibleByArrayLiteral，你可以使用数组字面量来生成一个选项集合。

```Swift
var hobbies: Hobbies = [.football, .swim]
_ = hobbies.contains(.football) // true
_ = hobbies.contains(.baseball) // false
// 增加`.badminton`选项
hobbies = hobbies.union(.badminton)
// 移除`.football`选项
hobbies.remove(.football)
```


### 源码

你可以在GitHub上查看到OptionSet的源码[`swift/stdlib/public/core/OptionSet.swift`](https://github.com/apple/swift/blob/master/stdlib/public/core/OptionSet.swift)，如果你有兴趣。
