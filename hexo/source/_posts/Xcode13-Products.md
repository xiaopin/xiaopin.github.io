---
title: Xcode13 在左侧项目目录结构树中没有 Products 文件夹
abbrlink: 677f6bca
date: 2022-04-28 14:19:36
tags:
    - iOS
    - macOS
    - Other
---

以前老版本 Xcode，编译项目后会在左侧项目目录中有个 Products 的文件夹，里面包含了我们编译生成的 app 文件，但是更新到 Xcode13 之后，新建的项目你会发现在左侧目录中没有了这个 Products 文件夹，但是我们可以通过菜单快速定位到这个目录。

> 点击菜单栏 Product — Show Build Folder in Finder

![Editor Product Debug Source Control Window](/images/2022/EditorProductDebugSourceControlWindow.png)