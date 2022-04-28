---
title: Unable to prepare iPhone for development
abbrlink: e1d3f6a5
date: 2022-04-28 10:05:06
tags:
    - iOS
---

Xcode 真机调试时突然提示：
```
Please check the connection to the device, and review all errors in the Devices and Simulators window.
```

打开 `Window` -- `Devices and Simulators` 面板，看到如下提示：

```
Failed to prepare device for development.
if youare certain thax xcode supports development on this device, try disconnecting and  reconnecting the device
```

因为之前就能正常调试的，排除了 Xcode 版本和手机系统版本的兼容问题，也没进行任何更新。

> 所以只需要 `重启手机` 即可。
