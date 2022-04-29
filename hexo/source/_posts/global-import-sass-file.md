---
title: Vue 全局引入 sass 文件
abbrlink: be7e4ca2
date: 2022-04-29 08:36:12
tags:
    - Vue
    - CSS
---

## 前言

现在的项目一般都会集成 [CSS预处理器](https://developer.mozilla.org/zh-CN/docs/Glossary/CSS_preprocessor)，不管用的是 [Sass](https://sass-lang.com)、[LESS](https://lesscss.org)、[Stylus](http://stylus-lang.com)、[PostCSS](http://postcss.org) 中的哪一种，都能提高我们 CSS 的编写效率，也能使我们编写的 CSS 看起来更像是一个程序。

这些预处理器都提供了变量、函数等功能方便我们书写 CSS，我们有时候为了复用，也会用一个文件来定义全局的变量、函数等（暂且把这个文件叫做 define.scss 吧），但是如果每个文件中都需要我们手动 import 一次，那岂不是累死人，程序员都是最懒的。毕竟这里定义的都是全局的东西，很多页面都会用到，所以这里我们也通过全局导入的方式来解放双手。

## Webpack

在 `vue.config.js` 文件中加入以下配置：

```JavaScript
const { defineConfig } = require('@vue/cli-service')

module.exports = defineConfig({
    chainWebpack: config => {
        // 全局引入scss文件, 无需在每个组件中再次手动导入
        const oneOfsMap = config.module.rule('scss').oneOfs.store
        oneOfsMap.forEach(item => {
            item.use('sass-resources-loader')
                .loader('sass-resources-loader')
                .options({
                    resources: './src/assets/css/define.scss'
                })
                .end()
        })
    }
})
```

## Vite

[Vite](https://vitejs.cn) 现在可是前端新宠，那么我们如何在 Vite 中全局引入 scss 文件呢？参考官方文档说明：[css.preprocessorOptions](https://vitejs.cn/config/#css-preprocessoroptions)

在 `vite.config.ts` 文件中加入以下配置信息：

```JavaScript
import { defineConfig } from 'vite'

export default defineConfig({
    css: {
        preprocessorOptions: {
            scss: {
                // 全局引入scss文件, 无需在每个组件中再次手动导入
                additionalData: '@use "./src/assets/css/define.scss" as *;'
            }
        }
    }
})
```

经过这么配置，我们就无需在每个页面中都 import 这个文件即可使用文件中定义的变量，又能愉快的 Coding 了...
