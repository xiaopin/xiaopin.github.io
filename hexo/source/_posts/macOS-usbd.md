---
title: iPhone 链接 macOS 不断闪烁
abbrlink: f42493ad
date: 2022-04-28 15:14:59
tags:
    - macOS
    - Other
---

当 iPhone 通过数据线连接 macOS 时，iPhone 的充电状态一直在闪烁，频繁的在 `断开/已连接` 之间不停地切换。

如果重启电脑/手机、更换数据线都不能解决时，可以尝试把 `usbd` 服务停掉：

```sh
sudo killall -STOP -c usbd
```

或

```sh
sudo pklll -9 usbd
```

然后重新连接试试。
