---
title: UICollectionView监听reloadData完成状态
tags:
  - iOS
abbrlink: 56cfe2af
date: 2017-07-14 10:42:39
---

> 如果你只想知道代码的正确使用姿势，请直接看文章底部的示例代码。

有时候我们需要在UICollectionView执行了reloadData方法后，监听reloadData的完成状态以便执行某些回调操作。如果我们紧跟着reloadData代码，在其后面继续编写我们的回调操作，这个时候你会发现程序的运行结果与我们的预期有出入，并非是我们所期望的结果。

最近在撸一个图片轮播的功能，用UICollectionView来做，因为UICollectionView本身就支持横向滚动，并且有很好的重用机制，也不用我们自己计算上一页下一页然后将视图挪来挪去的。由于要做一个无限滚动的功能，所以做法就是在`- (NSInteger)collectionView:numberOfItemsInSection:`方法中返回`图片个数x基数(如:100)`的总数，然后将UICollectionView滚动至第`图片个数x基数x0.5`个Cell，这样便可以左右无限滑动了，问题的关键就在于如何在realoadData完成之后滚动到指定的IndexPath上。如果你直接像下面这样写代码，你会惊讶地发现，UICollectionView并没有滚动到指定的IndexPath，而是仍然显示第一个Cell。

```ObjC
[collectionView reloadData];
// 在这里继续编写你的特定代码
NSIndexPath *indexPath = [NSIndexPath indexPathForItem:100 inSection:0];
[collectionView scrollToItemAtIndexPath:indexPath atScrollPosition:UICollectionViewScrollPositionNone animated:NO];
```


UICollectionView有一个方法`-performBatchUpdates:completion:`，其定义如下：

```ObjC
- (void)performBatchUpdates:(void (^ __nullable)(void))updates completion:(void (^ __nullable)(BOOL finished))completion; // allows multiple insert/delete/reload/move calls to be animated simultaneously. Nestable.
```

根据方法名和注释很容易明白这个接口是用来批量操作更新UICollectionView的，把你需要插入/删除/重新加载/移动UICollectionViewCell的代码统统放到`updates`这个block中，并且在当这些更新操作完成时系统会回调`completion`这个block。


看到这里，你是不是很激动，是不是已经想到该如何写代码了，是不是像下面这样:

```ObjC
// 错误的使用姿势
[collectionView performBatchUpdates:^{
    [collectionView reloadData];
} completion:^(BOOL finished) {
    NSLog(@"*************reloadData执行完毕*********");
}];

```

如果你是这样子写的代码，很好，和我想法一致，虽然程序`有可能`运行很正常，为什么说有可能呢，因为当`[collectionView reloadData]`之后，如果展示的数据并没有发生改变，数据未增加也未减少，那么此时程序就能够正常运行；但是如果执行realodData后，数据源返回的数据有所改变，需要展示的数据条目增多或减少了，那么这个时候程序就会Crash了，并且控制台会打印如下的错误信息：

```ObjC
2017-07-14 11:22:57.054 Demo[9368:620226] *** Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: 'Invalid update: invalid number of sections.  The number of sections contained in the collection view after the update (0) must be equal to the number of sections contained in the collection view before the update (2), plus or minus the number of sections inserted or deleted (0 inserted, 0 deleted).'
*** First throw call stack:
(
	0   CoreFoundation                      0x0000000109a9fb0b __exceptionPreprocess + 171
	1   libobjc.A.dylib                     0x000000010c770141 objc_exception_throw + 48
	2   CoreFoundation                      0x0000000109aa3cf2 +[NSException raise:format:arguments:] + 98
	3   Foundation                          0x000000010a9f13b6 -[NSAssertionHandler handleFailureInMethod:object:file:lineNumber:description:] + 193
	4   UIKit                               0x000000010dec753d -[UICollectionView _endItemAnimationsWithInvalidationContext:tentativelyForReordering:animator:] + 15364
	5   UIKit                               0x000000010ded0259 -[UICollectionView _endUpdatesWithInvalidationContext:tentativelyForReordering:animator:] + 71
	6   UIKit                               0x000000010ded05a0 -[UICollectionView _performBatchUpdates:completion:invalidationContext:tentativelyForReordering:animator:] + 437
	7   UIKit                               0x000000010ded03c8 -[UICollectionView _performBatchUpdates:completion:invalidationContext:tentativelyForReordering:] + 91
	8   UIKit                               0x000000010ded034a -[UICollectionView _performBatchUpdates:completion:invalidationContext:] + 74
	9   UIKit                               0x000000010ded029f -[UICollectionView performBatchUpdates:completion:] + 53
	10  VBemall                             0x00000001082da642 __31-[VBHomeController viewDidLoad]_block_invoke + 194
	11  libdispatch.dylib                   0x000000010f6aa05c _dispatch_client_callout + 8
	12  libdispatch.dylib                   0x000000010f686c6e _dispatch_continuation_pop + 1020
	13  libdispatch.dylib                   0x000000010f69b9fc _dispatch_source_latch_and_call + 230
	14  libdispatch.dylib                   0x000000010f6944f1 _dispatch_source_invoke + 1167
	15  libdispatch.dylib                   0x000000010f68b69a _dispatch_main_queue_callback_4CF + 1066
	16  CoreFoundation                      0x0000000109a64909 __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__ + 9
	17  CoreFoundation                      0x0000000109a2aae4 __CFRunLoopRun + 2164
	18  CoreFoundation                      0x0000000109a2a016 CFRunLoopRunSpecific + 406
	19  GraphicsServices                    0x0000000111ae5a24 GSEventRunModal + 62
	20  UIKit                               0x000000010d57c0d4 UIApplicationMain + 159
	21  VBemall                             0x00000001082da1ef main + 111
	22  libdyld.dylib                       0x000000010f6f665d start + 1
)
libc++abi.dylib: terminating with uncaught exception of type NSException
```

看来是不能这样子用了，不能直接在`updates`这个block中执行reloadData方法。

然后我尝试了将reloadData放到block外，发现能够程序能够正常按照我们的预期运行了，代码如下:

```ObjC
// 代码的正确使用姿势
[collectionView reloadData];
[collectionView performBatchUpdates:nil completion:^(BOOL finished) {
	NSIndexPath *indexPath = [NSIndexPath indexPathForItem:100 inSection:0];
	[collectionView scrollToItemAtIndexPath:indexPath atScrollPosition:UICollectionViewScrollPositionNone animated:NO];
}];

```

ps: UITableView在iOS11也引入了`-performBatchUpdates:completion:`这个接口，使用上应该也是和UICollectionView一样一样的。
