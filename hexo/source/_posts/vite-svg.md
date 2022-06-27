---
title: 在Vue3+Vite项目中使用SVG图标
abbrlink: 7e0e72c3
date: 2022-05-31 15:15:01
tags:
    - Vue
    - JavaScript
    - TypeScript
---

在 Vue + Webpack 的项目中，可以通过 [svg-sprite-loader](https://github.com/JetBrains/svg-sprite-loader) 插件来使用 SVG 图标，但是在 Vite 项目下则行不通。

## vite-svg-loader

通过 [vite-svg-loader](https://github.com/jpkleemans/vite-svg-loader) 插件可以将SVG图标作为组件使用。

1、安装插件
```shell
npm install vite-svg-loader --save-dev
```

2、配置 vite.config.js
```JavaScript
import svgLoader from 'vite-svg-loader'

export default defineConfig({
  plugins: [vue(), svgLoader()]
})
```

3、使用SVG图标
```Vue
<template>
  <MyIcon />
</template>

<script setup>
import MyIcon from './my-icon.svg'
</script>
```

通过简单几步即可加载显示SVG，更多功能可以参考官方说明。

## 自定义Vite插件

如果你对 svg-sprite-loader 的使用方式情有独钟，那么我们也可以通过自己写一个Vite插件来加载SVG文件，达到和 svg-sprite-loader 的使用方式一样。

1、将所有SVG文件统一放在 `src/assets/svg/` 目录下

2、新建 `SvgIcon.vue` 组件
```Vue
<template>
    <svg :class="svgClass" :style="svgStyle" aria-hidden="true">
        <use :xlink:href="iconName" />
    </svg>
</template>

<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
    icon: string
    className?: string
    width?: string
    height?: string
    color?: string
}>()

const svgClass = computed(() => `svg-icon ${props.className ?? ''}`)

const svgStyle = computed(() => {
    let style = ''
    if (props.width) style += `width: ${props.width};`
    if (props.height) style += `height: ${props.height};`
    if (props.color) style += `color: ${props.color};`
    return style
})

const iconName = computed(() => `#icon-${props.icon}`)
</script>

<style>
.svg-icon {
    width: 1em;
    height: 1em;
    vertical-align: -0.15em;
    fill: currentColor;
    overflow: hidden;
}
</style>
```

3、新建 `SVGLoader.js` 插件
```JavaScript
import { readFileSync, readdirSync } from 'fs'

let idPerfix = ''
const svgTitle = /<svg([^>+].*?)>/
const clearHeightWidth = /(width|height)="([^>+].*?)"/g
const hasViewBox = /(viewBox="[^>+].*?")/g
const clearReturn = /(\r)|(\n)/g

function findSvgFile(dir) {
    const svgRes = []
    const dirents = readdirSync(dir, {
        withFileTypes: true
    })
    for (const dirent of dirents) {
        if (dirent.isDirectory()) {
            svgRes.push(...findSvgFile(dir + dirent.name + '/'))
        } else {
            const svg = readFileSync(dir + dirent.name)
                .toString()
                .replace(clearReturn, '')
                .replace(svgTitle, ($1, $2) => {
                    let width = 0
                    let height = 0
                    let content = $2.replace(clearHeightWidth, (s1, s2, s3) => {
                        if (s2 === 'width') {
                            width = s3
                        } else if (s2 === 'height') {
                            height = s3
                        }
                        return ''
                    })
                    if (!hasViewBox.test($2)) {
                        content += `viewBox="0 0 ${width} ${height}"`
                    }
                    return `<symbol id="${idPerfix}-${dirent.name.replace('.svg', '')}" ${content}>`
                })
                .replace('</svg>', '</symbol>')
            svgRes.push(svg)
        }
    }
    return svgRes
}

export const svgLoader = (path, perfix = 'icon') => {
    if (path === '') return
    idPerfix = perfix
    const res = findSvgFile(path)
    return {
        name: 'svg-transform',
        transformIndexHtml(html) {
            return html.replace(
                '<body>',
                `
          <body>
            <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" style="position: absolute; width: 0; height: 0">
              ${res.join('')}
            </svg>
        `
            )
        }
    }
}
```

4、配置 `vite.config.js`
```JavaScript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { svgLoader } from './src/plugins/SVGLoader'

export default defineConfig({
    plugins: [
        vue(), 
        svgLoader('./src/assets/svg/')
    ]
})
```

5、使用
```Vue
<script setup lang="ts">
import SvgIcon from './components/SvgIcon.vue'
</script>

<template>
    <!-- email 对应的就是 src/assets/email.svg -->
    <SvgIcon icon="email" width="100px" height="100px" color="red"></SvgIcon>
</template>
```

为了方便使用，可以直接将 `SvgIcon` 注册为全局组件。

在 Webpack 项目中，就是上述的第3、4步不一样而已，第3步为安装 `svg-sprite-loader` 插件，第4步为配置 `vue.config.js`。

## 参考

- [svg-sprite-loader](https://github.com/JetBrains/svg-sprite-loader)
- [vite-svg-loader](https://github.com/jpkleemans/vite-svg-loader)
- [在vue3+vite项目中使用svg](https://segmentfault.com/a/1190000039255368)
