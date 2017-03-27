---
title: Xcode8 Swift3通过Playground运行UI
date: 2017-03-27 14:28:21
tags:
	- iOS
---

我们都知道在Playground上能够一边写代码一边预览代码运行效果（所见即所得），如果我们需要在Playground上调试UI，就得借助于PlaygroundSupport系统库了。

1、导入PlaygroundSupport库

``` Swift
import PlaygroundSupport
```

2、创建UIView，在这一步你可以创建你自己想要的视图

``` Swift
let frame = CGRect(x: 0, y: 0, width: 100, height: 50)
let view = UIView(frame: frame)
view.backgroundColor = UIColor.purple
```

3、将UIView赋值给liveView

``` Swift
PlaygroundPage.current.liveView = view
```

4、Xcode打开UI测试界面：

> View —> Assistant Editor —> Show Assistant Editor

> 快捷键：`Option`+`Command`+`Enter`
