---
title: iOS Haptics Feedback (触感反馈)
date: 2018-08-22 14:17:42
tags:
	- iOS
---

## 要求

- iOS 10.0+
- iPhone 7/7P 或更新机型
- 打开"设置--声音与触感--系统触感反馈"选项

## 相关类说明

UIFeedbackGenerator：一个抽象类，不要子类化或创建此类的实例。我们应该使用它的三个子类 (UIImpactFeedbackGenerator, UISelectionFeedbackGenerator, UINotificationFeedbackGenerator)。

| 类 | 说明 |
| :- | :- |
| UIImpactFeedbackGenerator | 提供物理体验，补充动作或任务的视觉反馈。 例如，当视图滑入到位或两个对象发生碰撞时，用户可能会感到砰的一声。 |
| UISelectionFeedbackGenerator | 表示选择正在积极更改。 例如，用户在滚动拾取轮时感觉轻敲。 |
| UINotificationFeedbackGenerator | 表示任务或操作（例如存入支票或解锁车辆）已完成、失败或产生某种警告。 |

## 使用姿势

1. 实例化Generator (UIImpactFeedbackGenerator, UISelectionFeedbackGenerator, UINotificationFeedbackGenerator)
2. 准备工作，调用 `-prepare` 方法
3. 触发触觉反馈，调用相应的触发方法(impactOccurred, selectionChanged, notificationOccurred:)
4. 释放Generator

其中第2、4步可省略。

如有兴趣，可 [点此](https://developer.apple.com/documentation/uikit/uifeedbackgenerator?language=objc) 前往查看官方的示例代码。

## 代码示例

- UIImpactFeedbackGenerator
	-
	```
	UIImpactFeedbackGenerator *feedbackGenerator = [[UIImpactFeedbackGenerator alloc] initWithStyle:UIImpactFeedbackStyleLight];
	[feedbackGenerator impactOccurred];
	```

	可以在实例化 UIImpactFeedbackGenerator 时指定震动反馈的力度，有三个枚举值：
	- UIImpactFeedbackStyleLight 轻微震动
	- UIImpactFeedbackStyleMedium 中度震动
	- UIImpactFeedbackStyleHeavy 重度震动

- UISelectionFeedbackGenerator
	-
	```
	UISelectionFeedbackGenerator *feedbackGenerator = [[UISelectionFeedbackGenerator alloc] init];
	[feedbackGenerator selectionChanged];
	```

- UINotificationFeedbackGenerator
	-
	```
	UINotificationFeedbackGenerator *feedbackGenerator = [[UINotificationFeedbackGenerator alloc] init];
	[feedbackGenerator notificationOccurred:UINotificationFeedbackTypeSuccess];
	```

	UINotificationFeedbackTypeSuccess 和 UINotificationFeedbackTypeWarning 均是震动两次，两者差异不是很大，就我个人感觉而已，UINotificationFeedbackTypeSuccess 的第一下震动力度稍微轻点；UINotificationFeedbackTypeError 则会震动三下，但震动的力度(频率)和每次震动之间的时间间隔也不相同。


## 参考：

- [https://developer.apple.com/library/archive/releasenotes/General/WhatsNewIniOS/Articles/iOS10.html](https://developer.apple.com/library/archive/releasenotes/General/WhatsNewIniOS/Articles/iOS10.html)
- [https://developer.apple.com/documentation/uikit/uifeedbackgenerator?language=objc](https://developer.apple.com/documentation/uikit/uifeedbackgenerator?language=objc)
