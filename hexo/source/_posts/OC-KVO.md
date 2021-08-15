---
title: Key-Value Observing(KVO)
tags:
  - Objective-C
  - iOS
  - macOS
abbrlink: 748a8892
date: 2021-07-30 21:23:02
---

## 关于 KVO

键值观察是一种机制，它允许对象在其他对象的指定属性发生更改时收到通知。

> Key-value observing is a mechanism that allows objects to be notified of changes to specified properties of other objects.

## KVO 初探

### KVO 的使用

- 通过 `-addObserver:forKeyPath:options:context:` 接口注册通知
- 实现 `-observeValueForKeyPath:ofObject:change:context:` 方法接收通知的回调
- 最后记得移除监听(`-removeObserver:forKeyPath:` / `-removeObserver:forKeyPath:context:`)

```ObjC
- (void)viewDidLoad {
    [super viewDidLoad];
    [self.person addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew|NSKeyValueObservingOptionOld context:NULL];
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context{
    // 此处进行业务逻辑处理
}

- (void)dealloc{
    [self.person removeObserver:self forKeyPath:@"name" context:NULL];
}
```

### KVO 开关控制

在 [KVO Compliance](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/Articles/KVOCompliance.html#//apple_ref/doc/uid/20002178-SW1) 这一章节内容中有提到，KVO 除了自动发送通知之外，还可以通过 `+automaticallyNotifiesObserversForKey:` 接口手动进行控制。

- 返回 NO 禁用自动通知

```ObjC
+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key {
    if ([key isEqualToString:@"name"]) {
        return NO; // 返回 NO 则不会触发自动通知
    }
    return [super automaticallyNotifiesObserversForKey:key];;
}
```

- 通过在更改值之前调用 `willChangeValueForKey:` 和 在更改值之后调用 `didChangeValueForKey:` 来完成手动通知

```ObjC
- (void)setName:(NSString *)name {
    [self willChangeValueForKey:@"name"];
    _name = name;
    [self didChangeValueForKey:@"name"];
}
```

> 如果 `automaticallyNotifiesObserversForKey:` 中返回了 `NO` 并且在 setter 中未手动调用 `willChangeValueForKey:` 和 `didChangeValueForKey:`，则不会触发通知，观察者也就无法观察到值的变化。

### Options

官方文档中有对 [Options](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/Articles/KVOBasics.html#//apple_ref/doc/uid/20002252-SW3) 的详细说明。

### Context

官方文档中有对 [Context](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/Articles/KVOBasics.html#//apple_ref/doc/uid/20002252-SW4) 的详细说明。

`addObserver:forKeyPath:options:context:` 消息的上下文指针可以包含任意数据，这些数据将在相应的更改通知中回传给观察者。您可以指定 `NULL` 并完全依赖键路径字符串来确定更改通知的来源，但是这种方式可能会出现意外情况从而导致程序的不安全性，比如在父类中由于某种原因也在观察相同 keyPath 的对象。

一种更安全、更可扩展的方法是使用上下文来确保您收到的通知是发送给您的观察者而不是父类的。

类中唯一命名的静态变量的地址是一个很好的上下文，在父类或子类中以类似方式选择的上下文不太可能重叠。您可以为整个类选择一个上下文，并依靠通知消息中的关键路径字符串来确定发生了什么变化。或者，您可以为每个观察到的键路径创建一个不同的上下文，这完全绕过了字符串比较的需要，从而提高了通知解析的效率。

1. 定义上下文

```ObjC
static void *PersonNameContext = &PersonNameContext;
static void *PersonAgeContext = &PersonAgeContext;
```

2. 注册通知时传递上下文

```ObjC
[self.person addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew|NSKeyValueObservingOptionOld context:PersonNameContext];
[self.person addObserver:self forKeyPath:@"age" options:NSKeyValueObservingOptionNew|NSKeyValueObservingOptionOld context:PersonAgeContext];
```

3. 在接收到更改通知时进行处理

```ObjC
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    if (context == PersonNameContext) {
        ...
    } else if (context == PersonAgeContext) {
        ...
    }
}
```

> 苹果推荐我们使用 `context` 上下文来标识通知来源，而不是依靠 `keyPath` 来进行区分。

### 在 NSMutableArray 中使用 KVO

在可变数组中直接通过 `[mArray addObject:obj]` 修改数组内容，观察者是无法观察到值的变化的；必须使用 KVC 中的 `mutableArrayValueForKey:` 方法获取到可变数组实例然后再调用 `addObject:` 方法。

![png](/images/2021/07/217.png)

### 注册依赖键

在许多情况下，一个属性的值取决于一个或多个其他属性的值，如果其中一个值发生变化，那么这个属性的值也应该随之变化。

例如：一个人有姓名和年龄，他还会自动跟人打招呼(嗨，我是xxx，我今年xxx岁了)，那么打招呼的时候就依赖了姓名和年龄的值。

对于一对一的关系，我们可以通过重写 `+keyPathsForValuesAffectingValueForKey:` 来返回对应的依赖关系即可。

![png](/images/2021/07/218.png)

[Registering Dependent Keys](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/Articles/KVODependentKeys.html#//apple_ref/doc/uid/20002179-BAJEAIEE) 这一章节内容详细介绍了注册依赖键的使用。其中还介绍了一对多的情况下如何注册依赖，感兴趣可以自行查阅。

## KVO 原理分析

[Key-Value Observing Implementation Details](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/Articles/KVOImplementation.html#//apple_ref/doc/uid/20002307-BAJEAIEE) 中说明了 KVO 是基于 `isa-swizzling` 进行实现的。

### isa 的指向

文档明确说明了会修改 isa 的指向，那么我们就来验证一下。

- 通过代码验证

![png](/images/2021/07/210.png)

- 通过 LLDB 进行验证

![png](/images/2021/07/210-1.png)

通过上图可以明确看出，当执行了 `addObserver:forKeyPath:options:context:` 方法后，isa 的指向从 `MYPerson` 改成了 `NSKVONotifying_MYPerson`。那么这个类是何时生成的呢，是编译时还是运行时？

![png](/images/2021/07/211.png)

在 `addObserver:forKeyPath:options:context:` 这行代码打个断点，分别在这个方法运行前后打印下 NSKVONotifying_MYPerson 这个类，结果如上图所示，这也充分说明了这个类是在运行时动态生成的。

### NSKVONotifying_MYPerson 与 MYPerson 有何关联？

那么这个新生成的 NSKVONotifying_MYPerson 和原来的 MYPerson 之间有何关联呢？你说他们是继承关系，又如何证明呢？

首先 `getSubClasses:` 方法可以获取到某个类的所有子类并返回，我们在 `addObserver:forKeyPath:options:context:` 方法调用前后分别获取了 MYPerson 的子类，发现当该方法执行完之后， MYPerson 的子类列表中多了一个 NSKVONotifying_MYPerson，同时，NSKVONotifying_MYPerson 并没有任何子类。

> NSKVONotifying_MYPerson 继承自 MYPerson，他们是父子关系。

![png](/images/2021/07/212.png)

```ObjC
- (NSArray *)getSubClasses:(Class)cls{
    int count = objc_getClassList(NULL, 0);
    NSMutableArray *mArray = [NSMutableArray array];
    Class* classes = (Class*)malloc(sizeof(Class)*count);
    objc_getClassList(classes, count);
    for (int i = 0; i < count; i++) {
        if (cls == class_getSuperclass(classes[i])) {
            [mArray addObject:classes[i]];
        }
    }
    free(classes);
    return [NSArray arrayWithArray:mArray];
}
```

### NSKVONotifying_MYPerson 都有哪些方法？

![png](/images/2021/07/213.png)

通过上图可以明确看出新生成的类会有以下方法：
- 被观察的属性 setter（如这里的 setName: 方法）
  - 重写 setter 方法，内部调用 `willChangeValueForKey:` 和 `didChangeValueForKey:` 并且会调用父类的 setter
- class
  - 用于返回 MYPerson 这个类，让外界感知不到 NSKVONotifying_MYPerson 的存在
- dealloc
  - 将 isa 指向修正回来
- _isKVOA

getClassMethods: 方法可以获取到类的所有方法(仅包含自身的方法，不含父类)，源码如下：

```ObjC
- (NSArray *)getClassMethods:(Class)cls{
    unsigned int count = 0;
    NSMutableArray *mArray = [NSMutableArray array];
    Method *methods = class_copyMethodList(cls, &count);
    for (int i = 0; i < count; i++) {
        Method method = methods[i];
        SEL sel = method_getName(method);
        IMP imp = class_getMethodImplementation(cls, sel);
        NSString *str = [NSString stringWithFormat:@"%@, %p", NSStringFromSelector(sel), imp];
        [mArray addObject:str];
    }
    free(methods);
    return [NSArray arrayWithArray:mArray];
}
```

### setter

NSKVONotifying_MYPerson 类重写了相应的 setter 方法，那么 setter 中又做了什么操作呢？

通过 `watchpoint` 命令下断点进行观察

![png](/images/2021/07/214.png)

点击屏幕，修改 name 的值以触发通知，然后查看相关信息

![png](/images/2021/07/215.png)

### dealloc

在移除观察的前后分别打印了 self.person，发现当 `removeObserver:forKeyPath:` 方法调用完毕之后，self.person 的 isa 又重新指回了 MYPerson

![png](/images/2021/07/216.png)

## 自定义 KVO

了解了 KVO 的实现原理后，我们就可以按照流程来实现自己的 KVO，这里不给出代码，有兴趣的可以自行研究由 Facebook 出品的 [KVOController](https://github.com/facebook/KVOController)(FBKVO)

## KVO 与 Notification 的区别

KVO 和 Notification 都是 iOS 中观察者模式的一种实现，区别在于：
- 相对于被观察者和观察者之间的关系，KVO 是一对一的关系，也就是 KVO 监听到被观察属性值改变时只会通知到观察者，是一对一的关系；而通知模式则是在被观察值改变的时候发送全局通知，任何对象都可以接收到这个通知，是一个一对多的关系
- KVO 对被监听对象无侵入性，不需要修改其内部代码即可实现监听；而通知需要在被监听对象改变的时候添加发送通知的代码

## 总结

当我们通过 `addObserver:forKeyPath:options:context:` 增加一个监听的时候，大致会有这么个流程：

1. 首先内部会动态生成一个 NSKVONotifiy_OriginalClassName 的子类
2. 然后重写相应的 setter/class/dealloc 方法，并新增一个 _isKVOA 方法
3. 在 setter 中通过 runtime 消息转发把消息发送给父类(调用父类的 setter 进行赋值)
4. 将被观察者的 isa 指向动态生成的类实例

当通过 `removeObserver:forKeyPath:` 移除观察时：

1. 修正 isa 的指向，将其重新指回被观察者

## 参考

- [Key-Value Observing Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html#//apple_ref/doc/uid/10000177i)
