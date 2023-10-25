---
title: 在 macOS/Linux 上查找大文件
abbrlink: b9cdd915
date: 2023-09-25 10:25:50
tags:
    - macOS
---

有时候我们想要在某个目录下过滤出一些大文件，通过 `find` 命令，我们可以轻松实现这个需求。

1、在当前目录下，找出文件大小大于 1G 的文件(+表示大于)

```shell
find . -type f -size +1G
```

2、在当前目录下，找出文件大小小于 10M 的文件(-表示小于)

```shell
find . -type f -size -10M
```

3、在当前目录下，找出文件大小为 100M ～ 1G 的文件

```shell
find . -type f -size +100M -size -1G
```

通过 `man find` 可以找到 `-size` 的相关文档说明：

```
-size n[ckMGTP]
    True if the file's size, rounded up, in 512-byte blocks is n.  If
    n is followed by a c, then the primary is true if the file's size
    is n bytes (characters).  Similarly if n is followed by a scale
    indicator then the file's size is compared to n scaled as:

    k       kilobytes (1024 bytes)
    M       megabytes (1024 kilobytes)
    G       gigabytes (1024 megabytes)
    T       terabytes (1024 gigabytes)
    P       petabytes (1024 terabytes)
``` 
