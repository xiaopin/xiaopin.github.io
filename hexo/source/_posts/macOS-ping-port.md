---
title: macOS 检测端口是否连通
abbrlink: 7595a178
date: 2022-05-06 09:30:02
tags:
    - macOS
---

我们想要知道某个IP是否能访问，可以通过 `ping` 命令检测：

```shell
ping -c 5 192.168.1.1
```

但是我们想进一步检测某个IP对应的电脑是否打开了某个端口，这时就不能用 `ping` 命令了，而是用 `nc` 命令：

```shell
nc -vz -w 2 196.168.1.1 443
```

如果看到终端输出 `Connection to 196.168.1.1 port 443 [tcp/canon-bjnp2] succeeded!` 字样字符串，则说明端口是连通的，否则说明端口未打开。
