---
title: Method Swizzling
date: 2019-05-22 17:21:13
tags:
	- iOS
	- macOS
	- Objective-C
---

#### 交换方法实现

得益于 Objective-C 的 runtime 机制，method swizzling 这个黑魔法可以为我们在实际开发中解决诸多常规手段所不能解决的问题。

- 1、直接通过 method_exchangeImplementations 方法实现，这是用的最多的一种方式

```ObjC
@interface UIViewController (MS)

@end

@implementation UIViewController (MS)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Method originalMethod = class_getInstanceMethod(self, @selector(viewDidLoad));
        Method swizzledMethod = class_getInstanceMethod(self, @selector(xx_viewDidLoad));
        method_exchangeImplementations(originalMethod, swizzledMethod);
    });
}

- (void)xx_viewDidLoad {
    [self xx_viewDidLoad];
    // Custom coding...
}

@end
```

- 2、通过 method_setImplementation 直接交换 IMP 实现方法的替换

```ObjC
@interface UIViewController (MS)

@end

@implementation UIViewController (MS)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        original_viewDidLoad = (void*)method_setImplementation(class_getInstanceMethod(self, @selector(viewDidLoad)), (IMP)xx_viewDidLoad);
    });
}

IMP (*original_viewDidLoad)(id self, SEL _cmd);

void xx_viewDidLoad(id self, SEL _cmd)
{
    original_viewDidLoad(self, _cmd);
    // Custom coding...
}

@end
```

- 3、先通过 class_addMethod 将方法添加到类上，然后再通过 method_exchangeImplementations 交换方法实现(推荐)

```ObjC
@interface UIViewController (MS)

@end

@implementation UIViewController (MS)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        SEL originalSelector = @selector(viewDidLoad);
        SEL swizzledSelector = @selector(xx_viewDidLoad);
        Method originalMethod = class_getInstanceMethod(self, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(self, swizzledSelector);
        BOOL didAdded = class_addMethod(self, originalSelector, method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod));
        
        if (didAdded) {
            class_replaceMethod(self, swizzledSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}

- (void)xx_viewDidLoad {
    [self xx_viewDidLoad];
    // Custom coding...
}

@end
```

上面介绍了三种方式来实现 Method Swizzling，但是前两个方案有点小瑕疵，如果稍不注意，可能就掉坑里了。

**在前两者中，如果被 hook 的类并没有实现被 hook 的方法(方法在其父类中实现)，这个时候被替换的将是父类的方法实现。**

什么意思呢，看下面的代码：

```ObjC
@interface A : NSObject
- (void)test;
@end

@implementation A

- (void)test {
    NSLog(@"A");
}

@end


@interface B : A
@end

@implementation B
@end

// 本来的目的只是想 hook B 的 test
@interface B (MS)
@end

@implementation B (MS)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Method originalMethod = class_getInstanceMethod(self, @selector(test));
        Method swizzledMethod = class_getInstanceMethod(self, @selector(xx_test));
        method_exchangeImplementations(originalMethod, swizzledMethod);
    });
}

- (void)xx_test {
    NSLog(@"Hook");
}

@end
```

A 类中有个 test 方法，B 是 A 的子类，但是 B 并没有重写 test 方法，此时如果 hook B 的 test 方法，实际上是会替换了 A 的方法实现(IMP)，这个时候你创建一个 A 的实例对象并调用 test 方法，实际上调用的是 hook 的方法。也就是当你 `[[A new] test]` 时会打印 Hook 字符串，而不是打印预期的 A 字符串。

> **所以推荐使用第三种方案，不推荐使用前两者，除非你明确知道当前被 hook 的类实现了对应的方法。**

****

#### Hook 不同的类

有时候我们想 hook 一些私有类的方法，我们只知道类名和方法名，这个时候不能直接通过 Category 来进行 Method Swizzling。

比如系统的 NSArray/NSMutableArray、NSDictionary/NSMutableDictionary 类的方法，如果你直接 hook 他们，你会发现你的代码并不会被执行。

这是为什么呢？不知道大家在平时的调试中有没有发现，在 Xcode 下方 Debug 区域左侧的 Variables View 中，会发现 

- `str = (__NSCFConstantString *) @"xxx"`
- `array = (__NSArrayI *) @"2 elements"`

这样的提示，我们的代码明明写的是 NSString 和 NSArray，怎么就变成了 `__NSCFConstantString` 和 `__NSArrayI` 这些莫名其妙的类了呢。其实这便是 Foundation 中被广泛使用的类簇，有兴趣请查阅 Apple 官方文档 [`Class cluster`](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/ClassCluster.html) 的内容。

这也正是我们如果直接对 NSArray/NSDictionary 进行 Hook 而不成功的原因，因为我们搞错了被 hook 的对象，我们实际要 hook 的是 `__NSArrayI` 和 `__NSDictionaryI` 这些类，但是这些类我们仅仅是知道类名和方法名，并没有对应的头文件，所以就不能通过 Category 的方式来进行 hook 了，得采用下面的方式。

假设：我们知道有一个私有类叫做 `Car` 并且有一个 `-runWithSpeed:` 方法，该方法对速度不限速，这不符合我大天朝的规章制度啊，所以我们要把它的速度限制在 0～120 之间，不能超速了。

代码如下：

```ObjC
@interface Car : NSObject

@end

@implementation Car

- (void)runWithSpeed:(CGFloat)speed {
    NSLog(@"The car current speed is %.02f KM/h", speed);
}

@end


@interface CarHook : NSObject

@end

@implementation CarHook

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class originalClass = NSClassFromString(@"Car");
        Class swizzledClass = [self class];
        SEL originalSelector = NSSelectorFromString(@"runWithSpeed:");
        SEL swizzledSelector = @selector(xx_runWithSpeed:);
        Method originalMethod = class_getInstanceMethod(originalClass, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(swizzledClass, swizzledSelector);
        
        class_replaceMethod(originalClass,
                            swizzledSelector,
                            method_getImplementation(originalMethod),
                            method_getTypeEncoding(originalMethod));
        class_replaceMethod(originalClass,
                            originalSelector,
                            method_getImplementation(swizzledMethod),
                            method_getTypeEncoding(swizzledMethod));
    });
}

- (void)xx_runWithSpeed:(CGFloat)speed {
    NSLog(@"%s", __FUNCTION__);
    NSLog(@"self: %@, _cmd: %@", self, NSStringFromSelector(_cmd));
    if (speed >= 0.0 && speed <= 120.0) { // Limit speed.
        [self xx_runWithSpeed:speed];
    }
}

@end
```

****

- [Method Swizzling](https://nshipster.cn/method-swizzling/)
- [Method Swizzling的各种姿势](http://www.cocoachina.com/ios/20160826/17422.html)
