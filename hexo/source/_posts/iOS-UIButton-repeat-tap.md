---
title: 解决 UIButton 快速点击后重复提交数据
date: 2018-09-06 16:03:54
tags:
	- iOS
---

有时候我们快速点击按钮多次，会触发多次点击事件，最终导致数据提交多次，这是很没必要的。

#### 说明

- 默认开启防重复点击功能，默认间隔时间 `0.5` 秒
- 一旦开启该功能，则无法同时触发多次 `UIControlEvents` 的事件，比如按钮同时绑定了 `UIControlEventTouchUpInside` 和 `UIControlEventTouchDownRepeat` 事件，但是只会触发 UIControlEventTouchUpInside 的事件；但是并不影响长按手势以及通过 `UITapGestureRecognizer` 实现的双击手势

总之，如果你的按钮有特殊的需求，可以通过将 `enabledRejectRepeatTap` 设置为 `NO` 来禁用该功能。

#### 代码

- `UIButton+RejectRepeatTapGesture.h` 头文件代码

```ObjC
#import <UIKit/UIKit.h>

@interface UIButton (RejectRepeatTapGesture)

/// 重复点击的时间间隔, 默认`0.5s`
@property (nonatomic, assign) IBInspectable double repeatTimeInterval;
/// 是否启用防重复功能, 默认`YES`
@property (nonatomic, assign, getter=isEnabledRejectRepeatTap) IBInspectable BOOL enabledRejectRepeatTap;

@end


/// 回调, 按钮的`target`对象遵守该协议即可接收回调事件
@protocol UIButtonRejectRepeatTapGestureCallback <NSObject>

/**
 当重复点击时的回调方法

 @param button 被点击的按钮
 @param timeInterval 两次点击之间的时间间隔
 */
- (void)rejectRepeatTapGestureFromButton:(UIButton *)button timeInterval:(NSTimeInterval)timeInterval;

@end
```

- `UIButton+RejectRepeatTapGesture.m` 代码

```ObjC
#import "UIButton+RejectRepeatTapGesture.h"
#import <objc/message.h>

static char const kLasttimeTapTimeKey = '\0';

@implementation UIButton (RejectRepeatTapGesture)

#pragma mark - Lifecycle

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        SEL originalSelector = @selector(sendAction:to:forEvent:);
        SEL swizllingSelector = @selector(rrtg_sendAction:to:forEvent:);
        Method originalMethod = class_getInstanceMethod(self, originalSelector);
        Method swizllingMethod = class_getInstanceMethod(self, swizllingSelector);
        BOOL flag = class_addMethod(self, originalSelector, method_getImplementation(swizllingMethod), method_getTypeEncoding(swizllingMethod));
        if (flag) {
            class_replaceMethod(self, swizllingSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizllingMethod);
        }
    });
}

- (void)rrtg_sendAction:(SEL)action to:(id)target forEvent:(UIEvent *)event {
    if (self.isEnabledRejectRepeatTap) {
    	if (@available(iOS 11.0, *)) {
            // Fix: UITableViewCell left-slide the delete button click is invalid on iOS 11.0 or later
            if (self.class == NSClassFromString(@"UISwipeActionStandardButton")) {
                return [self rrtg_sendAction:action to:target forEvent:event];
            }
        }
        NSTimeInterval lasttimeTapTime = [objc_getAssociatedObject(self, &kLasttimeTapTimeKey) doubleValue];
        if (lasttimeTapTime > 0.0) {
            NSTimeInterval currentTime = [[NSDate date] timeIntervalSince1970];
            NSTimeInterval interval = currentTime - lasttimeTapTime;
            if (interval <= self.repeatTimeInterval) {
                if ([target conformsToProtocol:@protocol(UIButtonRejectRepeatTapGestureCallback)]) {
                    id<UIButtonRejectRepeatTapGestureCallback> delegate = (id<UIButtonRejectRepeatTapGestureCallback>)target;
                    [delegate rejectRepeatTapGestureFromButton:self timeInterval:interval];
                }
                return;
            }
        }
        objc_setAssociatedObject(self,
                                 &kLasttimeTapTimeKey,
                                 @(NSDate.date.timeIntervalSince1970),
                                 OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    }
    [self rrtg_sendAction:action to:target forEvent:event];
}

#pragma mark - setter & getter

- (void)setRepeatTimeInterval:(NSTimeInterval)repeatTimeInterval {
    objc_setAssociatedObject(self,
                             @selector(repeatTimeInterval),
                             @(repeatTimeInterval),
                             OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (NSTimeInterval)repeatTimeInterval {
    NSTimeInterval timeInterval = [objc_getAssociatedObject(self, _cmd) doubleValue];
    if (timeInterval >= 0.01) {
        return timeInterval;
    }
    return 0.5; // default.
}

- (void)setEnabledRejectRepeatTap:(BOOL)enabledRejectRepeatTap {
    objc_setAssociatedObject(self,
                             @selector(isEnabledRejectRepeatTap),
                             @(enabledRejectRepeatTap),
                             OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (BOOL)isEnabledRejectRepeatTap {
    id obj = objc_getAssociatedObject(self, _cmd);
    if (obj) {
        return [obj boolValue];
    }
    return YES; // default.
}

@end
```
