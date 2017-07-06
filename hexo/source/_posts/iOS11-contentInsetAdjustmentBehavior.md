---
title: UIScrollView在iOS11新增的contentInsetAdjustmentBehavior属性
date: 2017-07-06 14:51:23
tags:
	- iOS
---

> 在iOS11中，苹果对UIScrollView新增了`contentInsetAdjustmentBehavior`属性来取代之前的UIViewController的`automaticallyAdjustsScrollViewInsets`属性。

在iOS11之前的项目中，如果我们需要将滚动视图的内容布局从屏幕左上角开始，也就是内容穿透UINavigationBar的效果，只需要设置UIViewController的`automaticallyAdjustsScrollViewInsets`属性为`NO`即可，告诉系统不要自动调整滚动视图的`contentInset`。但是在iOS11中，你将发现尽管将automaticallyAdjustsScrollViewInsets设置为NO了，但是滚动视图的内容还是往下偏移了64pt，从导航栏底部开始显示，通过查看automaticallyAdjustsScrollViewInsets的声明发现在iOS11中该属性已经被废弃以及被UIScrollView的`contentInsetAdjustmentBehavior`属性所替代了。

```ObjC
@property(nonatomic,assign) BOOL automaticallyAdjustsScrollViewInsets API_DEPRECATED_WITH_REPLACEMENT("Use UIScrollView's contentInsetAdjustmentBehavior instead", ios(7.0,11.0),tvos(7.0,11.0)); // Defaults to YES

```

我们再看下contentInsetAdjustmentBehavior的声明

```ObjC
@property(nonatomic) UIScrollViewContentInsetAdjustmentBehavior contentInsetAdjustmentBehavior API_AVAILABLE(ios(11.0),tvos(11.0));
```

contentInsetAdjustmentBehavior对应的是一个UIScrollViewContentInsetAdjustmentBehavior枚举类型的值，该枚举定义如下:

```ObjC
typedef NS_ENUM(NSInteger, UIScrollViewContentInsetAdjustmentBehavior) {
    UIScrollViewContentInsetAdjustmentAutomatic, // Similar to .scrollableAxes, but will also adjust the top & bottom contentInset when the scroll view is owned by a view controller with automaticallyAdjustsScrollViewContentInset = YES inside a navigation controller, regardless of whether the scroll view is scrollable
    UIScrollViewContentInsetAdjustmentScrollableAxes, // Edges for scrollable axes are adjusted (i.e., contentSize.width/height > frame.size.width/height or alwaysBounceHorizontal/Vertical = YES)
    UIScrollViewContentInsetAdjustmentNever, // contentInset is not adjusted
    UIScrollViewContentInsetAdjustmentAlways, // contentInset is always adjusted by the scroll view's safeAreaInsets
} API_AVAILABLE(ios(11.0),tvos(11.0));

```

通过查看UIScrollViewContentInsetAdjustmentBehavior的定义，我们很容易知道想要实现前面所说的效果，只需将滚动设置的contentInsetAdjustmentBehavior属性设置为对应的UIScrollViewContentInsetAdjustmentNever即可。

```ObjC
if (@available(iOS 11.0, *)) {
    scrollView.contentInsetAdjustmentBehavior = UIScrollViewContentInsetAdjustmentNever;
} else {
	self.automaticallyAdjustsScrollViewInsets = NO;
}
```

在iOS11之前，系统会通过动态调整UIScrollView的contentInset属性来实现视图内容的偏移，但是在iOS11中，系统不再调整UIScrollView的contentInset，而是通过`contentInsetAdjustmentBehavior`和`adjustedContentInset`来配合完成。
