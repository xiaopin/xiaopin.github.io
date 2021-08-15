---
title: GCD面试题分享
tags:
  - Objective-C
  - iOS
  - macOS
abbrlink: c43fa08b
date: 2021-08-03 22:04:57
---

> GCD 老生常谈的话题，下面分享几道面试题。

## 某团面试题

```ObjC
@property (nonatomic, assign) int num;

- (void)MT {
    while (self.num < 5) {
        dispatch_async(dispatch_get_global_queue(0, 0), ^{
            self.num++;
        });
    }
    NSLog(@"%d", self.num);
}
```

问：上面的打印结果可能为什么？

答：打印结果为 `>=5`。

分析：
1. 当首次进入 while 循环时，`self.num == 0`，此时会创建一个异步任务，但是这个任务并不一定是立马得到执行，会等待线程的调度；此时就会进入下一次的循环，由于任务未执行，所以 self.num++ 也就未执行，所以第二次循环时，self.num 还是等于 0，此时又会再次创建一个新的异步任务；以此类推，至少都会创建 5 条异步任务
2. 由于 while 循环中使用了 `self.num < 5` 作为终止条件，所以只有当 `self.num >= 5` 时才能跳出循环，执行打印
3. 在跳出循环执行打印的中间时间，可能存在多个异步任务被执行了，也就相当于 `self.num++` 被执行了 N 次
4. 所以打印结果必须 `>=5`

> 扩展：如果把 dispatch_async 换成 dispatch_sync 同步任务，则打印结果必定为 5，因为同步任务会堵塞线程， self.num++ 没执行完毕之前，下一次循环就不可能执行。

## 某手面试题

```ObjC
@property (nonatomic, assign) int num;

- (void)KS {
    for (int i = 0; i < 10000; i++) {
        dispatch_async(dispatch_get_global_queue(0, 0), ^{
            self.num++;
        });
    }
    NSLog(@"%d", self.num);
}
```

问：上面的打印结果可能为什么？

答：打印结果为 `0 ~ 9999`。

分析：这道题和上面的有异曲同工之处，但是考察的点又不一样
1. 这里使用 for 循环，并且用了临时变量 `i` 而不是 `self.num` 来进行判断，首先 self.num 和循环就没关系了，所以这里只会执行 9999 次循环，也就是创建了 9999 个异步任务
2. 当 `i == 10000` 时，for 循环终止，此时虽然创建了 9999 个异步任务，但是这些任务都执行完毕了吗？显然没有，需要等待线程来调度；所以在打印之前，到底有多少个任务能够被执行呢，这个不确定，可能一个任务都没执行，也可能全部都执行完毕，那么打印结果就有可能是 `0 ~ 9999`
3. 虽然理论上是 0 ~ 9999，但是打印结果还是会趋近于 9999

## 某博面试题

```ObjC
- (void)WBDemo1 {
    dispatch_queue_t queue = dispatch_queue_create("com.demo", DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(queue, ^{ NSLog(@"1"); });
    dispatch_async(queue, ^{ NSLog(@"2"); });
    dispatch_sync(queue, ^{ NSLog(@"3"); });

    NSLog(@"0");

    dispatch_async(queue, ^{ NSLog(@"4"); });
    dispatch_async(queue, ^{ NSLog(@"5"); });
    dispatch_async(queue, ^{ NSLog(@"6"); });
}
```

问：上面的打印结果可能为什么？
- A: 1230456
- B: 1234560
- C: 3120465
- D: 2134560

答：A和C

分析：
1. 任务3为同步任务，同步任务有个特性就是会阻塞线程的执行(护犊子，自己的任务未完成，后面的任务休想执行)，所以在任务3未执行完毕之前，任务0以及之后的任务都会处于阻塞状态，所以 3 必定在 0 之前进行打印
2. 然后根据函数的按顺序执行的规律，任务0必定会在 4、5、6 之前进行打印
3. 异步任务之间是无序的，所以 1、2、3 之间顺序不定，4、5、6 之间顺序也是不定
4. 所以这里的打印结果必须满足两个条件：
  - 3 在 0 之前
  - 0 在 4、5、6 之前

对于 A、B、C、D 来说，只有 A、C 符合上述两个条件。

> 如果 queue 是 DISPATCH_QUEUE_SERIAL 类型的串行队列的话则答案是 A

***

```ObjC
- (void)WBDemo2 {
    dispatch_queue_t queue = dispatch_queue_create("com.demo", DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(queue, ^{ NSLog(@"1"); });
    dispatch_async(queue, ^{ NSLog(@"2"); });
    dispatch_barrier_async(queue, ^{ NSLog(@"3"); });
    dispatch_async(queue, ^{ NSLog(@"4"); });
}
```

问：上面的打印结果可能为什么？

答：1234 或 2134

分析：
1. queue 是一个 DISPATCH_QUEUE_CONCURRENT 类型的并发队列，所以任务1和任务2顺序是不定的
2. 任务3使用了 dispatch_barrier_async 栅栏函数来创建任务，它有个特点，就是会将前后任务进行隔离，任务1和任务2必须都执行完了才会执行任务3，任务4必须等任务3执行完了才能执行
3. 所以任务3在任务1和任务2之后，但是在任务4之前

## Other

```ObjC
- (void)demo1 {
    dispatch_queue_t queue = dispatch_queue_create("com.demo", DISPATCH_QUEUE_CONCURRENT);
    NSLog(@"1");
    dispatch_async(queue, ^{
        NSLog(@"2");
        dispatch_async(queue, ^{ NSLog(@"3"); });
        NSLog(@"4");
    });
    NSLog(@"5");
}
```

问：上面的打印结果是什么？

答：15243

分析：这个应该没什么疑问吧
1. demo1 按顺序执行，先打印 1 这个没问题吧
2. dispatch_async 创建了异步任务，在后台等待执行，继续往下执行打印5
3. 异步任务被调用后，先打印2，然后创建一个新异步任务等待调度执行，继续往下执行打印4
4. 最后打印3

***

```ObjC
- (void)demo2 {
    dispatch_queue_t queue = dispatch_queue_create("com.demo", DISPATCH_QUEUE_CONCURRENT);
    NSLog(@"1");
    dispatch_async(queue, ^{
        NSLog(@"2");
        dispatch_sync(queue, ^{ NSLog(@"3"); });
        NSLog(@"4");
    });
    NSLog(@"5");
}
```

问：上面的打印结果是什么？

答：15234

分析：
1. demo2 按顺序执行，先打印 1
2. dispatch_async 创建了异步任务，在后台等待执行，继续往下执行打印 5
3. 异步任务被调用后，先打印 2
4. 创建一个同步任务堵塞线程，等待同步任务执行完毕后才能继续往下执行，所以打印 3
5. 最后打印 4

***

```ObjC
- (void)demo3 {
    dispatch_queue_t queue = dispatch_queue_create("com.demo", NULL);
    NSLog(@"1");
    dispatch_async(queue, ^{
        NSLog(@"2");
        dispatch_sync(queue, ^{ NSLog(@"3"); });
        NSLog(@"4");
    });
    NSLog(@"5");
}
```

问：上面的打印结果是什么？

答：152蹦溃

分析：
1. demo3 按顺序执行，先打印 1
2. dispatch_async 创建了异步任务，在后台等待执行，继续往下执行打印 5
3. 异步任务被调用后，先打印 2
4. 创建一个同步任务3，由于 queue 是串行队列，任务3需要等待异步任务执行完毕才能执行，但是异步任务又需要等待任务3执行完毕后才能继续往下执行打印4，你依赖我我依赖你，这里就造成了死锁，最终导致 `EXC_BAD_INSTRUCTION` 异常错误