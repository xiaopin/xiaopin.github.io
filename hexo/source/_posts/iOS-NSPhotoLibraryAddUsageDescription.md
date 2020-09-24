---
title: iOS11保存图片应用闪退
tags:
  - iOS
abbrlink: b4cba55d
date: 2017-10-09 11:12:21
---

今天产品经理向我反馈在iOS11上点击保存图片后App闪退了，通过调试发现控制台输出了如下错误信息：

```
[access] This app has crashed because it attempted to access privacy-sensitive data without a usage description.  The app's Info.plist must contain an NSPhotoLibraryAddUsageDescription key with a string value explaining to the user how the app uses this data.
```

根据控制台输出的错误日志，已经很明确知道原因所在了，那就是Info.plist中缺少了一个`NSPhotoLibraryAddUsageDescription`的键值对。打开Info.plist，点击`+`添加一个键值对，在Key中输入`Privacy - Photo Library Additions Usage Description`，Value类型为`String`并输入相应的提示语即可。

你也可以直接以`Source Code`方式打开Info.plist文件，然后复制以下内容并粘贴到Info.plist中：
```
<key>NSPhotoLibraryAddUsageDescription</key>
<string>此App需要您的授权才能保存图片</string>
```

iOS11中新增了以下的Key：
- NFCReaderUsageDescription
- NSFaceIDUsageDescription
- NSPhotoLibraryAddUsageDescription

如果你的App使用到了NFC、FaceID、保存图片的功能，记得添加相应的Key。

注意：
- iOS10上图库的访问和写入权限依然依赖于`NSPhotoLibraryUsageDescription`Key，如果在iOS10上只添加了`NSPhotoLibraryAddUsageDescription`而没有添加`NSPhotoLibraryUsageDescription`，当访问图库或者保存图片时应用依然会发生Crash
- iOS11上只要添加了`NSPhotoLibraryAddUsageDescription`即具备图库的访问和写入权限，无需增加`NSPhotoLibraryUsageDescription`
- iOS11上即使不添加`NSPhotoLibraryUsageDescription`和`NSPhotoLibraryAddUsageDescription`也可以通过`UIImagePickerController`来访问图库(其他API方式未测试，真机和模拟器均可访问到图库数据)，也不知道是否是Bug，居然不需要申请权限就可以访问系统图片。

综上所述，如果App需要兼容iOS10，则添加`NSPhotoLibraryUsageDescription`和`NSPhotoLibraryAddUsageDescription`，如果只兼容iOS11则只添加`NSPhotoLibraryAddUsageDescription`即可。


有关Key的说明请参考这里：
[https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html)
