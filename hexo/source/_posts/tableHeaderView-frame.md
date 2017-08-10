---
title: UITableView动态修改tableHeaderView的高度
date: 2017-08-10 16:16:36
tags:
	- iOS
---

有时候我们会用表格的头部视图来做一些简单的UI展示，如果遇到高度固定的情况还好，并无任何问题。但是如果遇到高度需要动态调整的时候，那就有点蛋疼了，因为你不能通过直接修改tableHeaderView的frame属性来达到效果。直接修改frame会导致UI有时候正常有时候又不正常，完全把控不了。

那么我们该如何解决这个问题呢，主要有以下几步：

1. 先取出UITableView的tableHeaderView

2. 将tableHeaderView从父视图移除并将UITableView的tableHeaderView设置为nil

3. 更新tableHeaderView的frame

4. 重新给UITableView的tableHeaderView赋值


演示代码如下：

```ObjC
UIView *tableHeaderView = self.tableView.tableHeaderView;
CGRect frame = tableHeaderView.frame;
[tableHeaderView removeFromSuperview];
self.tableView.tableHeaderView = nil;
frame.size.height = 100.0; // 新高度
tableHeaderView.frame = frame;
self.tableView.tableHeaderView = tableHeaderView;
```
