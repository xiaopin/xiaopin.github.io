---
title: iOS 逆向之 Cycript 篇
tags:
  - iOS
  - 逆向
abbrlink: 19b70caf
date: 2019-06-06 15:33:43
---

Cycript 是由 saurik 推出的一款脚本语言，可以看作是 Objective-C 和 JavaScript 的结合物，可通过 Cydia 安装。

#### 注入进程

一般都是使用 cycript 来进行一些代码的测试，所以需要先把 cycript 注入到目标应用的进程中，通过如下方式注入：

```
cycript -p pid/进程名称
```

比如我们想注入到微信的进程中，可以先通过 ps 命令查找到微信的进程信息，然后再注入：

```
iPhone:~ root# ps -e | grep WeChat
 4778 ??         0:08.66 /var/containers/Bundle/Application/D2F238D4-3D14-402A-B3A8-826D9FCD39C6/WeChat.app/WeChat
 5259 ttys001    0:00.01 grep WeChat
iPhone:~ root# cycript -p 4778
cy# 
```

当终端出现 `cy#` 时表示注入成功，除了 `cycript -p 4778` 外还可以 `cycript -p WeChat` 这样子进行注入。

#### 调试代码

当终端出现 cy# 提示符时，说明我们已经成功将 cycript 注入到目标应用进程当中了，这个时候我们可以输入 OC 代码进行调试，比如打印 keyWindow

```
cy# [[UIApplication sharedApplication] keyWindow]
#"<iConsoleWindow: 0x1102c6100; baseClass = UIWindow; frame = (0 0; 320 568); gestureRecognizers = <NSArray: 0x1102c8160>; layer = <UIWindowLayer: 0x1102cd330>>"
```

你会发现这和我们平时写的 OC 代码没什么两样，没错，cycript 就是能够执行你编写的 OC 代码。

当我们知道一个对象的内存地址时，可以通过 `#` 操作符来获取这个对象，如打印根控制器：

```
cy# [#0x1102c6100 rootViewController]
#"<MMUINavigationController: 0x10d001200>"
```

怎么样，是不是很兴奋很激动，蠢蠢欲试呢。

#### choose 命令

如果你知道一个类对象存在于当前进程中，却不知道它的内存地址，那就不能通过 # 操作符来获取到这个对象了，但是，cycript 为我们提供了一个 choose 命令，用法如下：

```
choose(ClassName)
```

> 类名不需要引号

```
cy# choose(MMUINavigationController)
[#"<MMUINavigationController: 0x10d001200>",#"<MMUINavigationController: 0x10e0b7800>"]
cy# choose(UIButton)
[#"<FixTitleColorButton: 0x1115734d0; baseClass = UIButton; frame = (170 18; 130 47); clipsToBounds = YES; opaque = NO; autoresize = LM; layer = <CALayer: 0x111573320>>",#"<UIButton: 0x111575aa0; frame = (234 20; 86 49); opaque = NO; autoresize = LM; layer = <CALayer: 0x1115724a0>>",#"<FixTitleColorButton: 0x1105d1700; baseClass = UIButton; frame = (20 18; 130 47); clipsToBounds = YES; opaque = NO; autoresize = RM; layer = <CALayer: 0x1105d19e0>>"]
```

我们只需要为 choose 提供一个类名即可，Cycript 就会为我们在当前进程内存中找出该类的实例对象供我们使用，很方便是不是？

#### 模块封装

如果我们在调试时经常用到某些代码，那每次都重复敲写，是不是特麻烦呢，吃力不讨好还效率低下。那我们能不能将这些代码封装起来进行复用呢，答案是当然可以。

我们可以将一些常用的功能代码封装到一个 `.cy` 结尾的文件中（假设叫 test.cy），

```ObjC
(function(exports) {
	AppId = NSBundle.mainBundle.bundleIdentifier;

	RootVC = function() {
		return [UIApplication sharedApplication].keyWindow.rootViewController;
	};

	exports.nothing = function() {};

	// ...
})(exports);
```

然后将该脚本文件拷贝到 iOS 的 `/usr/lib/cycript0.9` 目录下，注入到目标进程，然后可以通过 `@import 模块名` 命令导入。

```
cy# @import test
{}
cy# AppId
@"com.tencent.xin"
cy# RootVC()
#"<MMUINavigationController: 0x10d001200>"
cy# test.nothing()
```

> 注意 nothing 方法的调用需要加上模块名

通过这样的方式，我们可以封装一些常用的功能，方便调试。

如果懂 JavaScript 的朋友，肯定会惊讶这不就是 JavaScript 的模块语法吗，是的，所以我说 cycript 是 Objective-C 和 JavaScript 结合物嘛。

推荐一下`小码哥`写的脚本 [mjcript](https://github.com/CoderMJLee/mjcript)
