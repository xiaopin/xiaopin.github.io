---
title: iOS 导航栏颜色渐变
tags:
  - iOS
abbrlink: fde6c29a
date: 2018-09-01 09:14:07
---

## 效果图

![PNG](/images/201809/gradient-navigation-bar.png)

## 代码

- 渐变的导航栏

```ObjC
@interface XPGradientNavigationBar : UINavigationBar
{
    CAGradientLayer *_gradientLayer;
}
@end

@implementation XPGradientNavigationBar

#pragma mark - Lifecycle

- (instancetype)initWithFrame:(CGRect)frame {
    if (self = [super initWithFrame:frame]) {
        [self commonInit];
    }
    return self;
}

- (void)awakeFromNib {
    [super awakeFromNib];
    [self commonInit];
}

- (void)layoutSubviews {
    [super layoutSubviews];

    if (!_gradientLayer.superlayer) {
        UIView *barBackgroundView = self.subviews.firstObject;
        if (@available(iOS 10.0, *)) {
            UIView *backgroundEffectView = [barBackgroundView valueForKey:@"_backgroundEffectView"];
            UIView *gradientView = backgroundEffectView.subviews.lastObject;
            [gradientView.layer addSublayer:_gradientLayer];
        } else {
            for (UIView *subview in barBackgroundView.subviews) {
                if ([subview isKindOfClass:NSClassFromString(@"_UIBackdropView")]) {
                    [subview.layer addSublayer:_gradientLayer];
                    break;
                }
            }
        }
    }

    if (_gradientLayer.superlayer) {
        _gradientLayer.frame = self.subviews.firstObject.bounds;
    }
}

#pragma mark - Private

- (void)commonInit {
    self.translucent = YES;
    _gradientLayer = [CAGradientLayer layer];
    _gradientLayer.colors = @[
                              (__bridge id)UIColor.purpleColor.CGColor,
                              (__bridge id)UIColor.orangeColor.CGColor
                              ];
    _gradientLayer.locations = @[@0.0, @1.0];
    _gradientLayer.startPoint = CGPointMake(0.0, 0.5);
    _gradientLayer.endPoint = CGPointMake(1.0, 0.5);
}

#pragma mark - setter & getter

- (void)setTranslucent:(BOOL)translucent {
    [super setTranslucent:YES];
}

@end
```

- 配合 UINavigationController 使用效果更佳 (非必须)

```ObjC
@implementation XPGradientNavigationController

- (instancetype)initWithRootViewController:(UIViewController *)rootViewController {
    self = [self initWithNavigationBarClass:nil toolbarClass:nil];
    if (self) {
        if (rootViewController) {
            [super pushViewController:rootViewController animated:NO];
        }
    }
    return self;
}

- (instancetype)initWithNavigationBarClass:(Class)navigationBarClass toolbarClass:(Class)toolbarClass {
    return [super initWithNavigationBarClass:XPGradientNavigationBar.class toolbarClass:toolbarClass];
}

- (instancetype)initWithCoder:(NSCoder *)aDecoder {
    if (self = [super initWithCoder:aDecoder]) {
        NSAssert([self.navigationBar isKindOfClass:XPGradientNavigationBar.class], @"self.navigationBar must be XPGradientNavigationBar");
    }
    return self;
}

@end
```

如果你是通过可视化方式创建的导航栏控制器，则需要在 Storyboard/xib 中手动将 `Navigation Bar` 的 Class 指定为 `XPGradientNavigationBar`，如果是通过代码创建 UINavigationController，则使用 `XPGradientNavigationController` 即可。
