---
title: Objective-C/Swift退出多层循环
tags:
  - iOS
abbrlink: a3df8b67
date: 2017-08-01 15:44:11
---

如果有多层嵌套的情况下，有时候我们需要在某处直接退出多层循环，在Objective-C下并没有比较好的方式实现；而在Swfit中可以通过标签语句来比较优雅地实现，首先需要使用标签标识不同的循环体，形式如下：

```Swift
LabelName: while condition {
	statements
}
```

Swift退出多层循环示例代码：

```Swift
let arr = [[1,2], [3,4], [5,6]]
for1: for items in arr {
    for2: for item in items {
        print(item)
        if item == 3 {
            break for1
        }
    }
}
```

通过`break for1`即可退出最外层循环，控制台将依次打印`1、2、3`。

说完了Swift，下面来说说Objective-C下如何实现这种操作。Objective-C既不支持标签语句，也没有类似PHP中`break 2`这种黑科技，但是Objective-C是C的超集，而C语言提供了强大的`goto`语句，虽然很多人都不建议使用goto，但是既然存在就必然有它存在的道理，是不？针对这种情况，goto就很适合。但是还是强烈建议你谨慎使用goto，除非你清楚你的代码在干什么。

Objective-C退出多层循环示例代码：

```ObjC
NSArray<NSArray<NSString *> *> *items = @[@[@"1",@"2"], @[@"3",@"4"], @[@"5",@"6"]];
for (NSArray<NSString *> *array in items) {
    for (NSString *item in array) {
        NSLog(@"%@", item);
        if ([item intValue] == 3) {
            // 跳出外层循环
            goto finally;
        }
    }
}

finally: {
    NSLog(@"goto label.");
}
```

控制台还是会依次打印出1、2、3的值，并且当item的intValue等于3的时候跳出多层for循环，直接执行finally语句块内的代码。

> 使用goto时需要注意：goto语句只能在函数内部进行转移，不能够跨越函数；label代码块的定义位置。

goto语句使用的一般格式为：
```
goto label;
label: { statements }
或者
label: { statements }
goto label;
```

前一种定义方式参见上面的示例，后一种定义方式的代码示例如下：

```ObjC
int i = 0;
first: {
    i++;
    NSLog(@"%d", i);
}
// 代码执行到这里时i等于1，控制台输出1
if (i < 3) {
    goto first;
}
NSLog(@"--end--");
// 控制台将输出1、2、3、--end--
```
