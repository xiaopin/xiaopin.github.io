---
title: 隐藏NSWindow的titleBar
date: 2017-03-27 14:46:21
tags:
	- macOS
---

新建一个macOS应用项目，编译运行后默认出现的界面是如下图所示的样子：

![默认效果](/images/201703/titlebar-default.png)

有时候我们需要去掉titleBar，类似QQ登录界面那种，此时我们可以在继承自NSWindowController的自定义类的-windowDidLoad方法中通过下面的代码实现效果：

``` ObjC
- (void)windowDidLoad {
  [super windowDidLoad];
  [self.window setTitleVisibility:NSWindowTitleHidden];
  [self.window setTitlebarAppearsTransparent:YES];
  [self.window setStyleMask:NSWindowStyleMaskFullSizeContentView|NSWindowStyleMaskTitled|NSWindowStyleMaskClosable|NSWindowStyleMaskMiniaturizable|NSWindowStyleMaskResizable];
}
```

我们也可以直接在IB中进行设置，打开Storyboard并选中NSWindowController中的NSWindow，展开右侧工具栏并切换至属性面板，勾选上`Full Size Content View`选项即可。如果在IB中进行设置，则无需在代码中调用`-setStyleMask:`方法。

最终效果如下图所示：

![](/images/201703/titlebar-hide.png)
