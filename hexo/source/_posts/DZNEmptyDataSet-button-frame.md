---
title: 修改DZNEmptyDataSet按钮的宽高
tags:
  - iOS
abbrlink: 912c0fee
date: 2018-06-03 16:00:23
---

相信没有哪款App是没有使用到滚动视图(UIScrollView、UITableView、UICollectionView)的吧，我相信大家用的最多的就是UITableView了，当需要展示列表数据时，我想大家第一次时间想到的就是UITableView。

如果数据不为空的时候还好，可谁也不能保证一定会有数据展示吧，如果没数据的时候，就会给客户呈现一个空白页面（白板），此时我们可以给用户一点提示信息（图片、文字），这不仅可以提升用户体验，也不至于让用户一脸懵逼。

[DZNEmptyDataSet](https://github.com/dzenbot/DZNEmptyDataSet.git) 就是为此而生的，只需设置好数据源和代理即可，然后根据你的需求返回对应的图片或者提示文字。

但是 DZNEmptyDataSet 提供的按钮默认不能调整宽高，如果你实现了该数据源方法 `- (UIImage *)buttonBackgroundImageForEmptyDataSet:(UIScrollView *)scrollView forState:(UIControlState)state` 并返回对应按钮背景图片后，DZNEmptyDataSet 会将按钮高度自动调整为图片的高度；即便如此，还是无法调整宽度。

由于 DZNEmptyDataSet 并没有提供相应的接口给我们设置按钮的宽高，那我们只能通过曲线救国的方式来达到目的了。

遵守 `DZNEmptyDataSetDelegate` 协议，实现 `- (void)emptyDataSetDidAppear:(UIScrollView *)scrollView` 方法，然后修改按钮的约束。

> 不要在 `- (void)emptyDataSetWillAppear:(UIScrollView *)scrollView` 这个方法里设置。宽度只能通过调整 `leading` 和 `trailing` 的约束来达到效果。

```ObjC
- (void)emptyDataSetDidAppear:(UIScrollView *)scrollView {
    UIButton *button = [scrollView valueForKeyPath:@"emptyDataSetView.button"];
    if ([button isKindOfClass:[UIButton class]]) {
        // Change button width
        for (NSLayoutConstraint *constraint in button.superview.constraints) {
            if (constraint.firstItem == button && constraint.firstAttribute == NSLayoutAttributeLeading) {
                constraint.constant = 130.0;
            } else if (constraint.secondItem == button && constraint.secondAttribute == NSLayoutAttributeTrailing) {
                constraint.constant = 130.0;
            }
        }
        // Change button height
        for (NSLayoutConstraint *constraint in button.constraints) {
            if (constraint.firstItem == button && constraint.firstAttribute == NSLayoutAttributeHeight) {
                constraint.constant = 40.0;
            }
        }
    }
}
```