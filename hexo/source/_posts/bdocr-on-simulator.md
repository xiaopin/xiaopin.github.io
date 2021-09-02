---
title: 解决集成百度OCR后，模拟器不能运行的问题
tags:
  - Objective-C
  - iOS
abbrlink: a345c52c
date: 2021-09-02 11:15:28
---

## 模拟器报错

由于项目需要识别银行卡信息和身份证信息，最终决定采用百度家的 [OCR SDK](https://ai.baidu.com/ai-doc/OCR/6kibizx7f)，一共有三个库：
- AipBase.framework
- AipOcrSdk.framework
- IdcardQuality.framework

当我们按照文档将 SDK 集成到项目后，想在模拟器上运行时，编译就报错了：

```
Building for iOS Simulator, but the linked and embedded framework 'AipBase.framework' was built for iOS.
Building for iOS Simulator, but the linked and embedded framework 'AipOcrSdk.framework' was built for iOS.
Building for iOS Simulator, but the linked and embedded framework 'IdcardQuality.framework' was built for iOS.
```

![png](/images/2021/09/101.png)

## 解决方案

既然 SDK 未提供模拟器的 x86_64/i386 的架构，那么我们就在模拟器运行时不启用这些功能就好了。

- 修改项目配置

找到 `Build Settings` - `Build Options` - `Excluded Source File Names`，在 DEBUG 模式下添加以下配置：
```
Any iOS Simulator: AipBase.framework
Any iOS Simulator: AipOcrSdk.framework
Any iOS Simulator: IdcardQuality.framework
```

![png](/images/2021/09/102.png)

- 修改代码

在你用到这些 SDK 的代码中进行处理，在模拟器环境下不启用这些功能：

```ObjC
#if !TARGET_IPHONE_SIMULATOR
#import <AipOcrSdk/AipOcrSdk.h>
#endif
```


```ObjC
- (IBAction)ocrButtonAction:(UIButton *)sender {
#if TARGET_IPHONE_SIMULATOR
    NSLog(@"该功能不支持模拟器，请真机测试");
#else
    [[AipOcrService shardService] authWithAK:kBaiduOCRAppKey andSK:kBaiduOCRSecretKey];
    // 真机环境的处理代码...
#endif
}
```

百度社区有人也提问，并有人给出了方案，但是我没测试，你可以自行尝试一下，帖子请戳[这里](https://ai.baidu.com/forum/topic/show/492731)。