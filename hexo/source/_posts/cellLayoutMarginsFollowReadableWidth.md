---
title: UITableView在iPad iOS9下cell宽度显示不全
tags:
  - iOS
abbrlink: 8999626b
date: 2017-03-22 17:22:13
---

UITableView在iOS9后新增了一个属性`cellLayoutMarginsFollowReadableWidth`，该属性会影响UITableViewCell在iPad下的显示，且该属性默认为`YES`。

当你发现在iPad下UITableViewCell不能占满整个UITableView的宽度时，只需将该属性设置为NO即可。

``` ObjC
[self.tableView setCellLayoutMarginsFollowReadableWidth:NO];
```
