---
title: macOS应用窗口居中显示
date: 2017-03-27 15:00:50
tags:
	- macOS
---

> 调整窗口大小或居中显示，必须将NSWindow的`restorable`属性设置为NO，否则无效果。

应用在关闭时会记住window当前的位置以及frame，在下次启动应用时，系统会将window显示在上次关闭时的位置，并恢复关闭时的frame。有时候我们希望在每次打开应用时，应用都能居中显示，或者是我们需要在应用启动时，指定窗口的frame，则可以在NSWindowController的windowDidLoad方法进行设置：

``` ObjC
- (void)windowDidLoad {
  [super windowDidLoad];
  [self.window setRestorable:NO];
  [self.window setFrame:NSMakeRect(0.0, 0.0, 800.0, 600.0) display:YES];
  [self.window center];
}
```
