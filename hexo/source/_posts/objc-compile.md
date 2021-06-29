---
title: objc4-818.2 源码编译调试
abbrlink: 1204fdf4
date: 2021-06-28 16:31:42
tags:
    - Objective-C
    - iOS
    - macOS
---

在平常的开发中，我们都只是知道有 objc 这个框架，为我们提供了一些底层接口(例如 runtime 以及 objc_msgSend 等)；如果我们能够将 objc 源码跑起来，加上 LLDB 进行调试，就能够清晰的分析整个代码的执行流程。

## 当前环境

- macOS 11.4 (MBP 2015)
- Xcode 12.5
- objc4-818.2

## 获取源码

源码可以从苹果开源官网直接下载回来

- [https://opensource.apple.com/](https://opensource.apple.com/)
- [https://opensource.apple.com/tarballs/](https://opensource.apple.com/tarballs/)

本来想试试最新的 `objc4-824` 的源码，但是下载的时候报错了，所以还是拿 818.2 版本来练手吧。

```
error message
Please report the missing source or broken link to opensource@apple.com
```

## 开始配置

- 解压下载好的 `objc4-818.2.tar`
- 打开 objc.xcodeproj 项目工程
- 将当前的 Scheme 设置为 `objc` --> `My Mac`

#### unable to find sdk 'macosx.internal'

- `PROJECTS` --> `objc` --> `Build Settings`, 将 `Base SDK` 改成 `macOS`
- `TARGETS` --> `objc` --> `Build Settings`, 将 `Base SDK` 改成 `macOS`

#### 'sys/reason.h' file not found

- 下载 [xnu-7195.81.3.tar.gz](https://opensource.apple.com/tarballs/xnu/)
- 在项目根目录新建一个文件夹(例如：MissFiles)
- 把 `xnu-7195.81.3/bsd/sys/reason.h` 文件复制到 `MissFiles/sys` 目录下
- 设置头文件搜索路径：`TARGETS` --> `objc` --> `Build Settings` --> `Header Search Paths`, 添加一条 `$(SRCROOT)/MissFiles`

#### 以下的报错处理方式和上面的处理方式相同, 都是拷贝对应的头文件到对应的目录下, 具体信息请参照下表：

| 错误描述 | 对应的资料 | 对应目录 | 备注 |
| :- | :- | :- | :- |
| 'sys/reason.h' file not found | [xnu-7195.81.3/bsd/sys/reason.h](https://opensource.apple.com/tarballs/xnu/xnu-7195.81.3.tar.gz) |  MissFiles/sys/   | |
| 'mach-o/dyld_priv.h' file not found | [dyld-851.27/include/mach-o/dyld_priv.h](https://opensource.apple.com/tarballs/dyld/dyld-851.27.tar.gz) | MissFiles/mach-o/ | 拷贝后会报类似错误 `/MissFiles/mach-o/dyld_priv.h:130:130: Expected ','`，需要将 `, bridgeos(3.0)` 这个内容全部注释(删除) |
| 'os/lock_private.h' file not found | [libplatform-254.80.2/private/os/lock_private.h](https://opensource.apple.com/tarballs/libplatform/libplatform-254.80.2.tar.gz) | MissFiles/os/ | |
| 'os/base_private.h' file not found | [libplatform-220.100.1/private/os/base_private.h](https://opensource.apple.com/tarballs/libplatform/libplatform-254.80.2.tar.gz) | MissFiles/os/ | 注意这里下载的是 `220.100.1` 版本，因为后面更新的版本未提供该文件 |
| 'pthread/tsd_private.h' file not found | [libpthread-454.100.8/private/pthread/tsd_private.h](https://opensource.apple.com/tarballs/libpthread/libpthread-454.100.8.tar.gz) | MissFiles/pthread | |
| 'pthread/spinlock_private.h' file not found | [libpthread-454.100.8/private/pthread/spinlock_private.h](https://opensource.apple.com/tarballs/libpthread/libpthread-454.100.8.tar.gz) | MissFiles/pthread | |
| 'System/machine/cpu_capabilities.h' file not found | [xnu-7195.81.3/osfmk/machine/cpu_capabilities.h](https://opensource.apple.com/tarballs/xnu/xnu-7195.81.3.tar.gz) | MissFiles/System/machine/ | |
| 'os/tsd.h' file not found | [xnu-7195.81.3/libsyscall/os/tsd.h](https://opensource.apple.com/tarballs/xnu/xnu-7195.81.3.tar.gz) | MissFiles/os/ | |
| 'System/pthread_machdep.h' file not found | [Libc-825.40.1/pthreads/pthread_machdep.h](https://opensource.apple.com/tarballs/Libc/Libc-825.40.1.tar.gz) | MissFiles/System/ | 拷贝后会有 4 个地方报错，把报错的代码通通注释掉 |
| 'CrashReporterClient.h' file not found | [Libc-825.40.1/include/CrashReporterClient.h](https://opensource.apple.com/tarballs/Libc/Libc-825.40.1.tar.gz) | MissFiles/ | |
| '\_simple.h' file not found | [libplatform-254.80.2/private/\_simple.h](https://opensource.apple.com/tarballs/libplatform/libplatform-254.80.2.tar.gz) | MissFiles/ | |
| 'objc-shared-cache.h' file not found | [dyld-851.27/include/objc-shared-cache.h](https://opensource.apple.com/tarballs/dyld/dyld-851.27.tar.gz) | MissFiles/ | |
| 'Block_private.h' file not found | [libclosure-79/Block_private.h](https://opensource.apple.com/tarballs/libclosure/libclosure-79.tar.gz) | MissFiles/ | |
| 'kern/restartable.h' file not found | [xnu-7195.81.3/osfmk/kern/restartable.h](https://opensource.apple.com/tarballs/xnu/xnu-7195.81.3.tar.gz) | MissFiles/kern/ | |
| 'os/linker_set.h' file not found | [Libc-1439.100.3/os/linker_set.h](https://opensource.apple.com/tarballs/Libc/Libc-1439.100.3.tar.gz) | MissFiles/os/ | |
| 'os/feature_private.h' file not found | | | 注释相关的引用 |
| 'Cambria/Traps.h' file not found | | | 注释相关的引用 |
| 'Cambria/Cambria.h' file not found | | | 注释相关的引用 |
| 'objc-bp-assist.h' file not found | | | 注释相关的引用 |
| 'os/reason_private.h' file not found | [xnu-7195.81.3/libkern/os/reason_private.h](https://opensource.apple.com/tarballs/xnu/xnu-7195.81.3.tar.gz) | MissFiles/os/ | |
| 'os/variant_private.h' file not found | [Libc-1439.100.3/os/variant_private.h](https://opensource.apple.com/tarballs/Libc/Libc-1439.100.3.tar.gz) | MissFiles/os/ | 拷贝后需要将 `, bridgeos` 和 `, bridgeos(4.0)` 注释掉 |

上面有些库并不是最新的，因为最新版的可能并未包含我们所需的头文件，所以你可以适当找老版本。如果你不知道确实的头文件在哪个库里面，你可以通过在谷歌中输入 `xxx.h site:opensource.apple.com` 来定向检索，或许能够直接定位到你需要的库。

#### 需要注释的地方(大体思路就是不清楚你就注释就行了)

- oah_is_current_process_translated
- dyld_platform_version_macOS_10_13
- dyld_fall_2020_os_versions
- os_feature_enabled_simple(objc4, preoptimizedCaches, true)
- dyld_platform_version_macOS_10_11
- LINKER_SET_FOREACH
- dyld_fall_2018_os_versions
- sdkIsAtLeast(10_12, 10_0, 10_0, 3_0, 2_0)
- '\_static_assert' declared as an array with a negative size

#### libobjc.order 错误

```
ld: can't open order file: /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX11.3.sdk/AppleInternal/OrderFiles/libobjc.order
clang: error: linker command failed with exit code 1 (use -v to see invocation)

Can't open order file: /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX11.3.sdk/AppleInternal/OrderFiles/libobjc.order
```

- 找到 `TARGETS` --> `objc` --> `Build Settings` --> `Order File`
- 把 `Debug` 的搜索路径改成 `$(SRCROOT)/libobjc.order`

#### library not found for -loah

- 找到 `TARGETS` --> `objc` --> `Build Settings` --> `Other Linker Flags`
- 把 Debug 和 Release 中的 `-loah` 都删掉

#### Xcode 编译脚本的错误

```
/Users/xiaopin/Downloads/objc4-818.2/xcodebuild:1:1: SDK "macosx.internal" cannot be located.
/Users/xiaopin/Downloads/objc4-818.2/xcrun:1:1: sh -c '/Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild -sdk macosx.internal -find clang++ 2> /dev/null' failed with exit code 16384: (null) (errno=No such file or directory)
/Users/xiaopin/Downloads/objc4-818.2/xcrun:1:1: unable to find utility "clang++", not a developer tool or in PATH
```

- 找到 `TARGETS` --> `objc` --> `Build Phases` -> `Run Script(markgc)`
- 把脚本的 `macosx.internal` 改成 `macosx`

#### 'CrashReporterClient.h' file not found

在上面明明把 `CrashReporterClient.h` 文件拷贝到 MissFiles 目录下了，但是还是报文件未找到的错误。

- `TARGETS` --> `objc` --> `Build Settings` --> `Other Linker Flags`, 删除 Debug 和 Release 中的 `-lCrashReporterClient`

#### Build Succeeded

经过上面繁琐的配置后，重新按下 command + B 进行编译，此时终于看到了久违的 `Build Succeeded`，恭喜 🎉🎉🎉，我们已经离成功不远了。

## 调试源码

- 新建一个终端命令行程序, `File` --> `New` --> `Target...` --> `macOS` --> `Command Line Tool`, 我取名为 `MYTest`
- 绑定二进制依赖
    - 找到 `TARGETS` --> `MYTest` --> `Build Phases` --> `Dependencies`, 点击 `+` 号把 `objc` 添加进去
    - 找到 `TARGETS` --> `MYTest` --> `Build Phases` --> `Link Binary Width Libraries`, 点击 `+` 号把 `libobjc.A.dylib` 添加进去
    
    ![PNG](/images/2021/06/001.png)
    
- 新建一个 `MYObject` 类, 并修改 main.m 的代码

    ```ObjC
    #import <Foundation/Foundation.h>
    #import "MYObject.h"

    int main(int argc, const char * argv[]) {
        @autoreleasepool {
            MYObject *obj = [MYObject alloc];
        }
        return 0;
    }
    ```

- 在 main.m 中打了断点，发现并不能断在断点处
    - 找到 `TARGETS` --> `MYTest` --> `Build Phases` --> `Compile Source`
    - 把 main.m 拖到第一个位置

好了，现在可以愉快地调试 objc 源码了 🎉🎉🎉

最后附上一张效果图

![PNG](/images/2021/06/002.png)

## 参考

- [最新 macOS 10.15 下 objc4-779.1 源码编译调试](https://juejin.cn/post/6844904082226806792)
- [iOS_objc4-756.2 最新源码编译调试](https://juejin.cn/post/6844903959161733133)
