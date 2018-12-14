---
title: SDWebImage 4.x 播放GIF图片
date: 2018-12-14 15:07:16
tags:
	- iOS
---

今天产品部的突然说 iOS 版 App 播放不了 GIF，一接到这个反馈信息，我一开始是不相信的，因为之前的旧版也是用的 `SDWebImage` 这个库且未做任何代码适配就能够播放 GIF，所以我内心是极不愿相信这是我的锅。

> 新版 App 重构后也上线了两个多月了，这期间由于 App 未涉及到 GIF 的播放，所以一直未发现该问题(~~是我工作的疏忽~~)。

虽然极不愿相信，但问题已出，不得不进行排查。网上随便一搜就发现事情不对，重构后的项目用的是 4.3.3 版本的 SDWebImage，而旧项目用的还是 3.8.2 版本的；而 SDWebImage 在 4.0 版本开始就已经采用 [FLAnimatedImage](https://github.com/Flipboard/FLAnimatedImage) 来播放 GIF 了。

下面是 SDWebImage 关于 GIF 的说明：

#### Animated Images (GIF) support

- Starting with the 4.0 version, we rely on [FLAnimatedImage](https://github.com/Flipboard/FLAnimatedImage) to take care of our animated images. 
- If you use cocoapods, add `pod 'SDWebImage/GIF'` to your podfile.
- To use it, simply make sure you use `FLAnimatedImageView` instead of `UIImageView`.
- **Note**: there is a backwards compatible feature, so if you are still trying to load a GIF into a `UIImageView`, it will only show the 1st frame as a static image by default. However, you can enable the full GIF support by using the built-in GIF coder. See [GIF coder](https://github.com/SDWebImage/SDWebImage/wiki/Advanced-Usage#gif-coder)
- **Important**: FLAnimatedImage only works on the iOS platform. For macOS, use `NSImageView` with `animates` set to `YES` to show the entire animated images and `NO` to only show the 1st frame. For all the other platforms (tvOS, watchOS) we will fallback to the backwards compatibility feature described above 

虽然采用了 FLAnimatedImage，但是也并没废弃之前的功能不是，SDWebImage 还是内建了 GIF 编码器的，只是默认并未启用而已。

SDWebImage 在 Readme 中已经清楚说明了如何播放 GIF：

- 使用 `FLAnimatedImageView` 代替 `UIImageView`
- 使用 GIF 编码器，详情参看 [文档](https://github.com/SDWebImage/SDWebImage/wiki/Advanced-Usage#gif-coder)

方案1的优点是节省性能、速度快，但是麻烦；方案2的优点是只需添加一行代码启用 GIF 编码器即可完成需求，但是缺点就是性能相对来说没那么高效。

我个人采用了方案2，毕竟要将项目中的 UIImageView 替换成 FLAnimatedImageView 的工作量相对来说比较大（~~虽然实际上可能也就半个小时的功夫，但是相对于方案2来说，还是太费时费力了，况且方案2的性能也不是不能接受，毕竟以前也是用的这套方案不是~~）。

---

解决方案：

在 UIImageView 的 Category 文件中添加以下代码:

```ObjC
#import <SDWebImage/SDWebImageCodersManager.h>
#import <SDWebImage/SDWebImageGIFCoder.h>

+ (void)initialize {
    [SDWebImageCodersManager.sharedInstance addCoder:SDWebImageGIFCoder.sharedCoder];
}
```

## 总结

既然用到了第三方库，有空还是要多了解一下版本更新情况。