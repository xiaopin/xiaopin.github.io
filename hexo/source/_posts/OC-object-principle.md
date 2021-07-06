---
title: OC对象的原理(上) -- alloc的本质
tags:
    - Objective-C
    - iOS
    - macOS
abbrlink: 5f2c8255
date: 2021-06-29 14:34:46
---

<!-- 我们都知道 -->

## 前因

新建一个 iOS App 项目工程，然后先来看段代码

```ObjC
MYObject *obj1 = [MYObject alloc];
MYObject *obj2 = [obj1 init];
MYObject *obj3 = [obj1 init];

NSLog(@"%@-%p-%p", obj1, obj1, &obj1);
NSLog(@"%@-%p-%p", obj2, obj2, &obj2);
NSLog(@"%@-%p-%p", obj3, obj3, &obj3);
```

结果输出

```
<MYObject: 0x10060e110>-0x10060e110-0x7ffeefbff470
<MYObject: 0x10060e110>-0x10060e110-0x7ffeefbff460
<MYObject: 0x10060e110>-0x10060e110-0x7ffeefbff468
```

通过分析上面的结果，我们发现，obj1、obj2、obj3 这三个变量都指向了同一个对象 `0x10060e110`。

为什么？
为什么？我明明调用了 init 方法啊
为什么？难道 init 并没有创建`新`对象给我吗

## alloc 到底干了什么

带着前面的疑问，我们一起走进 alloc 方法，看看它到底干了什么事情

### 定位 alloc

想要探究 alloc 方法的实现，那么我们必须先要定位到 alloc 的源码所在位置；有 3 中方式可以定位：

1、符号断点

-   1.1 `MYObject *obj1 = [MYObject alloc];` 该行代码先打个断点
-   1.2 下一个 alloc 符号断点(Symbolic Breakpoint...)
-   1.3 先禁用该符号断点
-   1.4 执行程序，当进入 1.1 的断点时启用 1.2 的符号断点
-   过掉当前断点，即可得到结果

![](/images/2021/06/007.png)

2、control + step into

![](/images/2021/06/003.png)

![](/images/2021/06/004.png)

3、汇编查看跟流程

当进入断点后，点击 `Debug` --> `Debug Workflow` --> `Always Show Disassembly`

![](/images/2021/06/005.png)

![](/images/2021/06/006.png)

上面的 2 和 3 方法都只能找到 alloc 最终调用了 `objc_alloc`，但是这个方法又是在哪定义的呢？我们现在也不知道，所以需要结合 1 方法来进行定位。

![](/images/2021/06/008.png)

通过上面的定位，我们知道了 alloc 方法位于 `libobjc.A.dylib` 动态库中，也就是 [objc](https://opensource.apple.com/tarballs/objc4/) 库。

### 分析 alloc

知道了 alloc 的源码所在，接下来我们就开始编译调试 objc 源码，有关编译过程请参考我的这篇文章 [objc4-818.2 源码编译调试](/posts/1204fdf4/index.html)，这里不再赘述。

#### LLVM 优化

我们明明调用的是 alloc 方法，但是通过上面的跟踪定位，你会发现最终调用的都是 objc_alloc 函数，这意味着 `@selector(alloc)` 所对应的 `imp` 被做了手脚并指向了 `objc_alloc` 的地址。相关代码大家可以查阅下 `fixupMessageRef(message_ref_t *msg)` 这个函数，下面是该函数的部分代码：

```ObjC
static void
fixupMessageRef(message_ref_t *msg)
{
    msg->sel = sel_registerName((const char *)msg->sel);

    if (msg->imp == &objc_msgSend_fixup) {
        if (msg->sel == @selector(alloc)) {
            msg->imp = (IMP)&objc_alloc;
        } else if (msg->sel == @selector(allocWithZone:)) {
            msg->imp = (IMP)&objc_allocWithZone;
        } else if (msg->sel == @selector(retain)) {
            msg->imp = (IMP)&objc_retain;
        } else if (msg->sel == @selector(release)) {
            msg->imp = (IMP)&objc_release;
        } else if (msg->sel == @selector(autorelease)) {
            msg->imp = (IMP)&objc_autorelease;
        } else {
            msg->imp = &objc_msgSend_fixedup;
        }
    }
    ...
}
```

通过该函数我们发现，苹果不仅仅是 Hook 了 alloc 方法，也 Hook 了其他的方法。

#### alloc 分析

类首次调用 alloc 流程会有点区别

```ObjC
MYObject *obj1 = [MYObject alloc];
MYObject *obj2 = [MYObject alloc];
```

-   `MYObject *obj1 = [MYObject alloc];` 的执行流程

![](/images/2021/06/009.png)

-   `MYObject *obj2 = [MYObject alloc];` 的执行流程

![](/images/2021/06/010.png)

非首次调用的时候，少了 alloc 这一层的调用，原因在于 objc_alloc -> callAlloc 的时候，走了 \_objc_rootAllocWithZone 这个分支，并未走 objc_msgSend 流程，所以最终并未调用 [NSObject alloc] 方法，感兴趣的话自己可以深入研究研究。

## alloc 流程图

> 不太会画图, 后面再补上吧

## new 又做了什么？

以下是 new 方法的具体实现：

```ObjC
+ (id)new {
    return [callAlloc(self, false/*checkNil*/) init];
}
```

通过上面对 alloc 的分析我们已经很清楚知道，alloc 最终也是调用 `callAlloc` 方法，所以可以得出结论 `[NSObject new]` 实际等同于 `[[NSObject alloc] init]`。

> 既然 new 这么好用，不仅能节省代码量还更贴近现代语言语义，为什么大家都不用呢？

上面 new 的源码已经十分清楚地告诉你了，new 内部只能调用 init 方法来进行初始化，并不适用于带参初始化的场景。

比如：

```ObjC
@interface MYObject : NSObject

@property (nonatomic, copy) NSString *name;

- (instancetype)initWithName:(NSString *)name;

@end
```

针对 `initWithName:` 这种带参初始化，new 显然做不到，所以有它自身的局限性，并不是不推荐使用，而是实在是有点鸡肋。

## 后果

因为对象内存是在 `alloc` 方法中进行开辟的，`init` 方法默认不做任何处理直接返回对象本身，所以 obj1、obj2、obj3 会同时指向同一个对象(内存地址)。
