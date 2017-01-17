---
title: 通过VFL(Visual Format Language)使用AutoLayout
date: 2016-12-02 21:26:47
tags:
	- iOS
---

Apple官网关于VFL的描述:[https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage.html#//apple_ref/doc/uid/TP40010853-CH27-SW1](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage.html#//apple_ref/doc/uid/TP40010853-CH27-SW1)

如果需要使用AutoLayout，那么必须将视图的`translatesAutoresizingMaskIntoConstraints`属性设置为`NO`，否则约束不会生效，并且控制器会打印一堆关于`[LayoutConstraints] Unable to simultaneously satisfy constraints.`的错误。


### 示例一: 单个view布局,左右距离父视图8pt,顶部距离父视图20pt,高度等于40pt

```ObjC
UIView *orangeView = [[UIView alloc] init];
[orangeView setBackgroundColor:[UIColor orangeColor]];
[self.view addSubview:orangeView];

[orangeView setTranslatesAutoresizingMaskIntoConstraints:NO];
NSArray *array1 = [NSLayoutConstraint constraintsWithVisualFormat:@"H:|-[orangeView]-|" options:0 metrics:nil views:NSDictionaryOfVariableBindings(orangeView)];
NSArray *array2 = [NSLayoutConstraint constraintsWithVisualFormat:@"V:|-20-[orangeView(==40)]" options:0 metrics:nil views:NSDictionaryOfVariableBindings(orangeView)];
[self.view addConstraints:array1];
[self.view addConstraints:array2];
```

### 示例二: 两个视图垂直布局,两个视图左右均距离父视图0pt,高度为40pt,orangeView顶部距离父视图20pt,purpleView顶部距离orangeView的底部8pt

```ObjC
UIView *orangeView = [[UIView alloc] init];
[orangeView setBackgroundColor:[UIColor orangeColor]];
[self.view addSubview:orangeView];

UIView *purpleView = [[UIView alloc] init];
[purpleView setBackgroundColor:[UIColor purpleColor]];
[self.view addSubview:purpleView];

NSMutableArray *constraints = [NSMutableArray array];

[orangeView setTranslatesAutoresizingMaskIntoConstraints:NO];
[constraints addObjectsFromArray:[NSLayoutConstraint constraintsWithVisualFormat:@"H:|[orangeView]|" options:NSLayoutFormatDirectionLeadingToTrailing metrics:nil views:NSDictionaryOfVariableBindings(orangeView)]];
[constraints addObjectsFromArray:[NSLayoutConstraint constraintsWithVisualFormat:@"V:|-20-[orangeView(==40)]" options:0 metrics:nil views:NSDictionaryOfVariableBindings(orangeView)]];

[purpleView setTranslatesAutoresizingMaskIntoConstraints:NO];
[constraints addObjectsFromArray:[NSLayoutConstraint constraintsWithVisualFormat:@"H:|[purpleView]|" options:NSLayoutFormatDirectionLeadingToTrailing metrics:nil views:NSDictionaryOfVariableBindings(purpleView)]];
[constraints addObjectsFromArray:[NSLayoutConstraint constraintsWithVisualFormat:@"V:[orangeView]-[purpleView(==orangeView)]" options:NSLayoutFormatDirectionLeadingToTrailing metrics:nil views:NSDictionaryOfVariableBindings(orangeView, purpleView)]];

[self.view addConstraints:constraints];
```

### 示例三: 两个视图水平局部,两个视图等宽等高,且间距相等

```ObjC
UIView *orangeView = [[UIView alloc] init];
[orangeView setBackgroundColor:[UIColor orangeColor]];
[self.view addSubview:orangeView];
UIView *purpleView = [[UIView alloc] init];
[purpleView setBackgroundColor:[UIColor purpleColor]];
[self.view addSubview:purpleView];

[orangeView setTranslatesAutoresizingMaskIntoConstraints:NO];
[purpleView setTranslatesAutoresizingMaskIntoConstraints:NO];
NSMutableArray *constraints = [NSMutableArray array];
[constraints addObjectsFromArray:[NSLayoutConstraint constraintsWithVisualFormat:@"H:|-margin-[orangeView(purpleView)]-margin-[purpleView]-margin-|" options:NSLayoutFormatAlignAllTop|NSLayoutFormatAlignAllBottom metrics:@{@"margin":@20.0} views:NSDictionaryOfVariableBindings(orangeView, purpleView)]];
[constraints addObjectsFromArray:[NSLayoutConstraint constraintsWithVisualFormat:@"V:|-20-[orangeView(==height)]" options:NSLayoutFormatDirectionLeadingToTrailing metrics:@{@"height":@40.0} views:NSDictionaryOfVariableBindings(orangeView, purpleView)]];
[self.view addConstraints:constraints];
```