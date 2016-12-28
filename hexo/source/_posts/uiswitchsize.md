---
title: 修改UISwitch大小
date: 2016-12-28 20:51:24
tags:
	- iOS
---

> UISwitch大小是不能修改的，iOS7开始，系统将UISwitch大小固定为51x31。

尽管UISwitch不能通过修改`frame`的方式修改大小，但是可以通过`transform`来进行缩放，变相达到我们想要的效果。

``` ObjC
UISwitch *aSwitch = [[UISwitch alloc] init];
[aSwitch setFrame:CGRectMake(100.0, 100.0, 0.0, 0.0)];
[aSwitch setTransform:CGAffineTransformMakeScale(0.5, 0.5)];
[self.view addSubview:aSwitch];
```
