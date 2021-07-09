---
title: objc_class 中对 cache 属性的初探
abbrlink: cab24a44
date: 2021-07-08 15:32:01
tags:
    - Objective-C
    - iOS
    - macOS
---

今天我们来探索下类的 `cache` 属性，cache 顾名思义就是 `缓存`，那么它到底缓存的是什么数据呢？

## cache 存的是什么？

源码在手，天下我有！

那我们就直接从 [objc](https://opensource.apple.com/tarballs/objc4/) 源码下手，跟着源码一直溯源，看看有没有什么蛛丝马迹。

- 先来看下类的定义：objc_class

```ObjC
struct objc_class : objc_object {
    // Class ISA;
    Class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits; 
    ...
};

struct objc_object {
private:
    isa_t isa;
    ...
};

union isa_t {
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    uintptr_t bits;

private:
    // Accessing the class requires custom ptrauth operations, so
    // force clients to go through setClass/getClass by making this
    // private.
    Class cls;
    
    ...
};
```

- cache_t

```ObjC
struct cache_t {
private:
    explicit_atomic<uintptr_t> _bucketsAndMaybeMask;
    union {
        struct {
            explicit_atomic<mask_t>    _maybeMask;
#if __LP64__
            uint16_t                   _flags;
#endif
            uint16_t                   _occupied;
        };
        explicit_atomic<preopt_cache_t *> _originalPreoptCache;
    };

public:
    struct bucket_t *buckets() const;
    void insert(SEL sel, IMP imp, id receiver);
    ...
};
```

通过查看 cache_t 的源码，发现基本在围绕着 `bucket_t` 这个数据做一些~不可描述~的事情。

- bucket_t

```ObjC
struct bucket_t {
private:
    // IMP-first is better for arm64e ptrauth and no worse for arm64.
    // SEL-first is better for armv7* and i386 and x86_64.
#if __arm64__
    explicit_atomic<uintptr_t> _imp;
    explicit_atomic<SEL> _sel;
#else
    explicit_atomic<SEL> _sel;
    explicit_atomic<uintptr_t> _imp;
#endif
    ...
};
```

顺着 objc_class --> cache_t --> bucket_t 这条线索，我们发现了两个关键的东西 `_sel` 和 `_imp`，那是不是就如我们看到的那样，cache 中缓存的就是方法呢？？？

## 验证猜测

老规矩，上测试代码：

<!-- ![](/images/2021/07/103.png) -->

```ObjC
@interface MYObject : NSObject
- (void)sayHello;
+ (void)say666;
@end

@implementation MYObject
- (void)sayHello {
    NSLog(@"hello");
}
+ (void)say666 {
    NSLog(@"666");
}
@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        MYObject *obj = [[MYObject alloc] init];
        [obj sayHello];
        NSLog(@"---"); // 此处下断点
    }
    return 0;
}
```

为了验证 cache 是否存储了成员方法，那我们就需要先拿到 bucket_t 这个数据。

- 获取 cache_t

![](/images/2021/07/104.png)

- 获取 bucket_t 并打印 _sel 和 _imp 所对应的数据

![](/images/2021/07/105.png)

以下为详细的调试Log：

```
(lldb) p/x MYObject.class
(Class) $0 = 0x0000000100008130 MYObject
(lldb) p/x (cache_t *)(0x0000000100008130 + 0x10)
(cache_t *) $1 = 0x0000000100008140
(lldb) p *$1
(cache_t) $2 = {
  _bucketsAndMaybeMask = {
    std::__1::atomic<unsigned long> = {
      Value = 4316096928
    }
  }
   = {
     = {
      _maybeMask = {
        std::__1::atomic<unsigned int> = {
          Value = 3
        }
      }
      _flags = 32784
      _occupied = 2
    }
    _originalPreoptCache = {
      std::__1::atomic<preopt_cache_t *> = {
        Value = 0x0002801000000003
      }
    }
  }
}
(lldb) p $2.buckets()
(bucket_t *) $3 = 0x00000001014269a0
(lldb) p $3[1]
(bucket_t) $4 = {
  _sel = {
    std::__1::atomic<objc_selector *> = "" {
      Value = ""
    }
  }
  _imp = {
    std::__1::atomic<unsigned long> = {
      Value = 3510592
    }
  }
}
(lldb) p $4.sel()
(SEL) $5 = "init"
(lldb) p $4.imp(nil, MYObject.class)
(IMP) $6 = 0x0000000100351070 (libobjc.A.dylib`-[NSObject init] at NSObject.mm:2557)
(lldb) p $3[2]
(bucket_t) $7 = {
  _sel = {
    std::__1::atomic<objc_selector *> = "" {
      Value = ""
    }
  }
  _imp = {
    std::__1::atomic<unsigned long> = {
      Value = 49072
    }
  }
}
(lldb) p $7.sel()
(SEL) $8 = "sayHello"
(lldb) p $7.imp(nil, MYObject.class)
(IMP) $9 = 0x0000000100003e80 (MYTest`-[MYObject sayHello] at main.m:21)
(lldb) 
```

> 结论：通过上面的验证，可以确定 cache 中缓存的就是 **成员方法**

<!-- ## 扩容时机 -->

## 补充说明

- 为什么通过内存平移 `0x10` 后就能直接拿到 cache

如果你仔细看前面关于 objc_class 的定义的话，那么你应该就会发现了，objc_class 从 objc_object 中继承了一个 `isa` 属性，然后 cache 前面还有一个 `superclass` 属性，而 isa 和 superclass 都各占 8 字节，加起来就刚好是 16 字节也就是 `0x10`

- 成员方法在缓存中都是懒加载的

你可以把 `[obj sayHello]` 这行代码注释掉，再来走一遍调试流程，你会发现根本就找不到这个 `sayHello` 方法，只有一个 `init` 方法，感兴趣的你可以自己研究研究
