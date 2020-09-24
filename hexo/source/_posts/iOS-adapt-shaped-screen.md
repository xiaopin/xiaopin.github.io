---
title: iOS 刘海屏适配
tags:
  - iOS
abbrlink: a933fc8a
date: 2018-09-21 16:21:12
---

今年苹果发布了三款 iPhone，分别为 iPhone XS、iPhone XS Max 以及 iPhone XR，尽管大家都在吐槽刘海屏，但是苹果今年已经是 iPhone 系列标配刘海屏了。

虽然苹果为我们带来了更大屏幕尺寸的 iPhone XS Max 和 iPhone XR，尽管一个是6.5英寸一个是6.1英寸，并且 iPhone XS Max 的分辨率更是达到了 1242px × 2688px，但是对于我们开发者来说，这两者的物理尺寸都一个样，都是 `414×896pt`，而 6/7/8 Plus 系列的宽度也是 414pt，所以你可以把他们看作是 plus 系列的加长版。

***

```ObjC
/// 是否为异形屏(刘海屏)
BOOL isShapedScreen(void)
{
    if (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPhone) {
        if (@available(iOS 11.0, *)) {
            static BOOL result = NO;
            static dispatch_once_t onceToken;
            dispatch_once(&onceToken, ^{
                CGFloat width = MIN(UIScreen.mainScreen.bounds.size.width, UIScreen.mainScreen.bounds.size.height);
                CGFloat height = MAX(UIScreen.mainScreen.bounds.size.width, UIScreen.mainScreen.bounds.size.height);
                if (@available(iOS 12.0, *)) {
                    if (width == 414.0 && height == 896.0) {
                        result = YES; // iPhone XS Max / iPhone XR
                    }
                }
                if (width == 375.0 && height == 812.0) {
                    result = YES; // iPhone X / iPhone XS
                }
            });
            return result;
        }
    }
    return NO;
}

/// 是否为iPhone X / iPhone XS
BOOL iPhoneX(void)
{
    if (@available(iOS 11.0, *)) {
        static BOOL result = NO;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^{
            if (isShapedScreen()) {
                CGFloat height = MAX(UIScreen.mainScreen.bounds.size.width, UIScreen.mainScreen.bounds.size.height);
                if (height == 812.0) {
                    result = YES;
                }
            }
        });
        return result;
    }
    return NO;
}

/// 是否为iPhone XS Max
BOOL iPhoneXSMax(void)
{
    if (@available(iOS 12.0, *)) {
        static BOOL result = NO;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^{
            if (isShapedScreen()) {
                CGFloat height = MAX(UIScreen.mainScreen.bounds.size.width, UIScreen.mainScreen.bounds.size.height);
                if (height == 896.0 && UIScreen.mainScreen.scale == 3.0) {
                    result = YES;
                }
            }
        });
        return result;
    }
    return NO;
}

/// 是否为iPhone XR
BOOL iPhoneXR(void)
{
    if (@available(iOS 12.0, *)) {
        static BOOL result = NO;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^{
            if (isShapedScreen()) {
                CGFloat height = MAX(UIScreen.mainScreen.bounds.size.width, UIScreen.mainScreen.bounds.size.height);
                if (height == 896.0 && UIScreen.mainScreen.scale == 2.0) {
                    result = YES;
                }
            }
        });
        return result;
    }
    return NO;
}
```

***

附上 Swift 代码

```Swift
/// 是否为异形屏(刘海屏)
let isShapedScreen : Bool = {
    if UI_USER_INTERFACE_IDIOM() == .phone {
        if #available(iOS 11.0, *) {
            let width = min(UIScreen.main.bounds.width, UIScreen.main.bounds.height)
            let height = max(UIScreen.main.bounds.width, UIScreen.main.bounds.height)
            if #available(iOS 12.0, *) {
                if width == 414.0 && height == 896.0 {
                    return true // iPhone XS Max / iPhone XR
                }
            }
            if (width == 375.0 && height == 812.0) {
                return true // iPhone X / iPhone XS
            }
        }
    }
    return false
}()

/// 是否为iPhone X / iPhone XS
let iPhoneX : Bool = {
    if #available(iOS 11.0, *) {
        if isShapedScreen {
            let height = max(UIScreen.main.bounds.width, UIScreen.main.bounds.height)
            return (height == 812.0)
        }
    }
    return false
}()

/// 是否为iPhone XS Max
let iPhoneXSMax : Bool = {
    if #available(iOS 12.0, *) {
        if isShapedScreen {
            let height = max(UIScreen.main.bounds.width, UIScreen.main.bounds.height)
            let scale = UIScreen.main.scale
            if height == 896.0 && scale == 3.0 {
                return true
            }
        }
    }
    return false
}()

/// 是否为iPhone XR
let iPhoneXR : Bool = {
    if #available(iOS 12.0, *) {
        if isShapedScreen {
            let height = max(UIScreen.main.bounds.width, UIScreen.main.bounds.height)
            let scale = UIScreen.main.scale
            if height == 896.0 && scale == 2.0 {
                return true
            }
        }
    }
    return false
}()
```

***

| 设备 | 分辨率 | 图片资源 |
|:-|:-|:-:|
|12.9" iPad Pro		|	2048px × 2732px		|  @2x |
|10.5" iPad Pro		|	1668px × 2224px		|  @2x |
|9.7" iPad			|	1536px × 2048px		|  @2x |
|7.9" iPad mini 4	|	1536px × 2048px		|  @2x |
|iPhone XS Max		|	1242px × 2688px		|  @3x |
|iPhone XS			|	1125px × 2436px		|  @3x |
|iPhone XR			|	828px × 1792px		|  @2x |
|iPhone X			|	1125px × 2436px		|  @3x |
|iPhone 8 Plus		|	1242px × 2208px		|  @3x |
|iPhone 8			|	750px × 1334px		|  @2x |
|iPhone 7 Plus		|	1242px × 2208px		|  @3x |
|iPhone 7			|	750px × 1334px		|  @2x |
|iPhone 6s Plus		|	1242px × 2208px		|  @3x |
|iPhone 6s			|	750px × 1334px		|  @2x |
|iPhone SE			|	640px × 1136px		|  @2x |


参考：
- [https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/adaptivity-and-layout/](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/adaptivity-and-layout/)
- [https://developer.apple.com/design/human-interface-guidelines/ios/icons-and-images/image-size-and-resolution/](https://developer.apple.com/design/human-interface-guidelines/ios/icons-and-images/image-size-and-resolution/)
- [https://developer.apple.com/library/archive/documentation/DeviceInformation/Reference/iOSDeviceCompatibility/Displays/Displays.html#//apple_ref/doc/uid/TP40013599-CH108-SW6](https://developer.apple.com/library/archive/documentation/DeviceInformation/Reference/iOSDeviceCompatibility/Displays/Displays.html#//apple_ref/doc/uid/TP40013599-CH108-SW6)
