---
title: 修复 iOS12.1 UITabBar 布局错乱的bug
date: 2018-11-02 17:16:34
tags:
	- iOS
---

> 此 Bug 是在 iOS 12.1 Beta2 版本中被引入的，没想到在 iOS 12.1 正式版中并未修复

## Bug触发条件

- 使用 UITabBarController + UINavigationController 组合
- UITabBar带半透明效果，`isTranslucent` 属性为 YES
- UIViewController的 `hidesBottomBarWhenPushed` 属性为 YES
- 通过导航栏返回上一页时(导航栏返回按钮 or 苹果左侧的滑动返回手势)


## Bug演示

![gif](/images/201811/iOS12.1-UITabBar-bug.gif)


## 解决方案

```ObjC
@interface XPTabBarButton : UIView

@end

@implementation XPTabBarButton

+ (void)load {
    if (@available(iOS 12.1, *)) {
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^{
            Class originalClass = NSClassFromString(@"UITabBarButton");
            SEL originalSelector = @selector(setFrame:);
            SEL swizzledSelector = @selector(xp_setFrame:);
            
            Method originalMethod = class_getInstanceMethod(originalClass, originalSelector);
            Method swizzledMethod = class_getInstanceMethod(self, swizzledSelector);
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
}

- (void)xp_setFrame:(CGRect)frame {
    if (!CGRectIsEmpty(self.frame)) {
        // for iPhone 8/8Plus
        if (CGRectIsEmpty(frame)) {
            return;
        }
        // for iPhone XS/XS Max/XR
        frame.size.height = MAX(frame.size.height, 48.0);
    }
    [self xp_setFrame:frame];
}

@end
```

不知为何，在非刘海屏机型上，frame 的 size 为 `{0, 0}`，但是在刘海屏上却不是这个值，而是高度为 `33.0` 的尺寸(也不确定这个值是否固定为33.0)。

## 使用

直接将代码拷贝到项目即可，无需进行任何方法调用。