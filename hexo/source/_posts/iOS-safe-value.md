---
title: 为 NSArray/NSDictionary 优雅地过滤 nil 值
date: 2018-08-25 16:46:51
tags:
	- iOS
---

作为一名 iOS 开发者，肯定知道 NSArray/NSDictionary 不能存储 `nil` 值，如果你试图往数组/字典中存储 nil，那么 App 也将毫不客气的为你闪退。

尽管在日常的编码中，我们都会小心翼翼的处理 nil，但是总会有纰漏，毕竟大部分数据都是从服务器下发的，我们很难彻底把控。作为一名码农，肯定是想着怎么偷懒的，既能自动规避 nil，又能够不影响现有代码，最好不用引入第三方方法。得益于 Objective-C 的 runtime 机制，我们可以很优雅地通过 `Method Swizlling` 来解决上述问题。

但是我们需要注意一点的是，我们不能直接对 NSArray/NSMutableArray、NSDictionary/NSMutableDictionary 这些类进行 method swizlling 操作，因为它们底层是通过 [`Class cluster`](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/ClassCluster.html) 来实现的，我们需要对隐藏在它们背后的真实的类进行 method swizlling，否则没任何效果。


```ObjC
#import <Foundation/Foundation.h>
#import <objc/message.h>


void safe_swizzle_method(Class originalClass, Class swizzledClass, SEL originalSelector, SEL swizzledSelector)
{
    Method originalMethod = class_getInstanceMethod(originalClass, originalSelector);
    Method swizzledMethod = class_getInstanceMethod(swizzledClass, swizzledSelector);
    
    IMP originalIMP = method_getImplementation(originalMethod);
    IMP swizzledIMP = method_getImplementation(swizzledMethod);
    const char *originalType = method_getTypeEncoding(originalMethod);
    const char *swizzledType = method_getTypeEncoding(swizzledMethod);
    
    class_replaceMethod(originalClass, swizzledSelector, originalIMP, originalType);
    class_replaceMethod(originalClass, originalSelector, swizzledIMP, swizzledType);
}


#pragma mark - Array

@interface XPSafeArray : NSObject

@end

@implementation XPSafeArray

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        NSArray<NSString *> *array1 = @[
                                        NSStringFromSelector(@selector(objectAtIndex:)),
                                        NSStringFromSelector(@selector(objectAtIndexedSubscript:))
                                        ];
        for (NSString *str in array1) {
            safe_swizzle_method(NSClassFromString(@"__NSArrayI"), self,
                                NSSelectorFromString(str),
                                NSSelectorFromString([@"safe_" stringByAppendingString:str]));
        }
        
        NSArray<NSString *> *array2 = @[
                                        NSStringFromSelector(@selector(objectAtIndex:)),
                                        NSStringFromSelector(@selector(objectAtIndexedSubscript:)),
                                        NSStringFromSelector(@selector(insertObject:atIndex:)),
                                        NSStringFromSelector(@selector(setObject:atIndexedSubscript:)),
                                        NSStringFromSelector(@selector(insertObjects:atIndexes:))
                                        ];
        for (NSString *str in array2) {
            safe_swizzle_method(NSClassFromString(@"__NSArrayM"), self,
                                NSSelectorFromString(str),
                                NSSelectorFromString([@"safe_" stringByAppendingString:str]));
        }
    });
}

- (id)safe_objectAtIndex:(NSUInteger)index {
    NSUInteger count = [(NSArray*)self count];
    if (count == 0 || index >= count) {
        return nil;
    }
    return [self safe_objectAtIndex:index];
}

- (id)safe_objectAtIndexedSubscript:(NSUInteger)index {
    NSUInteger count = [(NSArray*)self count];
    if (count == 0 || index >= count) {
        return nil;
    }
    return [self safe_objectAtIndexedSubscript:index];
}

- (void)safe_insertObject:(id)anObject atIndex:(NSUInteger)index {
    if (anObject == nil) return;
    [self safe_insertObject:anObject atIndex:index];
}

- (void)safe_setObject:(id)obj atIndexedSubscript:(NSUInteger)idx {
    if (obj == nil) return;
    [self safe_setObject:obj atIndexedSubscript:idx];
}

- (void)safe_insertObjects:(NSArray *)objects atIndexes:(NSIndexSet *)indexes {
    if (objects && objects.count == indexes.count) {
        [self safe_insertObjects:objects atIndexes:indexes];
    }
}

@end


#pragma mark - Dictionary

@interface XPSafeDictionary : NSDictionary

@end

@implementation XPSafeDictionary

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        safe_swizzle_method(NSClassFromString(@"__NSPlaceholderDictionary"),
                            self,
                            @selector(initWithObjects:forKeys:count:),
                            @selector(safe_initWithObjects:forKeys:count:));
        
        safe_swizzle_method(NSClassFromString(@"__NSDictionaryM"),
                            self,
                            @selector(setObject:forKey:),
                            @selector(safe_setObject:forKey:));
        
        safe_swizzle_method(NSClassFromString(@"__NSDictionaryM"),
                            self,
                            @selector(removeObjectForKey:),
                            @selector(safe_removeObjectForKey:));
    });
}

- (id)safe_initWithObjects:(id  _Nonnull const [])objects forKeys:(id<NSCopying>  _Nonnull const [])keys count:(NSUInteger)cnt {
    id safeObjects[cnt];
    id safeKeys[cnt];
    NSUInteger count = 0;
    
    for (NSUInteger idx=0; idx<cnt; idx++) {
        id key = keys[idx];
        id obj = objects[idx];
        
        if (!key) {
            continue;
        }
        if (!obj) {
            obj = [NSNull null]; // 可根据你项目需求，将`NSString`作为默认值
        }
        
        safeKeys[count] = key;
        safeObjects[count] = obj;
        count++;
    }
    return [self safe_initWithObjects:safeObjects forKeys:safeKeys count:count];
}

- (void)safe_setObject:(id)anObject forKey:(id<NSCopying>)aKey {
    if (anObject && aKey) {
        [self safe_setObject:anObject forKey:aKey];
    }
}

- (void)safe_removeObjectForKey:(id)aKey {
    if (aKey) {
        [self safe_removeObjectForKey:aKey];
    }
}

@end
```

以上只给出部分实现，但是已基本满足日常使用，你可以自己根据自己的需求进行完善。
