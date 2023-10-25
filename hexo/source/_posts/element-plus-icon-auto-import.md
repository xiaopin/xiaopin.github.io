---
title: Element-Plus 中图标的使用
abbrlink: e8dd0661
date: 2023-02-19 15:20:23
tags:
  - JavaScript
  - Vue
---
element-plus 提供了一套常用的SVG图标集，可以根据项目实际情况进行使用。

## 安装图标库

```shell
npm install @element-plus/icons-vue
```

## 全局注册

一次导入，全局使用。只需要在 `main.ts` 中导入所有图标并进行全局注册，即可保证在整个项目中所有图标可用，无需再次导入。

```TypeScript
// main.ts

import * as ElementPlusIcons from '@element-plus/icons-vue'

const app = createApp(App)
for (const [key, component] of Object.entries(ElementPlusIcons)) {
    app.component(key, component)
}
```

在模版中使用
```html
<el-icon>
    <Setting />
</el-icon>
```

## 按需导入

### 手动导入

```Vue
<script setup lang="ts">
import { Setting } from '@element-plus/icons-vue'
</script>

<template>
    <el-icon color="red" :size="20">
        <Setting />
    </el-icon>
</template>
```

### 自动导入

安装依赖
```shell
npm i unplugin-icons unplugin-auto-import unplugin-vue-components -D
```

修改配置vite.config.ts
```TypeScript
import { fileURLToPath, URL } from 'node:url'

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'
import Icons from 'unplugin-icons/vite'
import IconsResolver from 'unplugin-icons/resolver'

export default defineConfig({
    plugins: [
        vue(),
        Icons({ autoInstall: true }),
        AutoImport({
            resolvers: [ElementPlusResolver(), IconsResolver()],
            dts: fileURLToPath(new URL('./src/@types/auto-imports.d.ts', import.meta.url))
        }),
        Components({
            resolvers: [ElementPlusResolver(), IconsResolver()],
            dts: fileURLToPath(new URL('./src/@types/components.d.ts', import.meta.url)),
            dirs: [], // src/components目录下的组件不自动导入
            version: 3
        })
    ],
    resolve: {
        alias: {
            '@': fileURLToPath(new URL('./src', import.meta.url))
        }
    }
})
```

通过 `<i-ep-xxx />` 来加载图标：
```html
<el-icon>
    <i-ep-setting />
</el-icon>
```

## 参考

- [官网Icon图标的说明文档](https://element-plus.org/zh-CN/component/icon.html)
- [vite.config.js配置参考](https://github.com/sxzz/element-plus-best-practices/blob/db2dfc983ccda5570033a0ac608a1bd9d9a7f658/vite.config.ts#L21-L58)