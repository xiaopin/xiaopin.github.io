---
title: iOS添加文件共享
tags:
  - iOS
abbrlink: 14aacc8d
date: 2017-10-09 09:36:55
---

由于iOS的沙盒机制，导致应用不能访问沙盒目录以外的文件目录。所以如果我们需要将音视频文件拷贝到App上，只能拷贝到App的`Documents`目录
如果我们的App需要支持从PC将文件拷贝到App的Documents目录并展示出来，则需要在Info.plist中添加`UIFileSharingEnabled`(Application supports iTunes file sharing)键，并将键值设置为`YES`。

添加了UIFileSharingEnabled键值后就可以通过iTunes来管理共享文件了，可以从PC拷贝到App，也可以从App拷贝到PC上，也可以通过iTunes删除App上的文件(选中需要删除的文件，按下键盘的`delete`键即可)。

App可以通过访问Documents目录来获取需要关心的文件，过滤掉无关文件。
```Swift
if let document = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true).first {
    if let files = try? FileManager.default.contentsOfDirectory(atPath: document) {
        for file in files {
            // 可以通过后缀名来获取你关心的文件类型
            print(file)
        }
    }
}
```

有关`File Sharing`请参考：[https://support.apple.com/zh-cn/HT201301](https://support.apple.com/zh-cn/HT201301)
