---
title: TypeScript 中 type 和 interface 的异同
abbrlink: cb0b7693
date: 2022-08-04 13:40:32
tags:
    - TypeScript
---

[interface](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#interfaces) 和 [type](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-aliases) 都是可以用来定义数据类型的，大部分场景下，能用 `interface` 定义的自然也可以用 `type` 定义，他们两者功能类似，却又有些许区别。

## 相同点

### 定义数据类型

- 定义对象/数据模型

```ts
interface IUser {
    username: string
    password: string
}

type User = {
    username: string
    password: string
}
```

- 定义函数

```ts
interface IUpdateUser {
    (name: string, age: number): void
}

const updateUser: IUpdateUser = (name: string, age: number): void => {
    
}
```

```ts
type UpdateUserHandler = (name: string, age: number) => void

const updateUser: UpdateUserHandler = (name: string, age: number): void => {
    
}
```

### 扩展(extends)

- interface extends interface

```ts
interface IParent {
    name: string
}

interface IChild extends IParent {
    age: number
}
```

- interface extends type

```ts
type Parent = {
    name: string
}

interface IChild extends Parent {
    age: number
}

const c: IChild = { name: 'zhangsan', age: 18 }
```

- type extends type

```ts
type Parent = {
    name: string
}

type Child = Parent & { age: number }
```

- type extends interface

```ts
interface IPerson {
    name: string
}

type Person = IPerson & { age: number }

const p: Person = { name: 'zhangsan', age: 18 }
```

## 区别

1、 interface 声明合并

对于相同名称的接口，TypeScript 会将他们进行合并成一个；如果用 type 定义了多个相同名称的数据类型，则会报重复定义的错误信息。

```ts
interface IUser {
    username: string
    password: string
}

interface IUser {
    username: string // 允许出现重复key, 但是类型必须一致, 如果值类型不相同则会报错
    email?: string
}

// TypeScript 会将相同接口进行合并，所以上述代码等价于：
interface IUser {
    username: string
    password: string
    email?: string
}
```

利用这个特性，我们可以往内置类中添加声明。比如我们在 `window` 中挂在一个 `__APP_VERSION__` 属性：

```ts
window.__APP_VERSION__ = '1.0.0'
// console.log(window.__APP_VERSION__)
```

如果我们直接这么写，代码会提示错误信息：`Property '__APP_VERSION__' does not exist on type 'Window & typeof globalThis'.`，这是因为内置的 `window` 对象并未声明 `__APP_VERSION__` 属性，该属性是我们动态添加上去的，所以我们需要声明一下，告诉 TypeScript 编译器我们有这个属性并且值类型是什么。

```ts
declare global {
    interface Window {
        __APP_VERSION__: string
    }
}
```

添加如上声明后，代码不报错了，并且会有正确的代码提示。

2、 类型别名（type aliases）

type 允许对已存在的数据类型进行重命名（起别名），但是 interface 做不到，interface 只能继承。

```ts
type CustomString = string

function toUpperCase(str: CustomString): CustomString {
    return str.toUpperCase()
}

console.log(toUpperCase('zhangsan'))
```

3、type 中允许使用 typeof 来获取数据类型

```ts
const name: string = 'zhangsan'
type NameType = typeof name // NameType => string

const name1: NameType = ''
```

## 总结

- interface 和 type 均可以定义数据类型，但是 interface 只能定义对象/函数，type 可以定义组合类型、交叉类型、原始类型、元组等
- interface 具有声明合并（`declaration merging`）的特性，但是 type 会报错
- type 可以对数据类型进行重命名，但是 interface 不行

通常你不需要考虑是用 interface 还是 type，你可以根据个人喜好来决定用 interface 还是 type。

## 参考

- [Type Aliases](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-aliases)
- [Interfaces](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#interfaces)
- [Differences Between Type Aliases and Interfaces](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces)
