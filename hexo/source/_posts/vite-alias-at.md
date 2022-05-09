---
title: Vite 配置别名 @
abbrlink: eaedff2e
date: 2022-04-29 09:05:51
tags:
    - Vue
---

## 配置别名@

我们在项目中 import 文件时都喜欢用 `@` 这个别名来表示 `src` 这个目录，在 Vite 项目中，如何配置这个别名呢？

```JavaScript
import { resolve } from 'path'
import { defineConfig } from 'vite'

const srcPath = resolve(__dirname, 'src')

export default defineConfig({
    resolve: {
        alias: {
            '@/': `${srcPath}/`
        }
    }
})
```

配置过后，我们就可以通过 `import XXX from '@/components/XXX.vue'` 来导入某个组件了。

## 添加 TypeScript 支持

如果项目中使用了 TypeScript 语言，按照上面的步骤配置好 Vite 后，就可以通过别名来导入文件，但是此时会提示一个错误信息 

```
Cannot find module '@/xxx' or its corresponding type declarations.Vetur(2307)
找不到模块“@/xxx”或其相应的类型声明。
```

虽然这个报错不会影响程序的运行，但是会导致 TypeScript 的类型提示功能异常，无法进行代码语法提示。

要解决这个错误，需在根目录下的 `tsconfig.json` 中配置 `paths` 内容:

```JSON
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
  }
}
```

## 参考

请参考文档说明：[resolve.alias](https://vitejs.cn/config/#resolve-alias)