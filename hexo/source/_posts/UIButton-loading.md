---
title: UIButton 被禁用时显示一个菊花提示用户正在加载中
date: 2018-09-14 10:51:19
tags:
	- iOS
---

当点击按钮后，可能会执行一个比较耗时的操作，但是又不方便在整个页面进行提示时，此时如果我们仅仅只是把按钮禁用了以防用户重复点击，但是用户并不知道操作是否还在继续，用户体验不够友好；但是如果我们在按钮被禁用时，自动显示一个转圈圈的小菊花进行提示，那用户体验就会好很多，用户一看就知道当前操作正在进行中。

## 演示

附上我项目中的一个使用场景：重新定位的按钮，点击后重新定位用户的位置，但是不影响页面的其他操作。

![gif](/images/201809/button-loading.gif)

## 要求

- iOS 9.0+ (使用了 `NSLayoutAnchor`, 如需要兼容低版本 iOS, 请自行改用 NSLayoutConstraint)

## 源码

新建一个 UIButton 的分类

- `.h` 头文件

```ObjC
/// 当按钮被禁用时, 是否显示正在加载的状态提示, 默认`NO`
@property (nonatomic, assign, getter=isShowLoadingWhenDisabled) IBInspectable BOOL showLoadingWhenDisabled;
```

- `.m` 文件

```ObjC

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        SEL originalSelector = @selector(setEnabled:);
        SEL swizllingSelector = @selector(xp_setEnabled:);
        Method originalMethod = class_getInstanceMethod(self, originalSelector);
        Method swizllingMethod = class_getInstanceMethod(self, swizllingSelector);
        BOOL flag = class_addMethod(self, originalSelector, method_getImplementation(swizllingMethod), method_getTypeEncoding(swizllingMethod));
        if (flag) {
            class_replaceMethod(self, swizllingSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizllingMethod);
        }
    });
}

static char const XPButtonLoadingIndicatorKey       = '\0';

/// 重写 `setEnabled:` 方法, 当按钮被禁用时显示一个菊花提示
- (void)xp_setEnabled:(BOOL)enabled {
    if (self.isShowLoadingWhenDisabled) {
        if (enabled) {
            UIActivityIndicatorView *indicatorView = objc_getAssociatedObject(self, &XPButtonLoadingIndicatorKey);
            [indicatorView stopAnimating];
        } else {
            // 显示菊花
            UIActivityIndicatorView *indicatorView = objc_getAssociatedObject(self, &XPButtonLoadingIndicatorKey);
            if (nil == indicatorView) {
                indicatorView = [[UIActivityIndicatorView alloc] initWithActivityIndicatorStyle:UIActivityIndicatorViewStyleGray];
                [indicatorView setTranslatesAutoresizingMaskIntoConstraints:NO];
                [indicatorView setHidesWhenStopped:YES];
                [self addSubview:indicatorView];
                [indicatorView.centerXAnchor constraintEqualToAnchor:self.centerXAnchor].active = YES;
                [indicatorView.centerYAnchor constraintEqualToAnchor:self.centerYAnchor].active = YES;
                objc_setAssociatedObject(self, &XPButtonLoadingIndicatorKey, indicatorView, OBJC_ASSOCIATION_RETAIN_NONATOMIC);

                // 清除禁用状态下的图片/文字(设置`hidden`属性无效)
                CGRect rect = (CGRect){CGPointZero, CGSizeMake(1.0, 1.0)};
                UIGraphicsBeginImageContext(rect.size);
                CGContextRef context = UIGraphicsGetCurrentContext();
                CGContextSetFillColorWithColor(context, UIColor.clearColor.CGColor);
                CGContextFillRect(context, rect);
                UIImage *clearImage = UIGraphicsGetImageFromCurrentImageContext();
                UIGraphicsEndImageContext();
                [self setImage:clearImage forState:UIControlStateDisabled];
                [self setBackgroundImage:clearImage forState:UIControlStateDisabled];
                [self setTitle:NSString.new forState:UIControlStateDisabled];
            }
            [indicatorView startAnimating];
        }
    }
    [self xp_setEnabled:enabled];
}

- (void)setShowLoadingWhenDisabled:(BOOL)showLoadingWhenDisabled {
    objc_setAssociatedObject(self, @selector(isShowLoadingWhenDisabled), @(showLoadingWhenDisabled), OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    if (showLoadingWhenDisabled && self.state == UIControlStateDisabled) {
        [self setEnabled:self.isEnabled];
    }
}

- (BOOL)isShowLoadingWhenDisabled {
    return [objc_getAssociatedObject(self, _cmd) boolValue];
}

```

## 用法

1、 设置按钮的 `showLoadingWhenDisabled` 属性为 `YES`

2、 设置按钮的启用/禁用 `setEnabled:`


## 注意

- 不建议将按钮默认为禁用状态