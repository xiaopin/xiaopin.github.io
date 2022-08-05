---
title: TypeScript 扩展基础类型
abbrlink: 55ab5906
date: 2022-08-05 10:18:27
tags:
    - TypeScript
---

有时候为了方便，我们会对 JavaScript 内置的基础数据类型进行扩展，添加一些自定义方法。这里介绍下在 TypeScript 中如何进行扩展，添加代码提示。

## 目录结构

```
src
├── @types
│   ├── string.extension.d.ts
│   └── ...
└── extension
    ├── string.extension.ts
    └── ...
```

我们新建两个文件夹 `@types` 和 `extension`，其中 `@types` 文件夹用于存放 `d.ts` 声明文件， `extension` 文件夹用于存放扩展的实现代码。

## 示例

我们以扩展字符串为例：

1、新建 `@types/string.extension.d.ts` 声明文件

```ts
interface String {
    /** 将字符串转成字符数组 */
    toArray(): string[]

    /** 将字符串转成整型数据(失败时返回 undefined) */
    toInt(): number | undefined

    /** 判断字符串是否为纯数字(如：123、123.45) */
    isPureNumber(): boolean
}
```

2、新建 `extension/string.extension.ts` 代码实现文件

```ts
/** 将字符串转成字符数组 */
String.prototype.toArray = function (): string[] {
    return this.valueOf().split('')
}

/** 将字符串转成整型数据(失败时返回 undefined) */
String.prototype.toInt = function (): number | undefined {
    const ret = parseInt(this.valueOf())
    return isNaN(ret) ? undefined : ret
}

/** 判断字符串是否为纯数字(如：123、123.45) */
String.prototype.isPureNumber = function (): boolean {
    return /^\d+(\.\d*)?$/.test(this.valueOf())
}

/** 这里执行 export 语句, 确保该文件是一个模块(可选操作) */
export default {}
```

3、在 `main.ts` 文件顶部导入扩展模块，使扩展方法生效

```ts
import './extension/string.extension'
```

4、在项目中使用扩展方法（无需导入，并且有代码提示）

```vue
<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
    mounted: function () {
        const str = '123456'
        console.log(str.toArray()) // ['1', '2', '3', '4', '5', '6']
        console.log(str.toInt()) // 123456
        console.log(str.isPureNumber()) // true
    }
})
</script>
```

以上便完成了对字符串的功能扩展，我们可以照此方式扩展 Number、Array 等。

> 如果提示 `Property 'xxx' does not exist on type 'String'.` 错误信息，则可以重启 VSCode 试试，因为默认会包含项目下的 `d.ts` 文件，我们无需手动调整。