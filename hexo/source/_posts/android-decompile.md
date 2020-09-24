---
title: Android 反编译
tags:
  - Android
abbrlink: a6e7e68d
date: 2019-03-25 12:53:36
---

> 可以从各大应用商店下载 apk 安装包

## 获取 classes.dex 文件

将 apk 后缀改成 `.zip`，然后解压出来的文件就会包含 `classes.dex`


## 将 dex 反编译成 jar 包

1、下载 [dex2jar](https://sourceforge.net/projects/dex2jar/)

2、将 dex2jar-2.0.zip 解压，并将 classes.dex 复制到解压出来的目录中

3、将 dex 转成 jar 包
```bash
$ cd dex2jar-2.0
$ chmod 0777 *.sh
$ sh d2j-dex2jar.sh classes.dex
```

执行完上面的命令，就会看到反编译后的 jar 包 `classes-dex2jar.jar`


## 通过 jd-gui 查看源码：

- 下载 [jd-gui](http://jd.benow.ca)

- 启动 jd-gui 工具
	```bash
	$ java -jar jd-gui-1.4.1.jar
	```

然后选择打开刚才反编译出来的 classes-dex2jar.jar 即可查看源码


## 获取 apk 的资源(图片、音视频等)文件

- 下载 [ApkTool](https://www.softpedia.com/get/Programming/Debuggers-Decompilers-Dissasemblers/ApkTool.shtml) 工具

- java -jar apktool_2.4.0.jar d -f xx.apk -o target_dir
	- xx.apk为你下载的apk安装包
	- target_dir为目标目录，反编译后的资源文件将保存在该目录下
