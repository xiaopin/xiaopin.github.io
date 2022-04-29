---
title: Vite 配置别名 @
abbrlink: eaedff2e
date: 2022-04-29 09:05:51
tags:
    - Vue
---

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

请参考文档说明：[resolve.alias](https://vitejs.cn/config/#resolve-alias)