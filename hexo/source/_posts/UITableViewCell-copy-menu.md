---
title: UITableViewCell长按拷贝菜单选项
tags:
  - iOS
abbrlink: 8e5ab17e
date: 2017-10-26 09:14:05
---

> 运营推广人员向我们反馈说需要一个能方便拷贝会员手机号码的功能，方便进行其他操作。由于会员信息是采用UITableView来进行展示的，而系统已经为我们准备好了UITableViewCell长按功能菜单的API，我们只需实现对应的代理方法即可。

要实现该功能，主要步骤为：

- 遵守`UITableViewDelegate`协议；
- 实现`- (BOOL)tableView:(UITableView *)tableView shouldShowMenuForRowAtIndexPath:(NSIndexPath *)indexPath`方法并返回YES；
- 实现`- (BOOL)tableView:(UITableView *)tableView canPerformAction:(SEL)action forRowAtIndexPath:(NSIndexPath *)indexPath withSender:(nullable id)sender`方法，根据action决定对应的菜单按钮是否需要显示；
- 实现`- (void)tableView:(UITableView *)tableView performAction:(SEL)action forRowAtIndexPath:(NSIndexPath *)indexPath withSender:(nullable id)sender`方法，根据action判断用户做出了什么操作。

具体实现代码：
``` ObjC
- (BOOL)tableView:(UITableView *)tableView shouldShowMenuForRowAtIndexPath:(NSIndexPath *)indexPath {
    return (indexPath.section == 1 && indexPath.row == 0);
}

- (BOOL)tableView:(UITableView *)tableView canPerformAction:(SEL)action forRowAtIndexPath:(NSIndexPath *)indexPath withSender:(id)sender {
    if (indexPath.section == 1 && indexPath.row == 0 && action == @selector(copy:)) {
        return YES;
    }
    return NO;
}

- (void)tableView:(UITableView *)tableView performAction:(SEL)action forRowAtIndexPath:(NSIndexPath *)indexPath withSender:(id)sender {
    if (action == @selector(copy:)) {
        [[UIPasteboard generalPasteboard] setString:@"需要拷贝的内容"];
    }
}
```

最后来一张效果图:

![效果图](/images/201710/UITableViewCell-copy.png)
