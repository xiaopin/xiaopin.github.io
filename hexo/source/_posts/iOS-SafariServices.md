---
title: SafariServices框架
tags:
  - iOS
abbrlink: 652a77d
date: 2018-03-14 15:46:51
---

相信大家在日常开发中，都离不开web吧，部分页面还是得用网页进行展示。在iOS7和之前我们可以用UIWebView来加载网页，在iOS8 Apple为我们带来了WKWebView(WebKit框架)，而在iOS9 Apple为我们提供了进一步的封装库SafariServices。

SafariServices的可定制性很差，可以说毫无定制性可言，我们只能控制一下导航栏和底部工具栏的颜色，以及通过SFSafariViewControllerDelegate可以添加自定的分享功能(UIActivity)。

### ***SFSafariViewController***

一个特殊的UIViewController，为我们使用Safari的UI框架加载网页内容，同时还能享受到Safari的一些便利性：
- 高度一致的用户体验性
- 和Safari共享Cookie
- 密码、证书自动填充
- Safari阅读器(可选是否默认进入阅读器模式)

```ObjC
NSURL *url = [NSURL URLWithString:@"http://www.0daybug.com"];
SFSafariViewController *safariVC;
if (@available(iOS 11.0, *)) {
    SFSafariViewControllerConfiguration *configuration = [[SFSafariViewControllerConfiguration alloc] init];
    configuration.entersReaderIfAvailable = NO;
    configuration.barCollapsingEnabled = YES;
    safariVC = [[SFSafariViewController alloc] initWithURL:url configuration:configuration];
} else {
    safariVC = [[SFSafariViewController alloc] initWithURL:url];
}
if (@available(iOS 11.0, *)) {
    safariVC.dismissButtonStyle = SFSafariViewControllerDismissButtonStyleClose;
}
if (@available(iOS 10.0, *)) {
    safariVC.preferredBarTintColor = [UIColor orangeColor];
    safariVC.preferredControlTintColor = [UIColor redColor];
}
// [self.navigationController pushViewController:safariVC animated:true];
[self presentViewController:safariVC animated:true completion:nil];
```

SFSafariViewController在push和present上有些许区别。
- push方式
	1. 点击左上角的按钮并不会关闭网页，需要自行处理
	2. 不支持屏幕左侧滑动返回上一页的手势
- present方式
	1. 点击左上角按钮会关闭网页返回上一个页面
	2. 支持屏幕左侧滑动返回上一页的手势


### ***SFSafariViewControllerConfiguration***

iOS11新增的配置类，目前的可配置选项仅有两个：
- entersReaderIfAvailable
	1. 在当前网页支持阅读器时是否自动进入阅读器模式浏览网页
	2. 默认NO
- barCollapsingEnabled
	1. 当拖动网页进行滚动时，是否折叠导航栏和隐藏底部工具栏
	2. 默认YES

### ***SSReadingList***

添加URL链接到Safari浏览器的阅读列表。需要注意一点的是相同的URL可以添加多次，系统并不会过滤相同的URL。

```ObjC
NSURL *url = [NSURL URLWithString:@"http://www.0daybug.com"];
SSReadingList *readingList = [SSReadingList defaultReadingList];
[readingList addReadingListItemWithURL:url title:@"我的博客" previewText:@"记录日常开发生涯中的点滴" error:nil];
```

### ***SFContentBlockerManager***

Content Blocker，据说是一个Browser Extension之类的玩意，和广告过滤的作用类似，目前功能不明，更多信息可以看[这里](https://webkit.org/blog/3476/content-blockers-first-look/)。

### 效果演示

![GIF](/images/201802/SafariServices.gif)
