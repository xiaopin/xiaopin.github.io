---
title: UITableViewHeaderFooterView 修改背景色/文字
date: 2018-08-31 17:15:53
tags:
	- iOS
---

注意点：

- 修改背景色只能通过修改 `contentView` 或 `backgroundView` 的背景色实现，修改 UITableViewHeaderFooterView 自身背景色无效果
- 只能在 `tableView:willDisplayHeaderView:forSection:` 代理方法中进行设置，在 `tableView:didEndDisplayingHeaderView:forSection:` 设置无效

```ObjC
- (void)tableView:(UITableView *)tableView willDisplayHeaderView:(UIView *)view forSection:(NSInteger)section {
    UITableViewHeaderFooterView *headerView = (UITableViewHeaderFooterView *)view;
    headerView.contentView.backgroundColor = [UIColor brownColor];
    headerView.textLabel.font = [UIFont systemFontOfSize:14.0];
    headerView.textLabel.textColor = [UIColor orangeColor];
}
```
