---
title: Key-Value Coding(KVC)
abbrlink: 7d3cc4b9
date: 2021-07-27 14:37:06
tags:
  - Objective-C
  - iOS
  - macOS
---

## 关于 KVC

键值编码是由 `NSKeyValueCoding` 非正式协议启用的一种机制，对象采用该机制提供对其属性的间接访问。当对象符合键值编码要求时，其属性可以通过字符串参数通过简洁、统一的消息接口寻址。这种间接访问机制补充了实例变量及其相关访问器方法提供的直接访问。

> Key-value coding is a mechanism enabled by the NSKeyValueCoding informal protocol that objects adopt to provide indirect access to their properties. When an object is key-value coding compliant, its properties are addressable via string parameters through a concise, uniform messaging interface. This indirect access mechanism supplements the direct access afforded by instance variables and their associated accessor methods.

您通常使用访问器方法来访问对象的属性，通过获取访问器（或 getter 方法）返回属性的值，通过设置访问器（或 setter）设置属性的值。在Objective-C中，您还可以直接访问属性的基础实例变量。以任何一种方式访问对象属性很简单，但需要调用特定于属性的方法或变量名。随着属性列表的增加或更改，访问这些属性的代码也必须增加或更改。相比之下，符合键值编码的对象提供了一个简单的消息传递接口，该接口在其所有属性中保持一致。

> You typically use accessor methods to gain access to an object’s properties. A get accessor (or getter) returns the value of a property. A set accessor (or setter) sets the value of a property. In Objective-C, you can also directly access a property’s underlying instance variable. Accessing an object property in any of these ways is straightforward, but requires calling on a property-specific method or variable name. As the list of properties grows or changes, so also must the code which accesses these properties. In contrast, a key-value coding compliant object provides a simple messaging interface that is consistent across all of its properties.

键值编码是一个基本概念，是许多其他 `Cocoa` 技术的基础，例如键值观察(KVO)、Cocoa 绑定、Core Data和AppleScript-ability。在某些情况下，键值编码还可以帮助简化您的代码。

> Key-value coding is a fundamental concept that underlies many other Cocoa technologies, such as key-value observing, Cocoa bindings, Core Data, and AppleScript-ability. Key-value coding can also help to simplify your code in some cases

***

KVC 在 Objective-C 中其实就是一个分类，通过分类定义了一些 API 以供我们使用，由于其是对 NSObject 进行的扩展，所以 NSObject 以及所有继承于 NSObject 的子类都默认拥有 KVC 功能。

![KVC](/images/2021/07/201.png)

KVC 常用的接口说明：

```ObjC
- (nullable id)valueForKey:(NSString *)key; // 通过 key 取值
- (void)setValue:(nullable id)value forKey:(NSString *)key; // 通过 key 设值
- (nullable id)valueForKeyPath:(NSString *)keyPath; // 通过 keyPath 取值
- (void)setValue:(nullable id)value forKeyPath:(NSString *)keyPath; // 通过 keyPath 设值
```

## 设值过程

可以查阅官方文档关于 [Search Pattern for the Basic Setter](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/SearchImplementation.html#//apple_ref/doc/uid/20000955-CJBBBFFA) 的相关说明。

下面介绍关于 `setValue:forKey:` 的默认实现，传递 key 和 value 两个参数，并尝试把名为 key 的属性值设置为 value，在接收调用的对象内部，按照以下顺序进行处理：

- 按顺序查找 `set<Key>:` 和 `_set<Key>:` 以及 `setIs<Key>:` 方法，如果查找到相应方法，则将 value 作为参数进行传递并调用（如果 value 需要进行解包操作，则会进行相应的处理， 如：将 NSNumber 类型转成 int 类型等）
- 如果未找到上述方法，并且类方法 `+ accessInstanceVariablesDirectly` 返回 `YES`，则按顺序查找名称类似于 `_<key>`、`_is<Key>`、`<key>` 或 `is<Key>` 的实例变量。 如果找到，直接使用 value（或解包值）设置变量并完成
- 在未找到 setter 或实例变量时，调用 `setValue:forUndefinedKey:` 方法（该方法默认会抛出异常）

下面给出整个 setter 流程图：

![](/images/2021/07/KVC-setter.png)

## 取值过程

可以查阅官方文档关于 [Search Pattern for the Basic Getter](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/SearchImplementation.html#//apple_ref/doc/uid/20000955-CJBBBFFA) 的相关说明。

下面介绍关于 `valueForKey:` 的默认实现：

1. 按顺序查找 `get<Key>`、`<key>`、`is<Key>` 或 `_<key>` 方法，如果查找到相应方法，则调用方法并返回结果，然后执行第 5 步
2. 查找 `countOf<Key>` 、`objectIn<Key>AtIndex:` 以及 `<key>AtIndexes:` 方法，如果找到第一个和其他两个中的其中一个，则创建一个集合代理对象（该对象响应所有 NSArray 方法）并返回该对象，否则，继续执行步骤 3
3. 查找 `countOf<Key>`、`enumeratorOf<Key>`、和 `memberOf<Key>:` 方法，如果三个方法都找到，则创建一个集合代理对象（该对象响应所有 NSSet 方法）并返回该对象，否则，继续执行步骤 4
4. 如果未找到上述方法，并且类方法 `+ accessInstanceVariablesDirectly` 返回 `YES`，则按顺序查找名称类似于 `_<key>`、`_is<Key>`、`<key>` 或 `is<Key>` 的实例变量。 如果找到，直接获取实例变量的值并执行步骤 5，否则执行步骤 6
5. 判断是否需要对返回值进行包装处理：
   - 如果检索到的属性值是一个对象指针，只需返回结果即可
   - 如果该值是 NSNumber 支持的标量类型，则将其存储在 NSNumber 实例中并返回
   - 如果结果是 NSNumber 不支持的标量类型，则转换为 NSValue 对象并返回
6. 上述情况都失败，则调用 `valueForUndefinedKey:` 方法（该方法默认会抛出异常）

下面给出 getter 流程图：

![](/images/2021/07/KVC-getter.png)

## 数组的处理

在数组上使用 KVC 时，有两种方式可以使用：
- 可以直接通过 `setValue:forKey:` 赋值一个新数组进行更新元素
- 通过 `mutableArrayValueForKey:` 获取 NSMutableArray，然后再通过 `addObject:` 添加元素

> 注意：后者会把属性从 NSArray 改为 NSMutableArray

![](/images/2021/07/202.png)

### 数组的消息传递

当我们对数组实例使用 `valueForKey:` 时，KVC 会遍历数组的每个元素并调用 getter 方法，并返回一个由 getter 返回值组成的新数组。

这也就是说，我们所传递的 key，并不是直接作用于数组对象，而是作用于数组中所包含的每个对象，也就是说，数组对象把 getter 消息转发给了它内部的元素对象。

![](/images/2021/07/204.png)

## 结构体的处理

由于结构体不是对象数据类型，所以不能直接使用 KVC 进行设值/取值操作，需要借助于 `NSValue` 来进行包装/拆包。

![](/images/2021/07/203.png)

> [Representing Non-Object Values](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/DataTypes.html#//apple_ref/doc/uid/20002171-BAJEAIEE) 章节有详细介绍关于非对象值时的处理。

KVC 默认使用 NSNumber 实例来对标量类型数据进行包装/拆包。

下表显示了对于每种数据类型，用于从基础属性值初始化 NSNumber 以提供 getter 返回值的创建方法(Wrapping)， 以及在设值过程中用于从 setter 输入参数中提取值的访问器方法(Unwrapping)。

| 数据类型 | 创建方法 | 访问器方法 |
|:-|:-|:-|
| BOOL | numberWithBool: | boolValue (iOS) / charValue (macOS) |
| char | numberWithChar: | charValue |
| double | numberWithDouble: | doubleValue |
| float | numberWithFloat: | floatValue |
| int | numberWithInt: | intValue |
| long | numberWithLong: | longValue |
| long long | numberWithLongLong: | longLongValue |
| short | numberWithShort: | shortValue |
| unsigned char | numberWithUnsignedChar: | unsignedChar |
| unsigned int | numberWithUnsignedInt: | unsignedInt |
| unsigned long | numberWithUnsignedLong: | unsignedLong |
| unsigned long long | numberWithUnsignedLongLong: | unsignedLongLong |
| unsigned short | numberWithUnsignedShort: | unsignedShort

## 集合运算符

[Using Collection Operators](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/CollectionOperators.html#//apple_ref/doc/uid/20002176-BAJEAIEE)

## 异常处理

在使用 KVC 进行设值/取值的过程中，如果传递了不存在的 key，则会调用 `- setValue:forUndefinedKey:` 或 `- valueForUndefinedKey:` 方法并抛出 `NSUnknownKeyException` 的异常信息，并且应用程序会 Crash。如果我们的项目有特殊需求，可以重写这两个方法并进行处理。

```ObjC
- (void)setValue:(id)value forUndefinedKey:(NSString *)key {
    if ([key isEqualToString:@"undefinedKey"]) {
        return;
    }
    [super setValue:value forUndefinedKey:key];
}

- (id)valueForUndefinedKey:(NSString *)key {
    if ([key isEqualToString:@"undefinedKey"]) {
        return nil;
    }
    return [super valueForUndefinedKey:key];
}

```

## 参考

- [Key-Value Coding Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/index.html#//apple_ref/doc/uid/10000107i)