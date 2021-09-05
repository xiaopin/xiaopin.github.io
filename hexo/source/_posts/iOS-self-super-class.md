---
title: '[self class] & [super class]'
abbrlink: 59d33e0c
date: 2021-09-04 22:34:02
tags:
  - Objective-C
  - iOS
  - macOS
---

## 思考

请问以下代码的运行结果是什么？

```ObjC
@interface MYObject : NSObject
@end

@implementation MYObject

- (instancetype)init {
    self = [super init];
    if (self) {
        NSLog(@"%@, %@", self.class, super.class);
    }
    return self;
}

@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        MYObject *obj = [[MYObject alloc] init];
    }
    return 0;
}
```

以上代码将会在控制台输出：
```
MYObject, MYObject
```

## 分析

上述代码为什么打印结果和我们想象中的 `MYObject, NSObject` 不一样呢？想要弄清楚到底发生了什么，那我们就得从底层源码来分析。

### class 方法做了什么？

通过 [objc](https://opensource.apple.com/tarballs/objc4/) 源码可以找到 class 的实现代码如下：

```ObjC
- (Class)class {
    return object_getClass(self);
}
```

通过分析源码得知，通过 `class` -> `object_getClass(self)` -> `getIsa()` -> `ISA()` 这整个执行流程后，已经通过 self 的 isa 指针拿到了当前的类；这就是 class 方法的实现，那么现在的关键在于，class 中的参数 self 到底是谁？？？

### 通过 clang 查看编译后的 C++ 代码

将上面的 OC 代码编译成 C++ 代码：
```shell
clang -rewrite-objc main.m -o main.cpp
```

打开 main.cpp 文件并搜索定位到 init 方法，代码如下：
```C++
static instancetype _I_MYObject_init(MYObject * self, SEL _cmd) {
    self = ((MYObject *(*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){(id)self, (id)class_getSuperclass(objc_getClass("MYObject"))}, sel_registerName("init"));
    if (self) {
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_t2_c8bf6cwj3cz1h771pr1h7nzc0000gn_T_main_404811_mi_0, ((Class (*)(id, SEL))(void *)objc_msgSend)((id)self, sel_registerName("class")), ((Class (*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){(id)self, (id)class_getSuperclass(objc_getClass("MYObject"))}, sel_registerName("class")));
    }
    return self;
}
```

其中 NSLog 的这行代码就是我们的关键所在了，不过这么一大串看起来既不好看也怪吓人的，将其精简一下后，大致等同于如下：
```
NSLog(
    "%@, %@",
    objc_msgSend(self, sel_registerName("class")),
    objc_msgSendSuper(
        (__rw_objc_super){self, class_getSuperclass(objc_getClass("MYObject"))}, 
        sel_registerName("class")
    )
);
```

```ObjC
struct __rw_objc_super { 
	struct objc_object *object; 
	struct objc_object *superClass; 
	__rw_objc_super(struct objc_object *o, struct objc_object *s) : object(o), superClass(s) {} 
};
```

通过上面精简后的 C++ 代码，我们知道 [super class] 其实调用了 `objc_msgSendSuper` 函数。

### objc_msgSendSuper

方法声明如下：

```ObjC
OBJC_EXPORT id _Nullable
objc_msgSendSuper(struct objc_super * _Nonnull super, SEL _Nonnull op, ...)
    OBJC_AVAILABLE(10.0, 2.0, 9.0, 1.0, 2.0);
```

其中第一个参数为 `objc_super` 结构体的指针，该结构体声明如下(已简化)：

```ObjC
struct objc_super {
    __unsafe_unretained _Nonnull id receiver;
    __unsafe_unretained _Nonnull Class super_class;
    /* super_class is the first class to search */
};
```

所以实际上 __rw_objc_super 结构体就等价于 objc_super 结构体，第一个属性就是消息接收者，第二个属性为父类。

所以当我们调用 [super class] 时，底层调用的是 objc_msgSendSuper 方法，并把 self 作为消息接收者，所以打印的还是当前的 MYObject。

### objc_msgSendSuper2

 上面经过编译后的 C++ 代码我们明显看到调用了 objc_msgSendSuper 函数，那么真实的调用是怎样的呢？

在 NSLog 这行代码打个断点，然后查看汇编代码：

![png](/images/2021/09/104.png)

通过汇编代码我们看到，调用的却是 objc_msgSendSuper2 函数，What happened？

继续从 objc 源码中查找蛛丝马迹，找到 objc_msgSendSuper 的实现代码：

![png](/images/2021/09/105.png)

恍然大悟，原来 objc_msgSendSuper 的实现就是调起了 objc_msgSendSuper2 函数。

```ObjC
// objc_msgSendSuper2() takes the current search class, not its superclass.
OBJC_EXPORT id _Nullable
objc_msgSendSuper2(struct objc_super * _Nonnull super, SEL _Nonnull op, ...)
    OBJC_AVAILABLE(10.6, 2.0, 9.0, 1.0, 2.0);
```

如上所示， objc_msgSendSuper2 函数的声明和 objc_msgSendSuper 的声明都是一样的。

但是为何上面断点中的汇编代码，直接能跳转到 objc_msgSendSuper2 而不是通过 objc_msgSendSuper 进行中转调用呢？这其中就是 LLVM 做的优化了，感兴趣可以自己深入研究下。

所以当我们手动调用 objc_msgSendSuper 函数时，实际上调用的还是 objc_msgSendSuper2，这也就和上面的汇编代码流程对上了。

objc_msgSendSuper2 就是一个过渡产物，由于 objc 版本的不断更新，这个过程中产生了 objc_msgSendSuper2 这个产物，但是需要兼容以前的版本，所以 objc_msgSendSuper 还是会保留下来，但是内部会调起 objc_msgSendSuper2；对其我们开发者来说不需要知道太多，继续保持调用 objc_msgSendSuper 就行了。

## 总结

- `[super class]` 等价于 `[self class]`，适用于未重写 class 方法的情况
- `self` 其实是函数的形参，而 `super` 则是编译器关键字，由编译器进行特殊处理
- 如果想获取父类，请通过 `[self superclass]` 获取