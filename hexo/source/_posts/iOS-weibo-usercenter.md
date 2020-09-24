---
title: iOS仿新浪微博个人页面
tags:
  - iOS
abbrlink: dcdf803f
date: 2018-02-08 14:02:01
---

## 代码

Demo地址：[https://github.com/xiaopin/weibo-UserCenter.git](https://github.com/xiaopin/weibo-UserCenter.git)

该Demo中并没有实现导航栏透明效果，也没搞下拉背景放大的效果，想研究这些的可以出门左拐去问度娘了。


## 效果图

![GIF](/images/201802/weibo-preview.gif)


## UI结构说明

如下图所示，我把UI分为三大部分：

![UI说明](/images/201802/weibo-UI-description.png)

完整的UI层级结构树：

```
UserCenterViewController.view
│
└── UserCenterScrollView
	│
	├── UserCenterHeaderView
	│
	├── UserCenterSegmentedView
	│
	└── UIPageViewController.view
		│
		├── HomeViewController.view
		│	│
		│	└── UITableView
		│
		├── WeiboListViewController.view
		│	│
		│	└── UITableView
		│
		└── AlbumViewController.view
			│
			└── UICollectionView
```


## 思路

我采用的是UIScrollView+UIPageViewController来实现的，UIScrollView作为最底层的容器视图，管理着头部视图、分段按钮视图、内容视图(实际上是UIPageViewController的view，由UIPageViewController来实现左右滑动切换内容视图控制器)。

从上面的结构中也可以看出，实际上是UIScrollView嵌套UIScrollView的范畴了，这样就涉及到了滚动手势的处理了。
我是这样处理的，重写了UserCenterScrollView的`func gestureRecognizer(_ gestureRecognizer: UIGestureRecognizer, shouldRecognizeSimultaneouslyWith otherGestureRecognizer: UIGestureRecognizer) -> Bool`方法，并通过返回true来同时使UserCenterScrollView和内容视图中的滚动视图可以同时接收到滚动手势事件，然后在UserCenterViewController定义一个标记记录UserCenterScrollView是否可以滚动，在UserCenterContentProxyViewController中也定义了一个标记用于记录内容视图是否可以滚动。通过这两个标记的配合使用来达到上面的效果。

具体实现还是看代码去吧，一行代码胜于千言万语。