---
title: UIColor转UIImage
date: 2017-12-14 17:13:23
tags:
	- iOS
---

没啥好说的，直接看代码

⬇⬇⬇⬇⬇⬇⬇⬇⬇⬇

### .h头文件

```ObjC
@interface UIImage (Color)

/**
 *  将UIColor转换为UIImage且大小为1x1
 *
 *  @param color 图片颜色
 *
 *  @return UIImage
 */
+ (nullable instancetype)imageWithColor:(UIColor * _Nonnull)color;

/**
 *  将UIColor转换为UIImage并指定大小
 *
 *  @param color 图片颜色
 *  @param size  图片大小
 *
 *  @return UIImage
 */
+ (nullable instancetype)imageWithColor:(UIColor * _Nonnull)color size:(CGSize)size;

@end

```


### .m文件

```ObjC
@implementation UIImage (Color)

/**
 *  将UIColor转换为UIImage且大小为1x1
 *
 *  @param color 图片颜色
 *
 *  @return UIImage
 */
+ (instancetype)imageWithColor:(UIColor *)color {
    return [self imageWithColor:color size:CGSizeMake(1.0, 1.0)];
}

/**
 *  将UIColor转换为UIImage并指定大小
 *
 *  @param color 图片颜色
 *  @param size  图片大小
 *
 *  @return UIImage
 */
+ (instancetype)imageWithColor:(UIColor *)color size:(CGSize)size {
    UIGraphicsBeginImageContext(size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetFillColorWithColor(context, color.CGColor);
    CGContextFillRect(context, (CGRect){CGPointZero, size});
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return image;
}


@end
```