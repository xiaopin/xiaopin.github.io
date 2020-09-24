---
title: UITabBar 点击刷新功能
tags:
  - iOS
abbrlink: 8bfb49d7
date: 2018-11-12 12:48:51
---

我们都知道 UITabBar 只能添加 UITabBarItem，而 UITabBarItem 是继承自 NSObject 的，但是我们可以发现 UITabBarItem 内部有一个 [`_view`](https://github.com/nst/iOS-Runtime-Headers/blob/10.3/Frameworks/UIKit.framework/UITabBarItem.h) 属性与私有类 [`UITabBarButton`](https://github.com/nst/iOS-Runtime-Headers/blob/10.3/Frameworks/UIKit.framework/UITabBarButton.h) 关联着，而这个 UITabBarButton 正是我们在 UITabBar 上看到的一个个按钮对象。

经过调试，发现 UITabBarButton 的 `UIControlEventTouchUpInside` 事件与 UITabBar 的 `_buttonUp:` 方法关联着，所以我们只需要通过 runtime hook 该方法即可拦截到 UITabBarButton 的点击事件。

这里提醒一下，虽然 UITabBarButton 继承自 UIControl，但是对于是否选中的标记，UITabBarButton 并没有采用 UIControl 的 `selected` 属性，而是在内部用了一个私有变量 [`_selected`](https://github.com/nst/iOS-Runtime-Headers/blob/10.3/Frameworks/UIKit.framework/UITabBarButton.h) 来标记。

附上 [Demo](https://github.com/xiaopin/UITabBarRefresh.git)

## 特性

- 不依赖 UITabBarControllerDelegate，通过对 `UITabBarItem` 进行扩展来实现
- UITabBar 支持刷新动画 ，默认关闭动画
- UITabBar 支持自定义刷新动画，需遵守`XPTabBarRefreshViewAnimating`协议，默认动画是`UIActivityIndicatorView`
- 支持 iPhone/iPad，适配横竖屏

## 效果演示

![gif](/images/201811/UITabBar-Refresh.gif)

## 代码使用演示

```ObjC
// 根据项目实际场景获取UITabBarItem
UITabBarItem *tabBarItem =  self.tabBarItem; //self.navigationController.tabBarItem;
// 启用刷新动画
tabBarItem.enabledRefreshAnimation = YES;
// 自定义动画
tabBarItem.refreshView = [[CustomRefreshView alloc] init];
// 监听刷新回调
[tabBarItem setRefreshBlock:^(UITabBar *tabBar, UITabBarItem *tabBarItem) {
    // 发送网络请求刷新数据
    NSLog(@"Refresh");
    // 当网络请求完毕时记得调用 `stopRefresh` 方法
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
       [tabBarItem stopRefresh];
    });
}];
```

***关于 `UITabBarItem` 的获取说明***

- 结构1

```
UITabBarController
│
├── UIViewController
│
├── UIViewController
│
└── UIViewController
```

- 结构2

```
UINavigationController
│
└── UITabBarController
	│
	├── UIViewController
	│
	├── UIViewController
	│
	└── UIViewController
```

对于上图的结构1、2，直接使用 `UIViewController.tabBarItem` 即可。

- 结构3
```
UITabBarController
│
└── UINavigationController
	│
	├── UIViewController
	│
	├── UIViewController
	│
	└── UIViewController
```

对于上图的结构3，使用 `UIViewController.navigationController.tabBarItem` 即可。


## 源码

- .h 文件

```ObjC
#import <UIKit/UIKit.h>

@protocol XPTabBarRefreshViewAnimating;
typedef void(^XPTabBarRefreshBlock)(UITabBar *tabBar, UITabBarItem *tabBarItem);


NS_CLASS_AVAILABLE_IOS(8_0) @interface UITabBarItem (XPTabBarRefresh)

/// 刷新回调
@property (nonatomic, copy) XPTabBarRefreshBlock refreshBlock;
/// 刷新动画视图, 默认`UIActivityIndicatorView`
@property (nonatomic, strong) UIView<XPTabBarRefreshViewAnimating> *refreshView;
/// 是否启用刷新动画, 默认`NO`
@property (nonatomic, assign, getter=isEnabledRefreshAnimation) BOOL enabledRefreshAnimation;

/// 停止刷新(请在`refreshBlock`回调中调用该方法)
- (void)stopRefresh;

@end


NS_CLASS_AVAILABLE_IOS(8_0) @interface UITabBar (XPTabBarRefresh)

@end


@protocol XPTabBarRefreshViewAnimating <NSObject>

/// 开始刷新动画
- (void)startRefreshAnimating;
/// 结束刷新动画
- (void)stopRefreshAnimating;

@end
```

- .m 文件

```ObjC
#import "UITabBar+XPTabBarRefresh.h"
#import <objc/runtime.h>

@interface UITabBarItem (XPTabBarRefreshPrivate)

/// 是否正在刷新
@property (nonatomic, assign, getter=isRefreshing) BOOL refreshing;

/// 开始刷新
- (void)startRefresh;

@end

@implementation UITabBarItem (XPTabBarRefreshPrivate)

- (void)startRefresh {
    self.refreshing = YES;
    if (!self.isEnabledRefreshAnimation) {
        return;
    }
    UIView *containerView = (UIView *)[self valueForKey:@"_view"];
    if (containerView == nil || ![containerView isKindOfClass:UIView.class]) {
        return;
    }
    for (UIView *subview in containerView.subviews) {
        subview.hidden = YES;
    }
    UIView<XPTabBarRefreshViewAnimating> *animatingView = self.refreshView;
    [animatingView setUserInteractionEnabled:NO];
    [animatingView setTranslatesAutoresizingMaskIntoConstraints:NO];
    [containerView addSubview:animatingView];
    NSDictionary *views = @{@"view" : animatingView};
    [NSLayoutConstraint activateConstraints:[NSLayoutConstraint constraintsWithVisualFormat:@"H:|[view]|" options:0 metrics:nil views:views]];
    [NSLayoutConstraint activateConstraints:[NSLayoutConstraint constraintsWithVisualFormat:@"V:|[view]|" options:0 metrics:nil views:views]];
    [animatingView startRefreshAnimating];
}

- (void)stopRefresh {
    self.refreshing = NO;
    if (!self.isEnabledRefreshAnimation) {
        return;
    }
    UIView<XPTabBarRefreshViewAnimating> *animatingView = self.refreshView;
    for (UIView *subview in animatingView.superview.subviews) {
        subview.hidden = NO;
    }
    [animatingView stopRefreshAnimating];
    [animatingView removeFromSuperview];
}

- (void)setRefreshBlock:(XPTabBarRefreshBlock)refreshBlock {
    objc_setAssociatedObject(self, @selector(refreshBlock), refreshBlock, OBJC_ASSOCIATION_COPY);
}

- (XPTabBarRefreshBlock)refreshBlock {
    return objc_getAssociatedObject(self, _cmd);
}

- (void)setRefreshView:(UIView<XPTabBarRefreshViewAnimating> *)refreshView {
    objc_setAssociatedObject(self, @selector(refreshView), refreshView, OBJC_ASSOCIATION_RETAIN);
}

- (UIView<XPTabBarRefreshViewAnimating> *)refreshView {
    UIView<XPTabBarRefreshViewAnimating> *view = objc_getAssociatedObject(self, _cmd);
    if (nil == view) {
        UIActivityIndicatorView *indicator = [[UIActivityIndicatorView alloc] initWithActivityIndicatorStyle:UIActivityIndicatorViewStyleGray];
        indicator.hidesWhenStopped = YES;
        view = (UIView<XPTabBarRefreshViewAnimating> *)indicator;
        [self setRefreshView:view];
    }
    return view;
}

- (void)setEnabledRefreshAnimation:(BOOL)enabledRefreshAnimation {
    objc_setAssociatedObject(self, @selector(isEnabledRefreshAnimation), @(enabledRefreshAnimation), OBJC_ASSOCIATION_RETAIN);
}

- (BOOL)isEnabledRefreshAnimation {
    return [objc_getAssociatedObject(self, _cmd) boolValue];
}

- (void)setRefreshing:(BOOL)refreshing {
    objc_setAssociatedObject(self, @selector(isRefreshing), @(refreshing), OBJC_ASSOCIATION_RETAIN);
}

- (BOOL)isRefreshing {
    return [objc_getAssociatedObject(self, _cmd) boolValue];
}

@end


#pragma mark -

@implementation UITabBar (XPTabBarRefresh)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        SEL originalSelector = NSSelectorFromString(@"_buttonUp:");
        SEL swizllingSelector = @selector(xp_buttonUp:);
        Method originalMethod = class_getInstanceMethod(self, originalSelector);
        Method swizlledMethod = class_getInstanceMethod(self, swizllingSelector);
        BOOL flag = class_addMethod(self, originalSelector, method_getImplementation(swizlledMethod), method_getTypeEncoding(swizlledMethod));
        if (flag) {
            class_replaceMethod(self, swizllingSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizlledMethod);
        }
    });
}

- (void)xp_buttonUp:(UIControl *)sender {
    BOOL selected = [[sender valueForKey:@"_selected"] boolValue];
    if (selected) {
        UITabBarItem *item = self.selectedItem;
        XPTabBarRefreshBlock refreshBlock = [item refreshBlock];
        if (refreshBlock && !item.isRefreshing) {
            [item startRefresh];
            refreshBlock(self, item);
            return;
        }
    }
    [self xp_buttonUp:sender];
}

@end


#pragma mark -

@interface UIActivityIndicatorView (XPTabBarRefresh)<XPTabBarRefreshViewAnimating>

@end

@implementation UIActivityIndicatorView (XPTabBarRefresh)

- (void)startRefreshAnimating {
    [self startAnimating];
}

- (void)stopRefreshAnimating {
    [self stopAnimating];
}

@end
```


最后提一下，除了 runtime 之外，也可以通过 UITabBarControllerDelegate 代理实现，可以从 `- (BOOL)tabBarController:(UITabBarController *)tabBarController shouldSelectViewController:(UIViewController *)viewController` 和 `- (void)tabBarController:(UITabBarController *)tabBarController didSelectViewController:(UIViewController *)viewController` 这两个方法下手。

网上随便一搜都是基于 `- (void)tabBarController:(UITabBarController *)tabBarController didSelectViewController:(UIViewController *)viewController`  方法的实现，这里就不再累赘了。