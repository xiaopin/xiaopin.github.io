---
title: iOS11保存图片应用闪退
date: 2017-10-9 11:12:21
tags:
	- iOS
---

今天产品经理向我反馈在iOS11上点击保存图片后App闪退了，通过调试发现控制台输出了如下错误信息：

```
[access] This app has crashed because it attempted to access privacy-sensitive data without a usage description.  The app's Info.plist must contain an NSPhotoLibraryAddUsageDescription key with a string value explaining to the user how the app uses this data.
```

根据控制台输出的错误日志，已经很明确知道原因所在了，那就是Info.plist中缺少了一个`NSPhotoLibraryAddUsageDescription`的键值对。打开Info.plist，点击`+`添加一个键值对，在Key中输入`Privacy - Photo Library Additions Usage Description`，Value类型为`String`并输入相应的提示语即可。

iOS11中新增了以下的Key：
- NFCReaderUsageDescription
- NSFaceIDUsageDescription
- NSPhotoLibraryAddUsageDescription

如果你的App使用到了NFC、FaceID、保存图片的功能，记得添加相应的Key。


有关Key的说明请参考这里：
[https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html)
