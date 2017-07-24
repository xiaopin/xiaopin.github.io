---
title: iOS截屏分享
date: 2017-07-24 10:36:49
tags:
	- iOS
---

最近在把玩`支付宝`和`小米商城`App时发现了个好玩的东西，在支付宝首页中截屏，则会在屏幕右侧弹出一个小窗口来展示首页截屏图片，点击图片就会进入求助反馈页面，再点击意见反馈，截屏的图片就会自动被选上作为反馈图片使用；而小米商城的做法则更好玩，在任意页面中进行截屏，则会在屏幕右下角中弹出一个`分享截图给好友`的提示框，点击则会弹出分享页面，在此可以看到我们刚才截取的屏幕图片，接下来你就可以把截屏图片分享给你的微信朋友、QQ好友、发到朋友圈、发微博等等，任君选择。


查了下资料，发现苹果在iOS7中新增了`UIApplicationUserDidTakeScreenshotNotification`通知，只要注册了这个通知，在用户截屏的时候App就会收到截屏的通知，接下来就可以做你想做的事情了。

```ObjC
// This notification is posted after the user takes a screenshot (for example by pressing both the home and lock screen buttons)
UIKIT_EXTERN NSNotificationName const UIApplicationUserDidTakeScreenshotNotification NS_AVAILABLE_IOS(7_0);
```

下面我们自己来撸一个自己的截屏分享功能。打开首页控制器，重写viewDidAppear和viewDidDisappear方法。因为我们只需要在首页中进行截屏分享，在其他页面截屏时不处理，所以我们在viewDidAppear方法中注册通知并且在viewDidDisappear方法中移除通知。

示例中通过`[UIApplication sharedApplication].keyWindow`来获取截屏图片，因为如果App中存在导航栏/TabBar的情况下，只有通过keyWindow生成的图片才能包含导航栏/TabBar，如果你仅仅想分享首页内容，不包含导航栏/TabBar的话，那你也可以直接用首页控制器的`self.view`来生成截屏图片，甚至如果你想分享一个长图片，如:首页是一个滚动视图，那么你完全可以通过滚动视图来生成一个长图片然后进行分享，这个完全可以根据你的需求来决定。

> 自己生成的截屏图片有一点很遗憾，就是生成的图片并不能包含顶部状态栏内容，支付宝和小米商城都有这个问题，这个应该是没辙了。或许只能去系统相册拿图片了，这个你就自己去测试吧。

```ObjC
- (void)viewDidAppear:(BOOL)animated {
    [super viewDidAppear:animated];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(userDidTakeScreenshotNotification:) name:UIApplicationUserDidTakeScreenshotNotification object:nil];
}

- (void)viewDidDisappear:(BOOL)animated {
    [super viewDidDisappear:animated];
    [[NSNotificationCenter defaultCenter] removeObserver:self name:UIApplicationUserDidTakeScreenshotNotification object:nil];
}

- (void)userDidTakeScreenshotNotification:(NSNotification *)sender {
    UIWindow *window = [UIApplication sharedApplication].keyWindow;
    UIImage *screenshotImage = [UIImage snapshotImageWithView:window];
    // TODO: 将screenshotImage进行分享，可以调用友盟SDK或自己集成第三方SDK实现，这里就不做演示了
}
```

下面附上从UIView获取图片的代码:

Objectie-C版本

```ObjC
/**
 *  通过视图获取视图快照
 *
 *  @param view 需要生成快照的视图
 *
 *  @return 快照图片
 */
+ (instancetype)snapshotImageWithView:(UIView * __nonnull)view {
    if (nil == view) {
        return nil;
    }
    UIGraphicsBeginImageContextWithOptions(view.bounds.size, YES, [UIScreen mainScreen].scale);
    CGContextRef context = UIGraphicsGetCurrentContext();
    [view.layer renderInContext:context];
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    return image;
}
```

Swift版本

```Swift
extension UIImage {

    /// 获取视图快照图片
    /// 必须在主线程上执行
    ///
    /// - Parameter view: 需要生成快照的视图
    /// - Returns: 快照图片
    static func snapshot(_ view: UIView) -> UIImage? {
        if !Thread.isMainThread {
            assert(false, "UIImage.\(#function) must be called from main thread only.")
        }
        UIGraphicsBeginImageContextWithOptions(view.bounds.size, true, UIScreen.main.scale)
        defer { UIGraphicsEndImageContext() }
        guard let context = UIGraphicsGetCurrentContext() else { return nil }
        view.layer.render(in: context)
        return UIGraphicsGetImageFromCurrentImageContext()
    }

}
```
