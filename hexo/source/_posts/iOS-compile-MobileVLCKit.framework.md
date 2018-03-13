---
title: iOS编译MobileVLCKit.framework
date: 2018-02-26 14:45:03
tags:
	- iOS
---

## 我的编译环境

- macOS 10.12.6
- Xcode9.2

## 编译MobileVLCKit.framework静态库

- 把源码clone下来
```
git clone https://code.videolan.org/videolan/VLCKit.git
```

- 开始编译MobileVLCKit,framework
```
cd VLCKit
./buildMobileVLCKit.sh -a all
```

![图1](/images/201802/vlc-compile.png)

如上图所示，说明编译完成并且成功了。整个编译过程异常缓慢，在我的iMac上足足编译了2个小时。

![图2](/images/201802/vlc-build-directory.png)

build目录下的MobileVLCKit.framework就是我们所需的静态库文件了。

## 示例代码

- 打开Xcode新建一个项目，然后把build目录下的MobileVLCKit.framework拖到项目中，并导入依赖库：（网上有些只是说导入后面的四个库即可，但是在我项目中如果仅导入后四个则会报错，需要完整导入上面的依赖库，你可以根据你的实际情况处理。）
	- CoreMedia.framework
	- VideoToolbox.framework
	- AVFoundation.framework
	- AudioToolbox.framework
	- libstdc++.tbd
	- libbz2.tbd
	- libz.tbd
	- libiconv.tbd

- 导入头文件
``` ObjC
#import <MobileVLCKit/MobileVLCKit.h>
```

- 声明一个`VLCMediaPlayer`类型的变量
```ObjC
@interface ViewController ()<VLCMediaPlayerDelegate>
@property (nonatomic, strong) VLCMediaPlayer *mediaPlayer;
@end
```

- 初始化VLCMediaPlayer
```ObjC
/* setup the media player instance, give it a delegate and something to draw into */
_mediaPlayer = [[VLCMediaPlayer alloc] init];
_mediaPlayer.delegate = self;
_mediaPlayer.drawable = self.view;

/* create a media object and give it to the player */
_mediaPlayer.media = [VLCMedia mediaWithURL:[NSURL URLWithString:@"https://streams.videolan.org/streams/mp4/Mr_MrsSmith-h264_aac.mp4"]];
```

- 默认不会自动播放视频，需要我们手动调用play方法
```ObjC
[_mediaPlayer play];
```


更多装B用法请自己去探索吧，毕竟臣妾也不知道啊[手动笑哭.png]~~~
