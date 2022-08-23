---
title: JavaScript 条件判断之 boolean 类型值的骚操作
abbrlink: 6262aff0
date: 2022-08-22 16:21:10
tags:
    - JavaScript
    - TypeScript
---

## 前言

条件判断是我们在写代码时最常用的语法之一，如：

```js
let n = 0
if (n) {}

let s = ''
if (s.length) {}

let v1 = undefined
if (v1) {}
```

有时候我们想要得到一个明确的 Boolean 来进行判断/处理逻辑，我们可以通过 `!!` 或者 `!!+` 运算符来强制转成 Boolean 类型，但是在不同的数据类型中，这两者的转换结果也会有所不同。

## String

测试代码：

```js
const strings = ['', ' ', 'a', '0', '1', '1a', '.', '.34']
strings.forEach(element => console.log(element, !!element, !!+element))
```

打印结果如下表所示：

| 原始值 | !! | !!+ |
| :-: | :-: | :-: |
|  | false | false |
|   | true | false |
| a | true | false |
| 0 | true | false |
| 1 | true | true |
| 1a | true | false |
| . | true | false |
| .34 | true | true |

> 结论：对于 String 数据类型，空字符串时两者转换结果都为 false，但是在字符串长度不为空时，`!!` 的转换结果则永远为 true，`!!+` 在数字类型字符串且不等于`'0'`时为 true，其他情况均为 false

可以简单理解为：
- `!!` 等价于 string.length > 0
- `!!+` 等价于 string.length > 0 && 是数字类型字符串 && string != '0'

## Number

测试代码：

```js
const numbers = [0, 1, 1.2, .3]
numbers.forEach(element => console.log(element, !!element, !!+element))
```

打印结果如下表所示：

| 原始值 | !! | !!+ |
| :-: | :-: | :-: |
| 0 | false | false |
| 1 | true | true |
| 1.2 | true | true |
| 0.3 | true | true |

> 结论：对于 Number 类型数据两者转换结果保持一致，对于 `0` 则为 `false`，其他情况均为 `true`

## Boolean

测试代码：

```js
const bools = [true, false]
bools.forEach(element => console.log(element, !!element, !!+element))
```

打印结果如下表所示：

| 原始值 | !! | !!+ |
| :-: | :-: | :-: |
| true | true | true |
| false | false | false |

> 结论：对于 Boolean 类型数据的转换结果和原始值保持一致

## Null

测试代码：

```js
const v = null
console.log(v, !!v, !!+v)
```

打印结果如下表所示：

| 原始值 | !! | !!+ |
| :-: | :-: | :-: |
| null | false | false |

> 结论：对于 Null 类型数据两者转换结果均为 `false`

## undefined

测试代码：

```js
const v = undefined
console.log(v, !!v, !!+v)
```

打印结果如下表所示：

| 原始值 | !! | !!+ |
| :-: | :-: | :-: |
| undefined | false | false |

> 结论：对于 undefined 类型数据两者转换结果均为 `false`

## NaN

测试代码：

```js
const v = NaN
console.log(v, !!v, !!+v)
```

打印结果如下表所示：

| 原始值 | !! | !!+ |
| :-: | :-: | :-: |
| NaN | false | false |

> 结论：对于 NaN 类型数据两者转换结果均为 `false`

## Array

测试代码：

```js
const values = [[], [1], [1, 2], ['1'], ['1', '2'], ['a'], ['a', 'b']]
values.forEach(element => console.log(element, !!element, !!+element))
```

打印结果如下表所示：

| 原始值 | !! | !!+ |
| :-: | :-: | :-: |
| [] | true | false |
| [1] | true | true |
| [1, 2] | true | false |
| ['1'] | true | true |
| ['1', '2'] | true | false |
| ['a'] | true | false |
| ['a', 'b'] | true | false |

> 结论：对于 Array 数据类型，`!!` 的转换结果永远为 `true`，而 `!!+` 的转换结果对于有且只有一个元素，并且该元素是 Number 类型或纯数字类型字符串时结果为 `true`，其他情况均为 `false`

## Function

测试代码：

```js
const func = () => {}
console.log(func, !!func, !!+func)
```

打印结果如下表所示：

| 原始值 | !! | !!+ |
| :-: | :-: | :-: |
| () = {} | true | false |

> 结论：对于 Functionp 类型数据，`!!` 的转换结果永远为 `true`，而 `!!+` 的转换结果则永远为 `false`

## Object

测试代码：

```js
const objs = [{}, { name: 'zhangsan' }]
objs.forEach(element => console.log(element, !!element, !!+element))
```

打印结果如下表所示：

| 原始值 | !! | !!+ |
| :-: | :-: | :-: |
| {} | true | false |
| {name: 'zhangsan'} | true | false |

> 结论：对于 Object 数据类型，`!!` 的转换结果永远为 `true`，而 `!!+` 的转换结果则永远为 `false`

## Symbol

测试代码：

```js
const symbol = Symbol('Test')
console.log(symbol)
console.log(!!symbol)
console.log(!!+symbol)
```

打印结果如下表所示：

| 原始值 | !! | !!+ |
| :-: | :-: | :-: |
| Symbol(Test) | true | TypeError: Cannot convert a Symbol value to a number |

> 结论：对于 Symbol 数据类型，`!!` 的转换结果永远为 `true`，而 `!!+` 的转换会抛出异常

## 结果

对于 `!!` 和 `!!+` 这两个运算符的转换结果，汇总如下表：

| 数据类型 | !! | !!+ |
| :-: | :-: | :-: |
| String | 空字符串时为 false，其他情况为 true | 数字类型字符串且不等于 `'0'` 时为 true，其他情况为 false |
| Number | 0 为 false， 其他情况均为 true | 0 为 false， 其他情况均为 true |
| Boolean | 和旧值保持一致 | 和旧值保持一致 |
| Null | false | false |
| undefined | false | false |
| NaN | false | false |
| Array | true | 当数组只有一个元素且元素为 Number 或 数字类型字符串为 true，其他情况为 false |
| Function | true | false |
| Object | true | false |
| Symbol | true | 抛异常(TypeError: Cannot convert a Symbol value to a number) |

注：
1. 数字类型字符串表示由纯数字和单个点号组成的字符串，如：123、1.2、.34 均为合法格式；1a、1.2.3 这种则为不合法格式
2. `!!` 和 `!!+` 两者转换只在 String/Array/Function/Object/Symbol 这几个数据类型中存在差异，其他数据类型的转换结果两者都保持一致
3. `!!+` 是由[双重逻辑非(!!)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Logical_NOT#double_not_!!)和[一元加法(+)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Unary_plus)两种运算符组合而成，但是一元加法的优先级比逻辑非的优先级要高，所以会先进行一元加法的运算，再进行双重逻辑非的运算，理解了优先级，那么这个转换结果就取决于一元加法了，因为一元加法会尝试将数据转成 Number 类型，转换失败时会返回 NaN，具体可以参考文档说明

对于使用 `!!` 还是 `!!+` 你可以对照上表的结果结合项目实际情况考虑吧，可以简单理解为 `!!` 是不严格的，`!!+` 是更加严格的，我平时都是使用 `!!`，只是在偶然间看到有人使用 `!!+`。

## 参考

- [Boolean](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean)
- [双重非（!!）运算符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Logical_NOT#double_not_!!)
- [一元加法(Unary plus (+))](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Unary_plus)
- [运算符优先级](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)
- [JavaScript 数据类型和数据结构](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures)
