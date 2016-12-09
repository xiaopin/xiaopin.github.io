---
title: iOS导航栏透明效果
date: 2016-12-09 17:04:15
tags:
	- iOS
---

废话不多说，直接上代码。

``` ObjC
// 去除导航栏底部黑色线条
UIImage *shadowImage = [UIImage pc_imageWithColor:[UIColor clearColor]];
[self.navigationController.navigationBar setShadowImage:shadowImage];
// 设置透明
[self setAutomaticallyAdjustsScrollViewInsets:NO];
UINavigationBar *navigationBar = self.navigationController.navigationBar;
[navigationBar setBarTintColor:[UIColor clearColor]];
[navigationBar setBackgroundImage:[UIImage pc_imageWithColor:[UIColor clearColor]]
                   forBarPosition:UIBarPositionAny
                       barMetrics:UIBarMetricsDefault];
```
### 效果图

这是iOS导航栏的默认效果，半透明。

![默认效果](/images/20161209/default.png)

这是将UINavigationBar的tintColor和背景图片设置为`[UIColor clearColor]`之后的效果，可以看到导航栏确实是变成完全透明的了，但是导航栏底部会有一个分割线。

![透明效果](/images/20161209/transparent.png)

这是将导航栏底部分割线也设置成`[UIColor clearColor]`之后的效果，可以看到，整个导航栏完全透明了，堪称`perfect`。

![透明效果](/images/20161209/transparent2.png)


### 附上将UIColor转成UIImage的代码

``` ObjC
@interface UIImage (PC)

+ (instancetype)pc_imageWithColor:(UIColor *)color;
+ (instancetype)pc_imageWithColor:(UIColor *)color size:(CGSize)size;

@end

@implementation UIImage (PC)

+ (instancetype)pc_imageWithColor:(UIColor *)color {
    return [self pc_imageWithColor:color size:CGSizeMake(1.0, 1.0)];
}

+ (instancetype)pc_imageWithColor:(UIColor *)color size:(CGSize)size {
    CGRect rect = (CGRect){CGPointZero, size};
    UIGraphicsBeginImageContext(rect.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetFillColorWithColor(context, [color CGColor]);
    CGContextFillRect(context, rect);
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return image;
}

@end
```
