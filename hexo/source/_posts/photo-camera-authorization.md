---
title: iOS判断相册、摄像头权限
date: 2017-03-22 14:30:48
tags:
	- iOS
---

## 摄像头权限判断

#### 判断设备是否支持摄像头
``` ObjC
if (![UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeCamera]) {
	// 不支持摄像头
}
```


#### 判断用户是否授权访问摄像头
``` ObjC
#import <AVFoundation/AVFoundation.h>
AVAuthorizationStatus status = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeVideo];
if (status == AVAuthorizationStatusDenied || status == AVAuthorizationStatusRestricted) {
	// 没有授权访问摄像头
}
```


## 判断系统相册权限

#### 判断相册是否可用的
``` ObjC
[UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypePhotoLibrary]
[UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeSavedPhotosAlbum]
```

#### 判断是否授权访问相册

###### PhotoKit，iOS8之后新增的Framework，推荐使用该方式
``` ObjC
#import <Photos/Photos.h>
PHAuthorizationStatus status = [PHPhotoLibrary authorizationStatus];
if (status == PHAuthorizationStatusDenied || status == PHAuthorizationStatusRestricted) {
	// 没有授权访问相册
}
```

###### 如果需要兼容iOS7，可以使用AssetsLibrary库进行判断
``` ObjC
#import <AssetsLibrary/AssetsLibrary.h>
ALAuthorizationStatus status = [ALAssetsLibrary authorizationStatus];
if (status == ALAuthorizationStatusDenied || status == ALAuthorizationStatusRestricted) {
    // 没有授权访问相册
}
```

## 访问相册示例代码
``` ObjC
#import <Photos/Photos.h>

PHAuthorizationStatus status = [PHPhotoLibrary authorizationStatus];
if (status == PHAuthorizationStatusDenied || status == PHAuthorizationStatusRestricted) {
    UIAlertController *controller = [UIAlertController alertControllerWithTitle:@"没有授权访问系统相册" message:@"是否立即前往设置页面打开授权？" preferredStyle:UIAlertControllerStyleAlert];
    [controller addAction:[UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:nil]];
    [controller addAction:[UIAlertAction actionWithTitle:@"前往设置" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        // 打开App系统设置页面
        NSURL *url = [NSURL URLWithString:UIApplicationOpenSettingsURLString];
        if ([[UIApplication sharedApplication] canOpenURL:url]) {
            [[UIApplication sharedApplication] openURL:url];
        }
    }]];
    [self presentViewController:controller animated:YES completion:nil];
} else if (status == PHAuthorizationStatusNotDetermined) {
    // 第一次使用，用户还未决定是否授权，此时应该请求用户授权获取权限
    [PHPhotoLibrary requestAuthorization:^(PHAuthorizationStatus status) {
       // 授权回调
    }];
} else if (status == PHAuthorizationStatusAuthorized) {
    // 用户已授权访问,做你想做的事
}
```
