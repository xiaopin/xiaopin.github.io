---
title: iOS 砸壳教程
date: 2019-05-28 14:39:31
tags:
	- iOS
	- 逆向
---

iOS 从 App Store 下载的应用都是加过壳的（`只有从 App Store 下载`的才是加壳的，系统应用和其他渠道下载的都是未加壳的），要想对应用进行逆向，首先得把这层壳去掉，俗称`脱壳` 或 `砸壳`。

#### 工具

- OpenSSH
- Cycript
- dumpdecrypted
- 已越狱的 iOS 设备

****

##### OpenSSH

通过 Cydia 安装，我们最常用的只有两个命令，`ssh` 和 `scp`，前者用于远程登录，后者用于远程拷贝文件。

- ssh

格式：
```
ssh user@ip
```

如用户名为 root，IP 为 192.168.1.2，则：ssh root@192.168.1.2

- scp
格式：
```
scp [-P port] /path/source-file user@ip:/path/target-directory
```

如：将 macOS 桌面的 test.txt 拷贝到 iPhone 的 /usr/local 目录下

```
scp ~/Desktop/test.txt root@192.168.1.2:/usr/local/
```

当然，也可以将 iPhone 中的文件拷贝出来
```
scp root@192.168.1.2:/usr/local/test.txt ~/Desktop/
```

> usbmuxd 工具使我们能够通过 USB 线 ssh 到 iOS 中，极大地增加了 ssh 连接的速度，强烈推荐。可以在 [https://cgit.sukimashita.com/usbmuxd.git/](https://cgit.sukimashita.com/usbmuxd.git/) 下载到，推荐下载 1.0.8 版本。

##### Cycript

是由 saurik 推出的一款脚本语言，可以说是 JavaScript + Objective-C 的结合物。功能强大，具体用法自行查阅吧。

这里推荐一下 MJ 推出的脚本 [mjcript](https://github.com/CoderMJLee/mjcript)，可以更加方便调试。

##### dumpdecrypted

dumpdecrypted 是个开源工具，可在 GitHub 上找到。

- 将 [dumpdecrypted](https://github.com/stefanesser/dumpdecrypted.git) 源码下载回来
- 通过 make 命令编译生成 dumpdecrypted.dylib

后面会通过 dumpdecrypted.dylib 动态库进行砸壳。

#### 砸壳

> 192.168.100.100 为我手机当前的 IP 地址

下面以 `VB酒庄` 这个 App 为例进行演示。

1、进行操作前，我们先通过 OpenSSH 连接到 iPhone。

```
ssh root@192.168.100.100
```

iPhone 默认有两个账户，`root` 和 `mobile`，默认密码都是 `alpine`。建议首次登录后，通过 passwd 命令修改这两个用户的默认密码。

2、找到应用的可执行文件路径(Mach-O)

```
ps -e | grep VBemall
```

执行上述命令后，可以看到 App 的 MachO 文件所在路径为: /var/containers/Bundle/Application/14E5846E-130A-4E7A-AF30-033EB6041A9D/VBemall.app/VBemall

3、获取 Documents 目录

- 把 cycript 注入到应用进程
```
cycript -p VBemall
```

如果终端中出现 `cy#` 则说明注入成功

- 执行 [[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask][0] 获取 Documents 目录路径

`control + D` 可以退出 Cycript，如果执行 `exit(0)` 则会退出 Cycript 并关闭 App

4、将 dumpdecrypted.dylib 拷贝到 Documents 目录下

>  scp 源文件 root@ip:目标路径

```
scp dumpdecrypted.dylib root@192.168.100.100:/var/mobile/Containers/Data/Application/210D1E28-3BD0-43E7-9FEC-AA311AD734F4/Documents/
```

5、 进入 Documents 目录开始砸壳

- 进入 Documents 目录
```
cd /var/mobile/Containers/Data/Application/210D1E28-3BD0-43E7-9FEC-AA311AD734F4/Documents
```

- 通过 `DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib /path/executable` 命令进行砸壳
```
DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib /var/containers/Bundle/Application/14E5846E-130A-4E7A-AF30-033EB6041A9D/VBemall.app/VBemall
```

如果砸壳成功，会在当前目录(dumpdecrypted.dylib 所处目录)生成 `VBemall.decrypted` 文件，该文件就是砸壳后的 MachO 文件，可以通过 class-dump 导出头文件或者通过 IDA、Hopper 查看汇编代码。

> 如果在执行砸壳命令时提示 `Killed: 9` 的错误信息，需要通过 `su mobile` 切换到 mobile 用户后再次砸壳。具体可以参看 [Issues](https://github.com/stefanesser/dumpdecrypted/issues/19)。
    
6、将砸壳后的文件从 iPhone 中拷贝到桌面
```
scp root@192.168.100.100:/var/mobile/Containers/Data/Application/210D1E28-3BD0-43E7-9FEC-AA311AD734F4/Documents/VBemall.decrypted ~/Desktop/
```
