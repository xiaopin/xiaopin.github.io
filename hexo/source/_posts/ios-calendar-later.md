---
title: iOS获取一年后的时间
date: 2016-12-08 13:50:10
tags:
	- iOS
---

虽然说是获取一年后的时间，但是可以通过设置不同的数值，达到获取N年前或N年后的时间的效果。
当`数值>0`时，获取之后的时间，如果`数值<0`则为获取之前的时间。

``` Objective-C
NSDate *date = [NSDate date];
NSCalendar *calendar = [NSCalendar currentCalendar];
NSDateComponents *components = [[NSDateComponents alloc] init];
[components setYear:1];
[components setMonth:0];
[components setDay:0];
// laterDate则为一年后的时间
NSDate *laterDate = [calendar dateByAddingComponents:components toDate:date options:NSCalendarWrapComponents];
```

善于使用`NSDateComponents`提供的属性，可以计算出你想要的任何时间，N年前、N年后、N个月前、N个月后、N天前、N天后...

``` Objective-C
@property NSInteger year;
@property NSInteger month;
@property NSInteger day;
@property NSInteger hour;
@property NSInteger minute;
@property NSInteger second;
```