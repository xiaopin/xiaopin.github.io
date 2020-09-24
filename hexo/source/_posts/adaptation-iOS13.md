---
title: 适配iOS13
tags:
  - iOS
abbrlink: 3de45a05
date: 2019-09-28 14:07:58
---

虽然苹果在今天推送了 `iOS13.1.1 `，但还是简单记录下适配 iOS13 的过程吧。

## KVC禁止访问私有属性

在 Xcode11 中苹果禁止了开发者通过 KVC 方式获取私有属性，如果你在代码中通过该方式访问了不该访问的东西，App 则会闪退并在控制台中可以看到如下错误信息：
```
Terminating app due to uncaught exception 'NSGenericException', reason: 'Access to UISearchBar's _searchField ivar is prohibited. This is an application bug'
```

简单明了，就是不允许你访问了！！！

- 获取 UISearchBar 的 UITextField

在 iOS13 中，苹果提供了 `searchTextField` 属性来获取对应的 UITextField，无需通过 KVC 获取。但是我们需要兼容旧系统，所以我的解决方案是给 iOS13 以下的系统动态添加 searchTextField 方法：
```ObjC
UITextField* UISearchBar_searchTextField(id self, SEL _cmd)
{
    if (@available(iOS 13.0, *)) {
        return [(UISearchBar *)self searchTextField];
    }
    return [self valueForKey:@"_searchField"];
}

@implementation UISearchBar (iOS13)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        if (@available(iOS 13.0, *)) {
            // do nothing.
        } else {
            // 动态添加 `-searchTextField` 方法, 兼容iOS13以下版本
            class_addMethod(self, @selector(searchTextField), (IMP)UISearchBar_searchTextField, "@@:");
        }
    });
}

@end
```

- 导航栏透明度

通过访问导航栏私有视图 `_backgroundEffectView` 并设置其透明度来达到导航栏透明的效果，现在已经不需要这么做了。

下面给出核心代码：
```ObjC
- (void)setNavigationBarBackgroundAlpha:(CGFloat)alpha {
    UIView *barBackgroundView = self.navigationBar.subviews.firstObject;
    if (@available(iOS 13.0, *)) {
        barBackgroundView.alpha = alpha;
        return;
    }
    // 处理底部分割线
    UIView *shadowView = [barBackgroundView valueForKey:@"_shadowView"];
    shadowView.hidden = (alpha < 1.0);
    // 处理背景
    if (self.navigationBar.isTranslucent) {
        if (@available(iOS 10.0, *)) {
            UIView *backgroundEffectView = [barBackgroundView valueForKey:@"_backgroundEffectView"];
            backgroundEffectView.alpha = alpha;
        } else {
            UIView *adaptiveBackdrop = [barBackgroundView valueForKey:@"_adaptiveBackdrop"];
            adaptiveBackdrop.alpha = alpha;
        }
    } else {
        barBackgroundView.alpha = alpha;
    }
}
```

完整代码请点击 [NavigationBarTranslucent](https://github.com/xiaopin/NavigationBarTranslucent)。

目前我的项目只有这两个地方访问了私有属性。

## UISegmentedControl

在 iOS13 中，苹果对 UISegmentedControl 的样式做了大改动，改成了白底黑字的方块。如果你在之前的版本中，通过 `tintColor` 设置了其颜色，那么在 iOS13 中是无效的，iOS13 提供了 `selectedSegmentTintColor` 属性来设置颜色。

> 幸好产品没说一定要改成以前的边框样式，不然我真的是欲哭无泪了...

## 模态窗口的默认样式调整

在 iOS13 中，通过 `presentViewController:animated:completion:` 弹出来的控制器，`modalPresentationStyle` 属性默认是 `UIModalPresentationPageSheet`，而之前的系统版本默认的是 `UIModalPresentationFullScreen`。

不仅将默认值改为 UIModalPresentationPageSheet，还对 UIModalPresentationPageSheet 的样式也进行了修改，将 iPad 中的样式引入到了 iPhone 中，并且自带下滑关闭手势，可以通过将 `modalInPresentation` 设置为 `YES` 关闭下滑手势。

针对这个改变，你可以手动指定 modalPresentationStyle 的值为 UIModalPresentationFullScreen 来恢复以前的样式。

如果想懒的话，也可以通过 Category 来解决(个人不推荐，自己决定)：
```ObjC
#import <objc/runtime.h>

@interface UIViewController (iOS13ResetModalStyle)

@end

@implementation UIViewController (iOS13ResetModalStyle)

static const char kIsSetedModalPresentationStyleKey = '\0';
 
+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        if (@available(iOS 13.0, *)) {
            NSArray *sels = @[
                NSStringFromSelector(@selector(setModalPresentationStyle:)),
                NSStringFromSelector(@selector(presentViewController:animated:completion:)),
            ];
            for (NSString *name in sels) {
                Method originalMethod = class_getInstanceMethod(self, NSSelectorFromString(name));
                Method swizlledMethod = class_getInstanceMethod(self, NSSelectorFromString([@"xp_" stringByAppendingString:name]));
                method_exchangeImplementations(originalMethod, swizlledMethod);
            }
        }
    });
}

- (void)xp_setModalPresentationStyle:(UIModalPresentationStyle)modalPresentationStyle {
    objc_setAssociatedObject(self, &kIsSetedModalPresentationStyleKey, @(YES), OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    [self xp_setModalPresentationStyle:modalPresentationStyle];
}

- (void)xp_presentViewController:(UIViewController *)viewControllerToPresent animated:(BOOL)flag completion:(void (^)(void))completion {
    if (@available(iOS 13.0, *)) {
        switch (viewControllerToPresent.modalPresentationStyle) {
            case UIModalPresentationPageSheet:
            case UIModalPresentationNone:
            case UIModalPresentationAutomatic:
            {
                BOOL isSetedModalPresentationStyle = [objc_getAssociatedObject(viewControllerToPresent, &kIsSetedModalPresentationStyleKey) boolValue];
                if (!isSetedModalPresentationStyle) {
                    viewControllerToPresent.modalPresentationStyle = UIModalPresentationFullScreen;
                }
            }
                break;
            default:
                break;
        }
    }
    [self xp_presentViewController:viewControllerToPresent animated:flag completion:completion];
}

@end
```

项目中只有登录页面、UIAlertController、UIImagePickerController等几个控制器是通过 `presentViewController:animated:completion:` 方式弹出来的，由于 UIAlertController 的特殊性不会被影响，实际受影响的只有登录页面和从系统相册选择相片这两个功能；由于页面少，且产品觉得在不影响使用的情况下就不用修改了，所以上面代码虽然撸了，但是在项目中是被注释掉的。

## Dark Mode

> 适配可以参看 UIView 和 UIViewController 的 `overrideUserInterfaceStyle` 属性。

项目并不打算适配暗黑模式，但是如果在 iPhone 中开启了 Dark Mode，将会导致 App 的部分视图变黑，UI很难看，所以这里需要关闭 Dark Mode 功能，告诉系统，我的 App 不支持暗黑模式，你别给我变黑了...

需要在 `Info.plist` 中加入 UIUserInterfaceStyle 这个key并指定样式为 `Light` 或 `Dark`：
```
<key>UIUserInterfaceStyle</key>
<string>Light</string>
```

通过将样式指定为 Light 后，App 将不会受暗黑模式的影响了。

PS: 公司项目在 Info.plist 中添加 UIUserInterfaceStyle 并已成功上架！！！

## UISearchDisplayController

UISearchDisplayController 在 iOS8 的时候就被废弃了，虽然在之后的版本中也一直可以使用，但是在最新的 iOS13 中，UISearchDisplayController 终于寿终正寝了，如果你在 iOS13 中继续使用它，那么你将收到如下的Crash信息：
```
Terminating app due to uncaught exception 'NSGenericException', reason: 'UISearchDisplayController is no longer supported when linking against this version of iOS. Please migrate your application to UISearchController.'
```

还是乖乖使用 `UISearchController` 吧。

## 蓝牙

在以前，App 可以直接使用蓝牙功能，不会出现权限的提示弹窗，在 iOS13 中，想使用蓝牙，则需要申请权限才行了，和相机、定位一样。

## Sign With Apple

[Authenticating Users with Sign in with Apple](https://developer.apple.com/documentation/signinwithapplerestapi/authenticating_users_with_sign_in_with_apple?language=data)
[WWDC2019](https://developer.apple.com/videos/play/wwdc2019/706/)

## 导航栏样式的定制

一般项目中都会有自定义导航栏的需求（修改标题文字样式、UIBarButtonItem样式、返回按钮等），在 iOS13 之前都是通过 `UIAppearance` 协议获取到一个全局的外观代理对象，然后通过该代理进行样式自定义，用法如下：
```ObjC
@implementation UINavigationController

+ (void)initialize {
    UIImage *backImage = [[UIImage imageNamed:@"icon-back"] imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal];
    UINavigationBar *navigationBar = [UINavigationBar appearance];
    navigationBar.translucent = YES;
    navigationBar.titleTextAttributes = @{NSForegroundColorAttributeName: [UIColor whiteColor]};
    navigationBar.backIndicatorImage = backImage;
    navigationBar.backIndicatorTransitionMaskImage = backImage;
    
    NSDictionary *attributes = @{
                                 NSFontAttributeName: [UIFont systemFontOfSize:15.0],
                                 NSForegroundColorAttributeName: [UIColor whiteColor]
                                 };
    UIBarButtonItem *barButtonItem = [UIBarButtonItem appearanceWhenContainedInInstancesOfClasses:@[[UINavigationBar class]]];
    [barButtonItem setTitleTextAttributes:attributes forState:UIControlStateNormal];
    [barButtonItem setTitleTextAttributes:attributes forState:UIControlStateHighlighted];
}

@end
```

在 iOS13 中，苹果新增了几个类：`UINavigationBarAppearance`、`UIBarAppearance`、`UIBarButtonItemAppearance`、`UIBarButtonItemStateAppearance`，通过这几个类，可以定制导航栏的标题、背景、返回按钮(参看[UINavigationBarAppearance](https://developer.apple.com/documentation/uikit/uinavigationbarappearance)类所提供的接口)，以及UIBarButtonItem在`normal`、`highlighted`、`disabled`、`focused`几种状态下的样式(参看[UIBarButtonItemAppearance](https://developer.apple.com/documentation/uikit/uibarbuttonitemappearance)类所提供的接口)。

苹果已经在 UINavigationBar 中提供了相关的接口来访问这些类：
```ObjC
/*
 Fallback Behavior:
 1) Appearance objects are used in whole – that is, all values will be sourced entirely from an instance of UINavigationBarAppearance defined by one of these named properties (standardAppearance, compactAppearance, scrollEdgeAppearance) on either UINavigationBar (self) or UINavigationItem (self.topItem).
 2) The navigation bar will always attempt to use the most relevant appearance instances first, before falling back to less relevant ones. The fallback logic is:
     AtScrollEdge: self.topItem.scrollEdgeAppearance => self.scrollEdgeAppearance => self.topItem.standardAppearance => self.standardAppearance
     CompactSize: self.topItem.compactAppearance => self.compactAppearance => self.topItem.standardAppearance => self.standardAppearance
     NormalSize: self.topItem.standardAppearance => self.standardAppearance
 */

/// Describes the appearance attributes for the navigation bar to use when it is displayed with its standard height.
@property (nonatomic, readwrite, copy) UINavigationBarAppearance *standardAppearance UI_APPEARANCE_SELECTOR API_AVAILABLE(ios(13.0), tvos(13.0));
/// Describes the appearance attributes for the navigation bar to use when it is displayed with its compact height. If not set, the standardAppearance will be used instead.
@property (nonatomic, readwrite, copy, nullable) UINavigationBarAppearance *compactAppearance UI_APPEARANCE_SELECTOR API_AVAILABLE(ios(13.0));
/// Describes the appearance attributes for the navigation bar to use when an associated UIScrollView has reached the edge abutting the bar (the top edge for the navigation bar). If not set, a modified standardAppearance will be used instead.
@property (nonatomic, readwrite, copy, nullable) UINavigationBarAppearance *scrollEdgeAppearance UI_APPEARANCE_SELECTOR API_AVAILABLE(ios(13.0));
```

基本用法如下：
```ObjC
@implementation UINavigationBar

+ (void)initialize {
    if (@available(iOS 13.0, *)) {
        UINavigationBarAppearance *standardAppearance = [[UINavigationBarAppearance alloc] init];
        standardAppearance.titleTextAttributes = @{NSForegroundColorAttributeName: [UIColor redColor]};
        standardAppearance.backgroundColor = [UIColor orangeColor];
        // 设置返回按钮(可以通过standardAppearance.backButtonAppearance进一步定制)
        UIImage *backImage = [[UIImage imageNamed:@"icon-back"] imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal];
        [standardAppearance setBackIndicatorImage:backImage transitionMaskImage:backImage];
        // 设置UIBarButtonItem样式
        NSDictionary *attributes = @{
            NSFontAttributeName: [UIFont systemFontOfSize:15.0],
            NSForegroundColorAttributeName: [UIColor whiteColor]
        };
        UIBarButtonItemAppearance *buttonAppearance = standardAppearance.buttonAppearance;
        buttonAppearance.normal.titleTextAttributes = attributes;
        buttonAppearance.highlighted.titleTextAttributes = attributes;
        // 记得覆盖原来的样式
        [[UINavigationBar appearance] setStandardAppearance:standardAppearance];
    } else {
        // 之前的做法
    }
}

@end
```

参考：

- [iOS13AdaptationTips](https://github.com/ChenYilong/iOS13AdaptationTips)
