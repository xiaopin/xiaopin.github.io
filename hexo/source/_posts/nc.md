---
title: ping IP和端口
abbrlink: 6e1c6434
date: 2021-05-17 15:43:51
tags:
    - macOS
---

## macOS

- IP

```Shell
ping 127.0.0.1
```

- 端口扫描

```Shell
nc -vz -w 5 127.0.0.1 8080
```

参数说明：

 - v 表示需要输出详细的日志信息
 - z 表示只扫描监听守护进程，而不是向他们发送任何数据(不能与 `-l` 参数一起使用)
 - w 设置超时时间(秒), 不设置则表示无超时一直等待

当看到类似 `Connection to 127.0.0.1 port 8080 [tcp/terabase] succeeded!` 的日志则说明端口是通的。

## Linux

Linux 的使用直接参考 macOS 即可。

## Windows

### ping

Windows 也可以直接使用 ping 命令。

### telnet

系统默认未开启 telnet 命令，需要先启用。

在控制面板--打开或关闭Windows功能--Telnet客户端，勾选并确定，等待系统把telnet工具安装好。

```Shell
telent 127.0.0.1 8080
```

如果能够进入 telnet 的窗口页面(全黑的窗口)，则说明端口可用，否则会给出相应的错误信息。

### tcping

Windows 可以通过安装 [tcping](https://download.elifulkerson.com/files/tcping/) 工具，也是可以 ping IP 和端口。

```Shell
tcping 127.0.0.1 8080
```
