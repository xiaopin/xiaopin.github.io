---
title: Sublime Text 报 There are no packages available for installation 错误
tags:
  - Other
abbrlink: 90f26031
date: 2019-05-27 17:53:23
---

#### 前言

最近升级了 Sublime Text 后突然安装不了插件了，卸载了重新安装还是照样报错。错误信息如下图所示：

![error.png](/images/201905/sublime-package-control.png)

通过 control + &#96; 组合键打开命令行面板，可以看到一些相关的错误日志。通过这些日志可以明显的知道是 https://packagecontrol.io/channel_v3.json 文件加载失败了（可能是被墙了吧，毕竟以前也没遇到这问题，Package Control 用的好好的嘛）。

既然知道问题的症结所在，那么我们就需要手动去下载 channel_v3.json 文件并告诉 Sublime Text 去加载它。

#### 下载 channel_v3.json 文件

你可以自行翻墙去下载，我在百度网盘上也提供了一份 [https://pan.baidu.com/s/1XDcnvNKKd2JJ5L3BHYsfXw](https://pan.baidu.com/s/1XDcnvNKKd2JJ5L3BHYsfXw)。

#### 修改 Package Control 配置

下载回来后，将 channel_v3.json 保存好，然后修改 Package Control 的配置。

依次打开菜单 Sublime Text -- Preferences -- Package Settings -- Package Control -- Settings - User，打开 Package Control 配置文件，将下面的配置项填写进去：

```
"channels":
[
	"/usr/local/opt/channel_v3.json"
]
```

以上修改针对 macOS 平台。