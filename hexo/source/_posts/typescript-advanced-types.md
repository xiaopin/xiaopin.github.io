---
title: TypeScript中的高级类型
abbrlink: cdf9fb67
date: 2022-07-06 08:41:14
tags:
    - TypeScript
---

对于熟悉 JavaScript 的开发者来说，要切换到 TypeScript 是很容易的，只需掌握 TypeScript 中的基础数据类型，然后在写代码时加上对应的类型即可。但是如果稍不注意就会导致很多地方都是用 `any`，最终把项目写成了 `AnyScript`，失去了 TypeScript 的意义。所以我们在掌握了基础类型后，更应该掌握一些高级类型，写出更加健壮的代码。

为了方便演示，我们先定义一个 `User` 数据类型，其数据结构如下所示：

```TypeScript
interface User {
    username: string
    password: string
    nickname?: string
    email?: string
}
```

## 交叉类型(&)

交叉类型是将多个类型合并为一个类型。我们可以把现有的多种类型叠加到一起成为一种类型，它包含了所需的所有类型的特性。

比如我们有一只鸟，但是它会说话，你说神不神奇（对，它就是鹦鹉哈哈哈哈哈）

```TypeScript
interface Bird {}
interface Skill {
    speak(): void
}

type Parrot = Bird & Skill // 一个鸟加上它会说话，得到我们的鹦鹉
```

## 联合类型(|)

联合类型是由两种或多种其他类型组成的类型，表示可能是这些类型中的任何一种的值。

比如你想养一只宠物，却又不知道养什么好，但是你只想从猫和狗中选一种来养：
```TypeScript
interface Cat {} // 猫
interface Dog {} // 狗
interface Parrot {} // 鹦鹉
...

type Pet = Cat | Dog // 宠物, 只能是猫或狗中的一种
```

## [keyof](https://www.typescriptlang.org/docs/handbook/2/keyof-types.html)

类型索引，功能和 `Object.keys` 类似，用于获取类型 T 中定义的所有属性名

```TypeScript
type UserKeys = keyof User
// 等价于
type UserKeys = 'username' | 'password' | 'nickname' | 'email'
```

## 工具类型

TypeScript 为我们定义了很多[工具类型(Utility Types)](https://www.typescriptlang.org/docs/handbook/utility-types.html)，通过这些工具类型，我们可以组合出我们需要的新类型，也能避免我们重复定义数据模型，简化我们的代码。

### Omit

```TypeScript
/**
 * Construct a type with the properties of T except for those in type K.
 */
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

> 将类型 T 中的属性 K 排除掉，得到新类型

比如我们现在需要更新用户信息，但是用户名和密码不能更新：

```TypeScript
type UpdateUser = Omit<User, 'username' | 'password'> // 排除 User 中的 username 和 password 两个属性
```

### Pick

```TypeScript
/**
 * From T, pick a set of properties whose keys are in the union K
 */
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};
```

> 从类型 T 中选取部分属性 K 组成新的类型

比如我们现在需要更新用户信息，但是只能更新昵称和邮箱：

```TypeScript
type UpdateUser = Pick<User, 'nickname' | 'email'> // UpdateUser 包含两个属性 nickname 和 email
```

你是不是发现这个 Pick 和前面的 Omit 有点类似，这两个例子得到的 `UpdateUser` 都是一致的。Omit 是排除，Pick 是选取(选择需要的)，所以如果需要排除的属性过多时，可以考虑用 Pick 代替，只选择需要的部分。

### Required

```TypeScript
/**
 * Make all properties in T required
 */
type Required<T> = {
    [P in keyof T]-?: T[P];
};
```

> 将类型 T 中定义的所有属性设置为必填的

```TypeScript
type RequiredUser = Required<User> // nickname 和 email 也是必填
```

### Partial

```TypeScript
/**
 * Make all properties in T optional
 */
type Partial<T> = {
    [P in keyof T]?: T[P];
};
```

> 将类型 T 中定义的所有属性设置为可选的

```TypeScript
type PartialUser = Partial<User> // username 和 password 也是可选的
```

### Record

```TypeScript
/**
 * Construct a type with a set of properties K of type T
 */
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```

> 用来构建一个对象，属性键为 K，属性值类型为 T

比如我们有一个表单，包含用户名和密码两个属性，属性值类型都是字符串

```TypeScript
type FormData = Record<'username' | 'password', string>

const data: FormData = { 'username': '', 'password': '' }
```

除了给出具体的属性名称，我们也可以只限制属性键的类型，而不给出具体的键名（对于表单类型，我们只限制属性键类型必须是字符串，属性值是字符串/数字/日期中的一种，对于构建动态表单时会很有用）

```TypeScript
type FormData = Record<string, string | number | Date>

const data: FormData = { 'username': '', 'password': '', 'birth': new Date(), 'age': 18 }
```

### Readonly

```TypeScript
/**
 * Make all properties in T readonly
 */
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```

> 将类型 T 中定义的所有属性设置只读(不可修改)

```TypeScript
type ReadonlyUser = Readonly<User> // 所有属性都是只读的

const user: ReadonlyUser = {...}
user.nickname = 'Test' // 报错, 只读属性不可修改
```

### Exclude

```TypeScript
/**
 * Exclude from T those types that are assignable to U
 */
type Exclude<T, U> = T extends U ? never : T;
```

> 通过从 T 中`排除`可分配给 U 的所有联合成员来构造一个新的类型（通俗来说就是获取 T 和 U 的差集）

```TypeScript
type T0 = Exclude<'a' | 'b' | 'c', 'a'> // 'b' | 'c'
type T1 = Exclude<'a' | 'b' | 'c', 'a' | 'b'> // 'c'
type T2 = Exclude<string | number | (() => void), Function> // string | number
type T3 = Exclude<keyof User, 'username' | 'password'> // 'nickname' | 'email'
```

### Extract

```TypeScript
/**
 * Extract from T those types that are assignable to U
 */
type Extract<T, U> = T extends U ? T : never;
```

> 通过从 T 中提取所有可分配给 U 的联合成员来构造一个类型（通俗来说就是获取 T 和 U 的交集）

```TypeScript
type T0 = Extract<'a' | 'b' | 'c', 'a' | 'd'> // 'a'
type T1 = Extract<'a' | 'b' | 'c', 'a' | 'b'> // 'a' | 'b'
type T2 = Extract<string | number | (() => void), Function> // () => void
type T3 = Extract<keyof User, 'username' | 'password' | 'qq'> // 'username' | 'password'

interface A {
    a: string
    test: () => void
}

interface B {
    b: string
    test: () => void
}

type AB = Extract<keyof A, keyof B> // 'test'
```

### NonNullable<T>：从T类型中排除null | undefined

```TypeScript
/**
 * Exclude null and undefined from T
 */
type NonNullable<T> = T extends null | undefined ? never : T;
```

> 将类型 T 中的 null 和 undefined 排除, 表示值是非空的

```TypeScript
type TmpValue = string | number | null | undefined
type NonNullableValue = NonNullable<TmpValue>

let value: NonNullableValue = '' // value 只能是 string 或 number 类型
```

### ReturnType<T>

```TypeScript
/**
 * Obtain the return type of a function type
 */
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
```

> 获取函数 T 的返回值类型

```TypeScript
// 表单规则校验的方法, 返回true表示校验通过, 返回false表示校验不通过
type ValidateRule = (field: string, value: any) => boolean
// 获取 ValidateRule 的返回值并定义为 RT 类型(此时：RT 等价于 boolean)
type RT = ReturnType<ValidateRule>

const checkUsernameValue: ValidateRule = (field: string, value: any) => (typeof value === 'string' && value.length > 2)
const rt: RT = checkUsernameValue('username', 'zhangsan') // true
```

## 参考

- [TypeScript 高级类型](https://www.w3cschool.cn/typescript/typescript-advanced-types.html)
- [Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)
