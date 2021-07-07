---
title: isKindOfClass & isMemberOfClass 的分析
abbrlink: 9972ffa7
date: 2021-07-07 15:13:30
tags:
  - Objective-C
  - iOS
  - macOS
---

## 测试代码

```ObjC
@interface MYObject : NSObject

@end

@implementation MYObject

@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        BOOL r1 = [(id)[NSObject class] isKindOfClass:[NSObject class]];
        BOOL r2 = [(id)[NSObject class] isMemberOfClass:[NSObject class]];
        BOOL r3 = [(id)[MYObject class] isKindOfClass:[MYObject class]];
        BOOL r4 = [(id)[MYObject class] isMemberOfClass:[MYObject class]];
        printf("%d - %d - %d - %d\n", r1, r2, r3, r4);

        BOOL r5 = [(id)[NSObject alloc] isKindOfClass:[NSObject class]];
        BOOL r6 = [(id)[NSObject alloc] isMemberOfClass:[NSObject class]];
        BOOL r7 = [(id)[MYObject alloc] isKindOfClass:[MYObject class]];
        BOOL r8 = [(id)[MYObject alloc] isMemberOfClass:[MYObject class]];
        printf("%d - %d - %d - %d\n", r5, r6, r7, r8);
    }
    return 0;
}
```

最终输出结果如下：

```
1 - 0 - 0 - 0
1 - 1 - 1 - 1
```

## 源码溯源

### 找出背后调用的方法

首先我们的第一反应就是 objc 源码中直接全局搜索 `isKindOfClass:`，也并没有让我们失望，在 `NSObject.mm` 文件中找到了两个相关的方法：

```ObjC
+ (BOOL)isKindOfClass:(Class)cls {
    for (Class tcls = self->ISA(); tcls; tcls = tcls->getSuperclass()) {
        if (tcls == cls) return YES;
    }
    return NO;
}

- (BOOL)isKindOfClass:(Class)cls {
    for (Class tcls = [self class]; tcls; tcls = tcls->getSuperclass()) {
        if (tcls == cls) return YES;
    }
    return NO;
}
```

真想是否像我们想象的那样，即将浮出水面呢？让我们打个断点调试一下吧

![](/images/2021/07/101.png)

让我们满怀期待的按下 Command + R 运行一下吧。

**Oh my god! 什么鬼！我的断点怎么没进来呢？？？**

让我们在 r1 这行代码中打个断点，然后查看下汇编代码：

![](/images/2021/07/102.png)

原来眼前的一切皆是假象，背后调用的实际是 `objc_opt_isKindOfClass` 这个函数。

### objc_opt_isKindOfClass

该函数的实现代码如下：

```
// Calls [obj isKindOfClass]
BOOL
objc_opt_isKindOfClass(id obj, Class otherClass)
{
#if __OBJC2__
    if (slowpath(!obj)) return NO;
    Class cls = obj->getIsa();
    if (fastpath(!cls->hasCustomCore())) {
        for (Class tcls = cls; tcls; tcls = tcls->getSuperclass()) {
            if (tcls == otherClass) return YES;
        }
        return NO;
    }
#endif
    return ((BOOL(*)(id, SEL, Class))objc_msgSend)(obj, @selector(isKindOfClass:), otherClass);
}
```

通过源码我们可以知道，除非 `fastpath(!cls->hasCustomCore())` 返回 `false` 或者在非 `__OBJC2__` 中才会走 `isKindOfClass:` 方法，这也就说明了为什么我们的断点没有进来的原因。

## 分析 isKindOfClass 执行过程

- 分析 `BOOL r1 = [(id)[NSObject class] isKindOfClass:[NSObject class]];`：
    - 进入 objc_opt_isKindOfClass 方法
    - otherClass 指向的是 NSObject 类
    - cls 通过 isa 指向根元类, 所以循环体中的 tcls 第一次指向的就是根元类
    - 根元类 ≠ 类，此时通过 `getSuperclass` 获取父类，根元类的父类就是 NSObject 类
    - 所以 tcls 和 otherClass 相等，返回 YES

- 分析 `BOOL r3 = [(id)[MYObject class] isKindOfClass:[MYObject class]];`：
    - 进入 objc_opt_isKindOfClass 方法
    - otherClass 指向的是 MYObject 类
    - cls 通过 isa 指向 MYObject 的元类, 所以循环体中的 tcls 第一次指向的就是 MYObject 元类
    - 元类 ≠ 类，此时通过 `getSuperclass` 获取父类(元类的父类就是父类的元类)，此时 tcls 指向 NSObject 的元类
    - ...
    - 经过几轮比对，tcls 都不等于 otherClass，所以返回了 NO

- 分析 `BOOL r7 = [(id)[MYObject alloc] isKindOfClass:[MYObject class]];`
    - 进入 objc_opt_isKindOfClass 方法
    - otherClass 指向的是 MYObject 类
    - 实例对象的 isa 指向类，所以此时的 cls 为 MYObject 类
    - MYObject 类 == MYObject 类，返回 YES

## isMemberOfClass

```ObjC
+ (BOOL)isMemberOfClass:(Class)cls {
    return self->ISA() == cls;
}

- (BOOL)isMemberOfClass:(Class)cls {
    return [self class] == cls;
}
```

`isMemberOfClass:` 的分析参考 `isKindOfClass:` 的过程就行了，这里不再阐述；归根结底还是要理解好 isa 的流程图。

## isa 流程图

> 请牢牢记住并充分理解 isa 流程图，这个是分析底层代码执行的前提。

![](/images/2021/07/isa.png)
